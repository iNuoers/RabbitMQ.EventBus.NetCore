# RabbitMq for .NetCore 2.2


#### Install

```c#
>> Install-Package RabbitMQ.EventBus.NetCore
 or
>> dotnet add package RabbitMQ.EventBus.NetCore
```

#### Startup.cs 

```c#

services.AddRabbitMQEventBus("amqp://you-queue", eventBusOptionAction: eventBusOption =>
{
          eventBusOption.ClientProvidedAssembly<Startup>();
          eventBusOption.EnableRetryOnFailure(true, 5000, TimeSpan.FromSeconds(30));
          eventBusOption.RetryOnFailure(TimeSpan.FromSeconds(1));
});

app.RabbitMQEventBusAutoSubscribe();

```

#### A simple model

```c#

    [EventBus(Exchange = "rabbitmq.eventBus.mail", RoutingKey = "rabbitmq.eventbus.mail")]
    public class MailModel : IEvent
    { 
        public string Email {get; set;} 
        public string SubTitle {get; set;}         
        public string Body { get; set; }
        public DateTimeOffset Time { get; set; }
    }
```
#### A model with N RouteKey

```c#

    [EventBus(Exchange = "rabbitmq.eventBus.mail", RoutingKey = "rabbitmq.eventbus.mail")]
    [EventBus(Exchange = "rabbitmq.eventBus.mail", RoutingKey = "rabbitmq.eventbus.mail-other")]
    public class MailOtherModel : IEvent
    {
        public string Email {get; set;} 
        public string SubTitle {get; set;}         
        public string Body { get; set; }
        public object Data { get; set; }
        public DateTimeOffset Time { get; set; }
    }

```
#### Subscribe to the event

```c# 

    public class MailBodyHandle : IEventHandler<MailOtherModel>, IDisposable
    {       
        public Task Handle(MailModel message)
        {
            Console.WriteLine(typeof(MailModel).Name);
            return Task.CompletedTask;
        }
        
        public Task Handle(MailOtherModel message)
        {
            Console.WriteLine(typeof(MailOtherModel).Name);
            return Task.CompletedTask;
        }
        
    }
    
```

#### Publish event

```c# 

    [ApiController]
    [Route("api/[controller]")]
    public class ValuesController : ControllerBase
    {
        private readonly IRabbitMQEventBus _eventBus;

        public ValuesController(IRabbitMQEventBus eventBus)
        {
            _eventBus = eventBus ?? throw new ArgumentNullException(nameof(eventBus));
        }

        // GET api/values
        [HttpGet]
        public ActionResult<string> Get()
        {
                _eventBus.Publish(new
                {
                    Body = "Hello Word",
                    Time = DateTimeOffset.Now
                }, exchange: "rabbitmq.eventBus.mail", routingKey: "rabbitmq.eventbus.mail");
                
                _eventBus.Publish(new
                {
                    Body = "Hello Word",
                    Data = new {
                       Name = "RabbitMq"
                    },                            
                    Time = DateTimeOffset.Now
                }, exchange: "rabbitmq.eventBus.mail", routingKey: "rabbitmq.eventbus.mail-other");
               
            return "Ok";
        }
}
```
