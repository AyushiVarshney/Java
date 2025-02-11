# Configuring Reply back topic in Kakfa so that client can be notified after successful processing
group-id here ensures that only 1 consumer withnin a group recevies that message. 
If multiple consumers have same group-id and same topic, then message will be distributed to any one consumer(Load-balancing)
If multiple consumers have different group-id and same topic, then each group will receive its own copy. (Broad casting) 

@RestController
@RequestMapping("/orders")
public class Order Controller{
      @Autowired
      private KakfaTemplate<String, String> kafkaTemplate;
      
      @PostMapping("/create/{orderId}")
      public String placeOrder(@PathVariable String orderId){
            kakfaTemplate.send("order-topic", orderId);
            return "Order " + orderId + " sent for processing";
      }
}

Kafka Consumer
@Component
public class OrderProcessor{
      @Autowired
      private KafkaTemplate<String, String> kafkaTemplate; //here this is required as we will send reply in diff topic

      @KakfaListener(topics = "order-topic", group-id = "order-group")
      @SentTo("reply-topic") //here sendto will automatically take return value of this method.
      public void processOrder(String orderId){
            try{
                  System.out.println("Processing order: " + orderId);
                  return "Order Id " + orderId + " processed successfully";
            } catch(Exception e){
                  //here we can keep a count also like how many times we want to try procesing before sending to DLQ
                  kafkaTemplate.send("deal-letter-topic", orderId);//can be send to separate topic for later processing
                  return "Failed to process Order Id " + orderId;
            }
      }
}

@Component
public class OrderResponseListener{

      @KakfaListener(topics = "reply-topic", group-id = "order-group")
      public void listenResponse(String message){
            System.out.println("Received response: ", message);
      }
}
