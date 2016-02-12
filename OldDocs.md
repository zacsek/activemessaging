# Tracing Messages #

It is possible to have activemessaging trace your messages, and show a simple graph of the messaage flows through your system.

View the screencast (2 minutes): http://www.skizz.biz/ActiveMessagingScreencast02-Tracer.mov

[[Image(trace.png)]]

# Prerequisites #

The tracer requires [wiki:Dot]

# How to add a tracer to your application #

Start with the TenMinuteIntroduction application or any other ActiveMessaging application.

Add a tracer

```
script/generate tracer Trace
```

Add a queue for the tracer. The tracer works by sending messages to this queue when any messages are sent or received in the system. Edit `config/messaging.rb` and add the following:

```
s.trace_on '/queue/Trace'
s.queue :trace, '/queue/Trace'
```

Go to the trace page

```
http://localhost:3000/tracer/index
```

Fire some messages though the system

Reload the trace to see the new paths

```
http://localhost:3000/tracer/index
```





# To Do #

  * Allow connection to a stomp server on a different machine
  * Support the Jabber protocol
  * Support multiple protocols

# Thoughts #

From a chat between Chris and Jon:

```
Chris S. 	
Dispatcher.setup(Jabber) do |j|
dispatcher.setup(Stomp) do |s|
?

Jon T. 	
I think you need to define multiple connections
for example you could have multiple STOMP connections in the same app
one to MQ one to ActiveMQ etc
so you would probably have names in a yml file
and then in some other file define the "internal name" to "external name"/connection mapping
then you would start the processor with processor.rb <connection name>
Connection.new(:name=>"ActiveMq", :adapter=>"Stomp" ...) do |it|

Chris S. 	
yeah that is better

Jon T. 	
yes, something like that

Chris S. 	
dispatcher.config("activemq") do |it|
with activemq defined in Conections.yaml

Jon T. 	
Dispatcher.subscribe(Connections["activemq"]) do |message|
  # dispatch message
end

Jon T. 	
oh the messaging connection definitions probably have to be environment specific
that is one connection spec for each connection for each environment
```


# Design Constraints #
  * Basic connection abstractions should closely model [wiki:Stomp] simply because this is the only messaging infrastructure that works well for Ruby at the moment. In particular this means we have to enlist all the channels we are interested in upfront (as subscriptions are added before a call to receive) and we would need to use polling consumers as [wiki:Stomp] works by using a blocking call to receive for receiving messages.
  * Should not require multi-threading. This implies that we can only process one connection at a time per consumer process as the call to receive will block.
  * Should probably be able to support multiple messaging connections. Not sure about this requirement as it could be solved on the messaging infrastructure layer (by for example bridging connections into ActiveMq).
  * Some of the abstractions (like the connection objects themselves) should be useful outside of Rails. This way we could write bridges and other infrastructural items outside of Rails.
  * Should support some form of filters (for tracing etc). Some filters can be built-in but the framework support should be generic.

More thoughts from Chris
Filters are crucial:
  * Filters could also be used to layer functionality onto protocols. For example a Jabber connection might not natively support named queues, but we could add a dispatching filter that would wrap/unwrap a messages with a queue construct `<queue name="foo">[original message]</queue>`
  * Filters can be added to layer Transactionality etc.
  * Many of the patterns in Gregor's book are about smart Filters. Layering them at th endpoints fits nicely with Jim Webber's conception of SOA - put the smarts at the endpoint, not in the network

# Dot #

Dot is a simple tool for vizualising graphs of connected nodes.

Dot is a part of the graphviz distribution, and is available on most modern operating systems. The Graphviz home page is http://www.graphviz.org.

If you are running linux, then it is probably already installed. You can find out by running the command

```
echo "digraph {hello->world}" | dot
```

You should see something like this:

```
digraph {
        node [label="\N"];
        graph [bb="0,0,58,108"];
        hello [pos="29,90", width="0.75", height="0.50"];
        world [pos="29,18", width="0.81", height="0.50"];
        hello -> world [pos="e,29,36 29,72 29,64 29,55 29,46"];
}
```

Dot can produce output in many different formats. For example, here is how to create a png version of the above graph:

```
echo "digraph {hello->world}" | dot -Tpng -o hello_world.png
```

which produces:

[[Image(hello\_world.png)]]

# Naming format #
Naming standard should be based on Gregor Hohpe's patterns:

http://www.enterpriseintegrationpatterns.com/

Another options is naming based on the JMS API, but I'm not too keen on that.

Queues or topics could be named '''Destination'''s or '''Channels'''. There is a point to just call it '''destination''' as that is what Stomp calls this and that is what the name of the header is when communicating with ActiveMq. '''Queue''' is '''Point-to-point Channel''', '''topic''' is '''Publish-Subscribe Channel'''. See http://www.enterpriseintegrationpatterns.com/MessageChannel.html and http://www.enterpriseintegrationpatterns.com/PointToPointChannel.html and http://www.enterpriseintegrationpatterns.com/PublishSubscribeChannel.html.

The processes that receive messages should probably be called '''Consumers''' or '''Polling Consumers''' (as that is how it works), see http://www.enterpriseintegrationpatterns.com/PollingConsumer.html. The type we have are also '''Competing Consumers''', see http://www.enterpriseintegrationpatterns.com/CompetingConsumers.html.

We should probably keep calling the next step in the chain a '''Message Dispatcher''', see http://www.enterpriseintegrationpatterns.com/MessageDispatcher.html.

In the pattern description '''Message Dispatcher''' dispatches to "performers" but there isn't an equivalent pattern in the pattern language. I suggest we keep on calling it '''Message Processor'''.

Once we get some more meat on our bones here, let's ask Gregor to review it all.

