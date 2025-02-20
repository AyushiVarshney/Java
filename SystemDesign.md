# Design Rate Limiter
Rate limiter is allowing certain number of request for a client in certain interval. Eg. Allowing only 10 req per minute for a client.
This saves us from Dos attack, using misbehaviour. Can be achieved using following algorithms
1. Token Bucket Algo: A bucket is filled with say 10 token per minutes and each token is deleted with client request. If at any time bucket becomes empty access is denied.
2. Fixed Window Counter: A counter resets every minutes. If counter value becomes greater than 10 then access is denied.
3. Sliding Window Log: Timestamps for each requests are maintained in log and log count is validated.
4. Sliding Window counter: Makes use of Sliding window log and fixed window counter.
5. Leaky Bucket: A bucket is maintained of fixed size which leaks a request at fixed interval. Request is added to bucket and if at any point bucket overflows, request is denied.

```java
Use Redis Cluster for distributed rate limiting (maintain counter)
Use API gateway for global rate limiting
Use Lua scripting to perform atomic checks to avoid race condition( when concurrent thread read counter value then there could be mismatch)
User Filter/Interceptor to apply rate limiter globally.
Rate liimting can be done at API level, Per User, Per microservice
```

Example using Leaky Bucket Rate Limiter
```java
@Service
class RateLimiter{

    private final RedisTemplate<String, String> redisTemplate;
    private static final int BUCKET_CAPACITY = 5; // Max requests in the bucket
    private static final int LEAK_RATE_SECONDS = 2; //leaks every 2 seconds

    public LeakyBucketRateLimiter(RedisTemplate<String, String> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public boolean allowRequest(String clientId){
        String queueKey = "leaky_bucket:" + clientId;
        long currentTime = Instant.now().getEpochSecond();

        // Remove old requests (leak processing)
        leakRequests(queueKey, currentTime);

        // Check if bucket has space
        Long bucketSize = redisTemplate.opsForList().size(queueKey);
        if (bucketSize != null && bucketSize >= BUCKET_CAPACITY) {
            return false; // Bucket full, reject request
        }

        // Add new request to the bucket
        redisTemplate.opsForList().rightPush(queueKey, String.valueOf(currentTime));

        // Set expiry for memory cleanup like user is not hitting any request then we expire the key after 10 seconds
        redisTemplate.expire(queueKey, LEAK_RATE_SECONDS * BUCKET_CAPACITY, TimeUnit.SECONDS);

        return true;
    }

    private void leakRequests(String queueKey, long currentTime) {
        while (true) {
            String oldestTimestampStr = redisTemplate.opsForList().index(queueKey, 0);
            if (oldestTimestampStr == null) break;

            long oldestTimestamp = Long.parseLong(oldestTimestampStr);
            if (currentTime - oldestTimestamp >= LEAK_RATE_SECONDS) {
                // Remove oldest request (leak simulation)
                redisTemplate.opsForList().leftPop(queueKey);
            } else {
                break;
            }
        }
    }
}

@Component
public class LeakyBucketFilter implements Filter { //Filter is servlet filter which will intercept all http requets

    private final LeakyBucketRateLimiter rateLimiter;

    public LeakyBucketFilter(LeakyBucketRateLimiter rateLimiter) {
        this.rateLimiter = rateLimiter;
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        String clientIp = httpRequest.getRemoteAddr();

        if (!rateLimiter.allowRequest(clientIp)) {
            httpResponse.setStatus(HttpServletResponse.SC_TOO_MANY_REQUESTS);
            httpResponse.getWriter().write("429 Too Many Requests - Leaky bucket full.");
            return;
        }

        chain.doFilter(request, response); //this will pass the flow to next filter or controller
    }
}
✅ Redis Cluster ensures scalability for millions of RPS
✅ Lua scripting prevents race conditions & enhances atomicity
✅ Redis Streams enable event-based rate limiting
✅ API Gateway filtering reduces backend load
✅ Auto-scaling with Kubernetes & Load Balancing for handling spikes
```

# Design Instagram
High Level Design
```java
                         ┌──────────────────┐
                         │  Client (Mobile) │
                         └────────▲─────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │       API Gateway        │
                    └─────────┬──────────┬─────┘
                              │          │
         ┌──────────────────┐ │          │ ┌──────────────────┐
         │   User Service   │ │          │ │   Post Service   │
         └──────────────────┘ │          │ └──────────────────┘
                              │          │
         ┌──────────────────┐ │          │ ┌──────────────────┐
         │  Feed Service    │ │          │ │ Like/Comment Svc │
         └──────────────────┘ │          │ └──────────────────┘
                              │          │
         ┌──────────────────┐ │          │ ┌──────────────────┐
         │  Notification    │ │          │ │ Messaging (DM)   │
         │      Service     │ │          │ │  WebSockets      │
         └──────────────────┘ │          │ └──────────────────┘
                              │          │
                         ┌──────────────────┐
                         │  Storage Service │  (Amazon S3)
                         └──────────────────┘
API Gateway: Authentication, Rate limiting and forward request to corressponding service (Apigee for gateway, jwt for authentication and Apigee and Redis for rate limiting)
UserService: Deals with user registration, follow/unfollow (Springboot, Oracle for storage and redis for session storage)
FeedService: Deals with generating personalized feed for users (Springboot, keep precomputed cache in Redis and use kafka to subscribe to new feeds)
PostService: Creation of posts (Springboot, Oracale, Redis for caching)
LikeCommentService: Delas with user like unlike comment (Springboot, Oracale, and Redis to store likes count)
NotificationService: Deals with sending notification about like comment follow unfllow, DMs (Kafka)
Messaging: Deals with group/one to one messaging. (Sockets, Kafka, Pub/sub Redis)
StorageService: Service which Stores actual media with userid (Amazon S3)
```

```java
Database schema

┌──────────────┐         ┌──────────────┐
│    Users     │         │   Follows    │
├──────────────┤         ├──────────────┤
│ id (PK)      │◄──┐     │ id (PK)      │
│ username     │   ├───► │ follower_id (FK → Users.id) │
│ email        │   ├───► │ following_id (FK → Users.id) │
│ password     │   │     │ created_at   │
│ profile_pic  │   │     └──────────────┘
│ bio          │   │
│ created_at   │   │     ┌──────────────┐
└──────────────┘   │     │    Posts     │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ user_id (FK → Users.id) │
                   │     │ media_url    │
                   │     │ caption      │
                   │     │ created_at   │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │   Hashtags   │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ name         │
                   │     └──────────────┘
                   │
                   │     ┌────────────────┐
                   │     │  Post_Hashtags │
                   │     ├────────────────┤
                   ├───► │ post_id (FK → Posts.id) │
                   │     │ hashtag_id (FK → Hashtags.id) │
                   │     └────────────────┘
                   │
                   │     ┌──────────────┐
                   │     │    Feeds     │
                   │     ├──────────────┤
                   ├───► │ user_id (FK → Users.id) │
                   │     │ post_id (FK → Posts.id) │
                   │     │ timestamp    │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │    Likes     │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ user_id (FK → Users.id) │
                   │     │ post_id (FK → Posts.id) │
                   │     │ created_at   │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │  Comments    │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ user_id (FK → Users.id) │
                   │     │ post_id (FK → Posts.id) │
                   │     │ content      │
                   │     │ created_at   │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │ Notifications│
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ user_id (FK → Users.id) │
                   │     │ type         │
                   │     │ reference_id │
                   │     │ seen         │
                   │     │ created_at   │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │  Messages    │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ sender_id (FK → Users.id) │
                   │     │ receiver_id (FK → Users.id) │
                   │     │ content      │
                   │     │ timestamp    │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │    Chats     │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ user1_id (FK → Users.id) │
                   │     │ user2_id (FK → Users.id) │
                   │     │ created_at   │
                   │     └──────────────┘
                   │
                   │     ┌──────────────┐
                   │     │ Media_Files  │
                   │     ├──────────────┤
                   ├───► │ id (PK)      │
                   │     │ user_id (FK → Users.id) │
                   │     │ file_url     │
                   │     │ file_type    │
                   │     │ created_at   │
                   │     └──────────────┘
```

# Design Notificatio System
