# YAMQTT

YAMQTT is an easy to use C++ client MQTT library. It is designed to wrap up all of the complexities associated with using the MQTT protocol in such a way that C++ developers can concentrate on building their applications functionality and not worry about the protocol itself or connection states.

Currently YAMQTT only supports Linux platforms but the only platform specific code is in the socket module and that is attached through dependency injection so porting should be simple.

### Project Background

The YA in YAMQTT is from the common and historical 'Yet Another' acronym and came about after several attempts were made to integrate existing MQTT library code into a new IOT project. Each solution looked like it would work given the necessary time and effort but in every case it either had large external dependences that could be platform specific, did not compile with standard toolchains or had significant limitations dictated by the target platform.

Having spent many hours on fruitless integration attempts it seemed that a new approach was needed and YAMQTT was born.

### Usage Example

As an example of how YAMQTT works, all that it needs is a supporting serial interface (usually a TCP/IP socket port) and a single call to connect to an MQTT broker. Once this is done the client 'engine' runs and it takes care of all connection, disconnection, ping, retry, timeout and keep alive responsibilities.

```C++
SocketObject clientSocket;

clientSocket.Open ("127.0.0.1", 1883);
YAMQTTClient mqttClient (clientSocket);

mqttClient.Connect ("MyConnection", true, 10, false);
```

The SocketObject class in this case wraps up the Linux OS INET API. As a minimum the call to Open requires just a source such as an IP address and the MQTT port. Once this is called the YAMQTT client is instantiated and the connection is made. All the connection needs is a name and a clean session flag. The final optional parameters are the time in seconds for pings to be issued to keep a connection alive and whether or not the call should block until the connection is established. Following this client methods can be invoked right away even before the connection to the broker is established.

### Subscription

In order to be notified about changes from the broker one of more subscriptions can be registered using standard MQTT filtering and wildcards.

```C++
mqttClient.Subscribe (std::make_shared<MQTTSubscription> ("Storage/#", 123, MQTTMessage::QualityOfService::AtLeastOnce));
```

In the above example the client method call creates a subscription to all topics under the top level 'Storage' topic using an application unique ID of 123. The unique ID is used to help identify the subscription information received when published updates arrive. Finally, the subscription Quality of Service value is specified which can affect the reliability of updates sent from the broker depending on the published QoS.

### Publish

In order to send messages to the broker they need to be published to the broker. This is achieved asynchronously through the publish method.

```C++
m_mqttClient.Publish ("Energy/TotalImport", "225", MQTTMessage::QualityOfService::FireAndForget);
```

In the above example a publish message is queued up and sent to the broker for the given topic using the quality of service requested. All retires, reconnection attempts and protocol negotiations for different quality of service levels are handled internally.

### Callbacks

In order to receive notifications of when a topic has changed a single callback interface is used.

```C++
void MyApp::MQTTSubscriptionUpdate (const std::shared_ptr<MQTTSubscription>& subscription)
{
	spdlog::trace ("MainApp::MQTTSubscriptionUpdate called on ID {0} with topic [{1}] and message [{2}]",           subscription->UniqueIdentifier (), subscription->BrokerTopic (), subscription->BrokerMessage ());
}
```

In the above example the function is called whenever an update is received to a subscribed topic. The subscription object parameter provides information about the topic and message along with the unique ID registered when the subscription was created. This allows the application to quickly match the subscriptions in cases where wildcards have been used and immediately match updates with no further decoding if wildcards have not been used.
