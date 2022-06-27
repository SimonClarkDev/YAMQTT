# YAMQTT

YAMQTT is an easy to build and use C++ client MQTT library. It is designed to wrap up all of the technical complexities associated with using the MQTT protocol in such a way that a C++ developer can concentrate on building their applications functionality and not worry about the protocol itself or connection states.

Currently YAMQTT only supports Linux platforms but the only platform specific code is in the socket module. This library has been created to use dependancy injection with cross platform operation in mind therefore it should be easily port to other popular platforms such as Windows.

### Project Background

The YA in YAMQTT is from the common and historical 'Yet Another' acronym and came about after several attempts were made to integrate existing MQTT library code into an IOT project. Each offering looked like it could work given time but in every case it either had large external dependances that could be platform specific, did not compile or had significant limitations dictated by the target operating system. Having spent several days on fruitless integrations it seemed that a new approach was needed.

I had a 2 week vacation in Cyprus coming up so what better way to relax is there than to impliment a new protocol API whilst sitting in the shade overlooking the Troodos mountains from a villa in Neo Chorio? Most of the work on this project took place by the pool. It's a tough job but someone has to do it.

### Usage Example

As an example of how YAMQTT works, all that is needed to initialise it is a supporting serial interface (usually a wired or wireless TCP/IP socket port) and a single call to connect to the broker. Once this is done the client 'engine' is up and running and it takes care of all connection/disconnection/ping/retry/timeout and keep alive responsibilities.

```C++
SocketObject clientSocket;

clientSocket.Open ("127.0.0.1", 1883);
YAMQTTClient mqttClient (clientSocket);

mqttClient.Connect ("MyConnection", true, 10, false);
```

The SocketObject class in this case wraps up the Linus OS INET API. The call to Open requires just a source such as an IP address and the MQTT port. Once this is called the YAMQTT client can be instantiated and the connection made. All the connection needs is a name and a clean session flag. The final optional parameters are the time in seconds for pings to be issued to keep a connection alive and whether or not the call should block until the connection is establshed. In either case the following methods can be invoked right away even before the connection to the broker is established.

### Subscription

In order to be notified about changes from the broker one of more subscription can be registered with or without wildcards.

```C++
mqttClient.Subscribe (std::make_shared<MQTTSubscription> ("Storage/#", 123, MQTTMessage::QualityOfService::AtLeastOnce));
```

In the above example the method call creates a subscription to all topics under the top level 'Storage' topic using an application unique ID of 123, which is used to quickly identify the subscription when published updates are recevied. Finally, the Quality of Service value is specified which can affect the reliability of updates sent from the broker.

### Publish

The last thing needed as and when required and totally application dependant is to publish application information to the broker. This is achieved asynchronously through the publish command.

```C++
m_mqttClient.Publish ("Energy/TotalImport", "225", MQTTMessage::QualityOfService::FireAndForget);
```

The above method call queues up a publish message to the broker for the given topic using the quality of service requested and is sent as soon as the next client thread time slice occurs.

### Callbacks

In order to receive notifications of when a topic has changed a single callback is used.

```C++
void MyApp::MQTTSubscriptionUpdate (const std::shared_ptr<MQTTSubscription>& subscription)
{
	spdlog::trace ("MainApp::MQTTSubscriptionUpdate called on ID {0} with topic [{1}] and message [{2}]", subscription->UniqueIdentifier (), subscription->BrokerTopic (), subscription->BrokerMessage ());
}
```

The above function is invoked whenever an update is received to a subscribed topic. The subscription object parameter provides information about the topic and message along with the unique ID provided when the subscription was created. This allows the application to quickly match the unique ID especially in cases where wildcards have not been used.

