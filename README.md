> ⚠️ **This project is no longer maintained.** Please use the [IBMMQDotnetClient](https://www.nuget.org/packages/IBMMQDotnetClient).
# Minimalist MQI.Net
Our own .Net Core interface for IBM MQ (WebSphere MQ, MQSeries) with Blackjack and Hookers.

| NuGet Version  | Downloads | Build Status |
| ------------- | ------------- |-----------|
| [![NuGet Version and Downloads count](https://img.shields.io/nuget/vpre/mmqi.net.svg)](http://www.nuget.org/packages/Mmqi.Net/)|[![NuGet Download count](https://img.shields.io/nuget/dt/mmqi.net.svg)](http://www.nuget.org/packages/Mmqi.Net/)|[![Build Status](https://travis-ci.org/fglaeser/mmqinet.svg?branch=master)](https://travis-ci.org/fglaeser/mmqinet)|
## Verbs features matrix

| MQI Verb  | Implemented | Class |
| ------------- | ------------- |-----------|
| MQCONN  | :heavy_check_mark:  | MQQueueManager |
| MQCONNX | :heavy_check_mark:  | MQQueueManager |
| MQDISC  | :heavy_check_mark:  | MQQueueManager |
| MQOPEN  | :heavy_check_mark:  | MQQueue, MQTopic |
| MQCLOSE  | :heavy_check_mark: | MQQueue, MQTopic |
| MQSUB  | :heavy_check_mark: | MQSubscription |
| MQPUT  | :heavy_check_mark: | MQQueue, MQTopic |
| MQPUT1  | :heavy_check_mark: | MQQueueManager |
| MQGET  | :heavy_check_mark: | MQQueue |
| MQINQ  | :x: | |
| MQSET  | :x: | |
| MQCMIT | :heavy_check_mark: | MQQueueManager |
| MQBACK | :heavy_check_mark: | MQQueueManager |
| MQCTL  | :x: | |

## Examples
### Connecting in client mode
```csharp
var queueManagerName = "QM01";
var channel = "SVRCONN.1";
var connectionInfo = string.Format("{0}({1})", "192.168.99.100", 1414);
var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo);
qmgr.Disconnect();
```
### How to put the message on a queue (MQPUT1)
```csharp
string message = "Put1InQueue";
var queueManagerName = "QM01";
var channel = "SVRCONN.1";
var connectionInfo = "192.168.99.100(1414)";
using (var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo))
{
  var outgoing = new MQMessage()
  {
    CharacterSet = MQC.CODESET_UTF,
    Encoding = MQC.MQENC_NORMAL
  };
  outgoing.WriteString(message);

  var od = new MQObjectDescriptor
  {
    ObjectType = MQC.MQOT_Q,
    ObjectName = "QL.QUEUE1"
  };
  qmgr.Put1(od, outgoing, new MQPutMessageOptions());
}
```
### Alternatively, use the Put extension method to make the code cleaner.
```csharp
using (var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo))
{
  var outgoing = new MQMessage()
  {
    CharacterSet = MQC.CODESET_UTF,
    Encoding = MQC.MQENC_NORMAL
  };
  outgoing.WriteString(message);

  qmgr.Put("QL.QUEUE1", outgoing);
}
```
### How to wait for a single message
```csharp
using (var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo))
using (var q = qmgr.AccessQueue("QL.QUEUE1", MQC.MQOO_INPUT_AS_Q_DEF + MQC.MQOO_FAIL_IF_QUIESCING))
{
      var incoming = new MQMessage();
      MQGetMessageOptions gmo = new MQGetMessageOptions();
      gmo.WaitInterval = (int) TimeSpan.FromSeconds(30).TotalMilliseconds; // or MQC.MQWI_UNLIMITED;
      gmo.Options |= MQC.MQGMO_WAIT;
      gmo.Options |= MQC.MQGMO_SYNCPOINT;
      q.Get(incoming, gmo);
      Console.WriteLine(incoming.ReadString(incoming.DataLength));
}
```
### How to publish messages on topics (MQPUT1)
```csharp
using (var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo))
{
  var outgoing = new MQMessage()
  {
    CharacterSet = MQC.CODESET_UTF,
    Encoding = MQC.MQENC_NORMAL
  };
  outgoing.WriteString(message);

  var od = new MQObjectDescriptor
  {
    ObjectType = MQC.MQOT_TOPIC,
    ObjectName = string.Empty,
    Version = MQC.MQOD_VERSION_4
  };
  od.ObjectString.VSString = topicName;
  qmgr.Put1(od, outgoing, new MQPutMessageOptions());
}
```
### Alternatively, use the Publish extension method to make the code cleaner.
```csharp
using (var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo))
{
  var outgoing = new MQMessage()
  {
    CharacterSet = MQC.CODESET_UTF,
    Encoding = MQC.MQENC_NORMAL
  };
  outgoing.WriteString(message);
  
  qmgr.Publish("Some/Topic", outgoing);
}
```
### How to subscribe to topic
```csharp

using (var qmgr = MQQueueManager.Connect(queueManagerName, MQC.MQCO_NONE, channel, connectionInfo))
{
  var subs = new MQSubscription(qmgr);
  subs.Subscribe(
    "MY.SUBS.NAME",
    MQC.MQSO_CREATE + MQC.MQSO_RESUME + MQC.MQSO_NON_DURABLE + MQC.MQSO_MANAGED,
    "Some/Topic");
  var incoming = new MQMessage();
  MQGetMessageOptions gmo = new MQGetMessageOptions();
  gmo.WaitInterval = (int)TimeSpan.FromSeconds(30).TotalMilliseconds; //MQC.MQWI_UNLIMITED;
  gmo.Options |= MQC.MQGMO_WAIT;
  gmo.Options |= MQC.MQGMO_SYNCPOINT;

  subs.Get(incoming, gmo);
  subs.Close(MQC.MQCO_REMOVE_SUB, closeSubQueue: true, closeSubQueueOptions: MQC.MQCO_NONE);
}
```
## License
Minimalist MQI.Net is licensed under the MIT license.
## Installation
* As a prerequisite, you first need to install an IBM MQ client in the system where Mmqi.Net is about to be installed; it is a free library offered by IBM on top of which higher-level ones, such as Mmqi.Net, can connect to queue managers. IBM MQ clients can be downloaded from IBM's website.
You can dowload the client from here ... [IBM MQ V8 Clients](https://www-01.ibm.com/support/docview.wss?uid=swg24037500)

