# YAMQTT

YAMQTT is an easy to use C++ client MQTT library. It has been designed specifically to wrap up the mechanics associated with using the MQTT protocol in such a way that developers can concentrate on building their applications functionality and not worry about the protocol itself or its connection states.

### Example

YAMQTT only needs a TCP/IP socket intertace and a single call to connect to a broker. Once this is done the client 'engine' runs and it automatically takes care of all connection, disconnection, ping, retry, timeout and keep alive responsibilities.

```C++
SocketObject clientSocket;

clientSocket.Open ("127.0.0.1", 1883);
YAMQTTClient mqttClient (clientSocket);

mqttClient.Connect ("MyConnection", true, 10, false);
```

The SocketObject class in this case wraps up the Linux OS INET API. As a minimum the call to Open requires just an IP address and the MQTT port number. Once this is called the YAMQTT client can be instantiated and the Connect call kicks off all asynchrnous communications.

### Publish

In order to send messages they need to be published to the broker. This is achieved asynchronously through the publish method.

```C++
m_mqttClient.Publish ("Energy/TotalImport", "225", MQTTMessage::QualityOfService::FireAndForget);
```

In the above example the publish message is queued up and sent to the broker for the given topic using the required quality of service with all retires, reconnection attempts and protocol negotiations for the different quality of service levels handled internally.

### Subscribe

In order to be notified about changes to a topic, one of more subscriptions are registered using MQTT filtering and wildcards.

```C++
mqttClient.Subscribe (std::make_shared<MQTTSubscription> ("Storage/#", 123, MQTTMessage::QualityOfService::AtLeastOnce));
```

In the above example a subscription is made to all topics under the top level 'Storage' topic using an application unique ID of 123. The unique ID is used to help identify the subscription information received when updates arrive. The subscription Quality of Service value is specified which can affect the reliability of updates sent from the broker depending on the brokers published QoS.

### Callback

In order to receive notifications of when a topic has changed a single callback interface is used.

```C++
void MyApp::MQTTSubscriptionUpdate (const std::shared_ptr<MQTTSubscription>& subscription)
{
	spdlog::trace ("MainApp::MQTTSubscriptionUpdate called on ID {0} with topic [{1}] and message [{2}]",           subscription->UniqueIdentifier (), subscription->BrokerTopic (), subscription->BrokerMessage ());
}
```

In the above example the function is called whenever an update is received to a subscribed topic. The subscription object parameter provides information about the topic and message along with the unique ID registered when the subscription was created. This allows the application to quickly match the subscriptions in cases where wildcards have been used and immediately match updates with no further decoding if wildcards have not been used.

### Roadmap

Currently this implementation supports MQTT 3.1.1. A newer release will also support 5.0 and be backward compatible.

The code has been passed through Clang Tidy with all filters applied and tested extensively using Mosquito, MQTT Explorer and Home Assistant there are no unit tests.  A future release will add Catch2 to support regression testing.

Currently this implementation supports native Linux. A newer release will also support Windows and FreeRTOS.

