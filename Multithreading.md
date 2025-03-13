# Thread vs Runnable
Sincle runnable is an interface, can do multiple inhertance as well.

# Name your thread class
```java
public class MyThread extends Thread {
   public MyThread(String name) {
      super(name);
   }
}
```

# join vs get in CompletableFuture
.join() method is used to block the current thread execution and retrieve the results when asynchronous computation is complete. It is similar to .get() except that it does not throw checked exceptions.

# Call multiple rest apis asynchronously and returning result
```java
   //creating custom config class (optional)
   @Configuration
   @EnableAsync
   public class AsyncConfig {
   
       @Bean(name = "customExecutor")
       public Executor taskExecutor() {
           ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
           executor.setCorePoolSize(5);
           executor.setMaxPoolSize(10);
           executor.setQueueCapacity(100);
           executor.setThreadNamePrefix("AsyncThread-");
           executor.initialize();
           return executor;
       }
   }

   //service
   @Async("customExecutor")
    public CompletableFuture<User> fetchUser(int userId) {
        return CompletableFuture.supplyAsync(() ->
            restTemplate.getForObject("https://jsonplaceholder.typicode.com/users/" + userId, User.class)
        );
    }

    @Async("customExecutor")
    public CompletableFuture<List<Order>> fetchOrders(int userId) {
        return CompletableFuture.supplyAsync(() ->
            List.of(restTemplate.getForObject("https://jsonplaceholder.typicode.com/posts?userId=" + userId, Order[].class))
        );
    }

    public CompletableFuture<UserData> fetchUserData(int userId) {
        CompletableFuture<User> userFuture = fetchUser(userId);
        CompletableFuture<List<Order>> ordersFuture = fetchOrders(userId);

        return userFuture.thenCombine(ordersFuture, UserData::new); // Non-blocking ✅
    }

   //
    @GetMapping("/{userId}/data")
    public CompletableFuture<UserData> getUserData(@PathVariable int userId) {
        return apiService.fetchUserData(userId); // Fully async ✅ as we are returning Completable<Future> so CompletableFuture will be immedietly returned to client
        //Spring will register this request and will free the main thread and when asynchronous task completes the data will be returned to client in json form
    }

   @GetMapping("/{userId}/data")
   public UserData getUserData(@PathVariable int userId) {
       return apiService.fetchUserData(userId).join(); // Blocking 🚨 (Not recommended)
   }
```
