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
