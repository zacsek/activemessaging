# Ten Minute Introduction #

We create a simple message consumer to receive messages, and a rails web controller that allows you to send a message when you click a button.  This example uses the stomp adapter and an activemq message broker, but other brokers and adapters are now available.

# Steps #

Start ActiveMQ

```
bin/activemq 
```

Create the rails application

```
rails demo
```

Add the activemessaging plugin

```
script/plugin install http://activemessaging.googlecode.com/svn/trunk/plugins/activemessaging
```

Create a processor

```
script/generate processor HelloWorld
```

This does the following if they don't exist already:
  * add the 'config/broker.yml' file to specify broker connections by environment
  * add the 'config/messaging.rb' file to map Stomp destinations to symbolic names
  * add the 'app/processor' folder where all prcessors that handle messages go
  * add the 'app/processor/application.rb' ApplicationProcessor class, parent to all processors

For your particular HelloWorld processor:
  * add the 'app/processor/hello\_world\_processor.rb'

By default, the processor will be subscribed to a queue ':hello\_world'

```
class HelloWorldProcessor < ApplicationProcessor

  subscribes_to :hello_world

  def on_message(message)
    logger.debug "received: " + message
  end

end
```

If the generator created a new 'config/messaging.rb', it will have by default the queue symbol name to Stomp destination mapping:

```
    s.destination :hello_world, '/queue/HelloWorld'
```

Create a controller with an index action. We will use this to send our messages

```
script/generate controller SayHelloWorld index
```

Add the following to the new controller to allow us to access activemessaging

```
require 'activemessaging/processor'
include ActiveMessaging::MessageSender

publishes_to :hello_world
```

Modify the controller to send a message

```
def say_hello
  publish :hello_world, "<say>Hello World!</say>"
end
```

Add a button to send the message

```
<%=button_to "Say Hello", :action=>"say_hello" %>
```

Send a message. Browse to `http://localhost:3000/say_hello_world` and click the "Say Hello" button.

This adds a say action to the queue, which automatically gets created on first request.

Now, to begin consuming the queue, start the `poller.rb` script by running the poller daemon script:
```
script/poller run
```

The Processor logs the message to the console.