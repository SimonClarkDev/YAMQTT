# YAMQTT
An easy to use asynchronous C++ MQTT Client library
# YAMQTT

YAMQTT is an easy to build and integrate C++ client MQTT library. It is designed to wrap up all of the technical issues associated with using the MQTT protocol in such a way that a C++ developer can concentrate on building their applications functionality and not worry about the protocol itself or connection states.

### Project Background

The YA in YAMQTT is from the historical 'Yet Another' acronym and came about after several attempts were made to integrate existing MQTT library code into an IOT project. Each offering looked like it could work but in every case it either had large external dependances that could be platform specific or it had significant limitations dictated by the target operating system. Having spent several days on failed integrations it seemed that a new approach was required.

### Usage Example

As an example of how YAMQTT works, all that is needed to initialise it is a supporting serial interface (usually a wired or wireless TCP/IP socket) and a call to connect to the broker. Once this is done the 'engine' is running and it takes care of all connection/disconnection/ping and keep alive responsibilities.

```C++
SocketObject clientSocket;

clientSocket.Open ("127.0.0.1", 1883);
YAMQTTClient mqttClient (clientSocket);

mqttClient.Connect ("MyConnection", true, 10, false);
```

The SocketObject class wraps up any OS INET API which is Linux in this case. The call to Open requires just and source such as an IP address and the MQTT port. Once this is called the YAMQQT client can be instantiated and the connection made. All the connection needs is a name and a clean session flag. The final optional parameters are the time in seconds for pings to be issued to keep a connection alive and whether or not the call should block until the connection is establshed.

Following this, one of more  subscriptions can be registered with or without wildcards.

```C++
mqttClient.Subscribe (std::make_shared<MQTTSubscription> ("Storage/#", 123, MQTTMessage::QualityOfService::AtLeastOnce));
```

The above call creates a subscription to all topics under the top level 'Storage' topic using an application unique ID of 123 in this case, which is used to help identify the subscription when published updated are recevied. Finally, the Quality of Service is specified which affects the reliability of updates sent from the broker.

The last thing to do as and when required and totally application dependant is to publish information to the broker. This is achieved asynchronously through the publish command.

```C++
m_mqttClient.Publish ("Energy/TotalImport", "225", MQTTMessage::QualityOfService::FireAndForget);
```

The above call asynchronously sends a publish message to the broker for the given topic using the quality of service requested.

### Callbacks

In order to get notifications of when an topic has changed a single callback is used.

```C++
void MyApp::MQTTSubscriptionUpdate (const std::shared_ptr<MQTTSubscription>& subscription)
{
	spdlog::trace ("MainApp::MQTTSubscriptionUpdate called on ID {0} with topic [{1}] and message [{2}]", subscription->UniqueIdentifier (), subscription->BrokerTopic (), subscription->BrokerMessage ());
}
```

The above function is called whenever an update is received to a subscribed topic. The subscription object provides information about the topic and message along with the unique ID used when the subscription was created. This allows the application to quickly switch on the unique ID especially in cases where wildcards have not been used.

