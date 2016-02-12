The dials to turn, and switches to flip, to make a13g behave the way you want it:

## broker.yml ##

When you run the processor generator for the first time, a default 'config/broker.yml' is created.  Its purpose is to configure the message broker connection for each environment, much the same as 'database.yml' for ActiveRecord.

Like database.yml, the top level of the yaml file is divided by environment, and the  section that matches the RAILS\_ENV environment value is the one that gets loaded:
```
development:
...
test:
...
production:
```

By default, a13g will create a single connection to a single broker as configured within the environment section.  This 'default' broker configuration will be used for all queues  unless a different broker is specified for the queue.

It is possible to define multiple connections in the broker.yml.  This is done by adding another level to the yaml file, giving the new connection info a name, for example, to add a 'foo' connection and a 'bar' connection configuration to the development section, do the following:

```
development:
    adapter: stomp
    host: localhost
    reliable: true

    foo:
        adapter: asqs
        access_key_id: XXXXXXXXXXXXXXXXXXXX
        secret_access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX 

    bar:
        adapter: wmq
        q_mgr_name: my_queue_manager

```

In the above configuration, the stomp adapter is the default, and will be used for queues that have no other broker name specified, but foo and bar are also defined, and can be specified as the broker to use for any queue in a messaging.rb queue statement (see syntax below).

Each adapter has it's own defaults and possible configuration values, but all of them start with an 'adapter: something' value to identify which adapter should be used to create the connection based on all the remaining configuration parameters.

### Stomp ###
Here are the defaults for an example stomp connection. With the defaults below, all you really have to specify is the adapter as stomp and you are ready to go:
```
development:
    adapter: stomp
    login: ""
    passcode: ""
    host: localhost
    port: 61613
    reliable: true
    reconnectDelay: 5
```

Here's what you can do with these options:
  * 'login' - set a login to use when connecting to the broker.
  * 'passcode' - set a passcode to use when connecting to the broker.
  * 'host' - address of the broker to connect to for both local and remote brokers.
  * 'port' - port of the broker address to connect to, 61613 seems to be Stomp default, certainly is in ActiveMQ's implementation.
  * 'reliable' - 'true' or 'false', controls whether or not the connection should try and reconnect when the broker is or becomes unavailable.  If true, will retry until it connects or is stopped.  Should almost always be 'true', except for unit testing perhaps.
  * 'reconnectDelay' - integer, number of seconds to wait between reconnect attempts, only meaningful when 'reliable' is 'true'.


### Amazon SQS ###
For Amazon SQS, you must specify a valid access key and secret key, all other values below are defaults, and optional.
```
development:
    adapter: asqs
    access_key_id: XXXXXXXXXXXXXXXXXXXX
    secret_access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    host: queue.amazonaws.com
    port: 80
    reliable: true
    reconnectDelay: 5
    aws_version: 2006-04-01 
    content_type: text/plain
    poll_interval: 1
    cache_queue_list: true
```

Here's what you can do with these options:
  * 'access\_key\_id' - AWS access key id, get this from amazon for your account.
  * 'secret\_access\_key' - AWS secret access key, get this from amazon for your account.
  * 'host' - domain/url to the SQS service for http requests.
  * 'port' - port for http connections to the SQS service.
  * 'reliable' - 'true' or 'false', controls whether or not the connection should retry transmissions to SQS.
  * 'reconnectDelay' - integer, number of seconds to wait between transmission attempts, only meaningful when 'reliable' is 'true'.
  * aws\_version - used in connecting to AWS, this 'constant' is the current version of the sqs service.
  * content\_type - for all messages to SQS, this is the content type of the http request/post.
  * poll\_interval - to receive messages, amount of time to wait between checking all subscribed queues for waiting messages.
  * cache\_queue\_list - a performance enhancement, the adapter will load and keep a list of  valid queues to use for later send and receive calls, or can reload this list each time a queue is accessed to ensure queue is still there and available.


### Beanstalk ###
For Beanstalk, you need to have the gem beanstalk client gem installed (see here for more information: http://beanstalk.rubyforge.org/).
```
sudo gem install beanstalk-client
```

```
development:
    adapter: beanstalk
    host: localhost
    port: 
```

Here's what you can do with these options:
  * 'host' - domain/url to the beanstalkd service.
  * 'port' - port for connections to the beanstalkd service.


### Websphere MQ ###
For Websphere MQ, at a minimum you need to specify the queue manager name, 'q\_mgr\_name', all other parameters are passed in to the underlying WMQ connection, see the WMQ::QueueManager connection [api docs](http://rubywmq.rubyforge.org/doc/classes/WMQ/QueueManager.html).
```
development:
    adapter: wmq
    q_mgr_name: ""
    poll_interval: .1
```

Here's what you can do with these options:
  * q\_mgr\_name - name of the queue manager to connect to.
  * poll\_interval - to receive messages, amount of time to wait between checking all subscribed queues for waiting messages.

### ReliableMessaging ###

TBD

## messaging.rb ##

The main place for configuring your messaging environment is 'config/messaging.rb', also created when the generator is first run.

Here are the methods available on the 'ActiveMessaging::Gateway' while in the 'define' block.  It bears mentioning that since this is just a ruby block, you can do whatever the language and Rails environment allow to solve your own unique issues.

**destination** ['name', symbol for destination] ['destination', string] ['headers', hash of defaults for send] ['broker name', string to specify broker connection info to use]
> You use this method to define destinations - be they queues or topics or whatever else.

> 'name' should be a symbol, and is the logical name for this destination that will be used in the code.

> 'destination' string is the actual value for the destination that the broker understands.

> 'headers' hash is how you can pass in the default headers that should be applied to each message sent to this destination, can be any pair.  The default includes :persistent => true, even if nothing is specified, in order to make sure messages will be saved by ActiveMQ when sent to a destination.  Other header values mapped to JMS headers are described in the ActiveMQ documentation, http://activemq.apache.org/stomp.html.

> 'broker name' string is the name of the broker connection configuration in 'broker.yml' to use for this queue, both for sending to and requesting messages from.  When not specified, uses the default broker connection configuration for the environment.

Here is an example of a 'bar\_queue' using the default broker connection, and the 'foo\_queue' specified to use the 'foo' Amazon SQS broker defined in broker.yml:
```
ActiveMessaging::Gateway.define do |s|
  s.queue :bar_queue, '/topic/bar.queue'
  s.queue :foo_queue, 'foosqsqueue', {}, 'foo'
end
```

With this definition, messages sent to :foo\_queue will be sent to Amazon SQS as configured for foo, and the poller will create a connection to Amazon SQS and a thread to receive messages for foo if a processor subscribes to the foo\_queue.

**connection\_configuration=** ['connection configuration', hash]
> You can use this method to override the configuration loaded for the connection in the 'config/broker.yml'.  All the same hash values can be set, the keys should be set as symbols.  You usually won't need this, but it could be useful for some environments, such as loading configuration from the database, or some other external resource.

**processor\_group** ['group name'] ['array of processors']
> A processor group is a way to run the poller to only execute a subset of the processors by passing the name of the group in the poller command line arguments.

> You specify the name of the processor as its underscored lowercase version.  So if you have a FooBarProcessor and BarFooProcessor in a processor group, it would look like this:
```
    ActiveMessaging::Gateway.define do |s|
      ...
      s.processor_group :my_group, :foo_bar_processor, :bar_foo_processor
    end
```
> The processor group is passed into the poller like the following:
```
    ./script/poller start -- process-group=my_group
```

### poller ###

Included with the procesor generator is the creation of a 'script/poller' script.
This script uses the [ruby daemons library](http://daemons.rubyforge.org) to turn the poller.rb into a daemon.  You need to have the daemon gem for this to work:
```
sudo gem install daemons
```

It is configured to allow running multiple instances, and to create a 'monitor' for each process which will restart it if the poller process ends.  The log file for the poller and the pid files are all put under the 'tmp' directory.

Daemon options can be configured in the 'script/poller' file's options hash:
```
options = {
  :app_name   => "poller",
  :dir_mode   => :normal,
  :dir        => tmp_dir,     #change this to move the pid and poller.output files
  :multiple   => true,        #change this to allow only one instance of the script
  :ontop      => false,
  :mode       => :load,
  :backtrace  => true,
  :monitor    => true,        #change this to turn off the monitor process
  :log_output => true
}
```

See the [ruby daemon documentation](http://daemons.rubyforge.org/) for more information.

Here are the available options when running 'script/poller':
```
> ./script/poller -h
Usage: poller <command> <options> -- <application options>

* where <command> is one of:
  start         start an instance of the application
  stop          stop all instances of the application
  restart       stop all instances and restart them afterwards
  run           start the application and stay on top
  zap           set the application to a stopped state

* and where <options> may contain several of the following:

    -t, --ontop                      Stay on top (does not daemonize)
    -f, --force                      Force operation

Common options:
    -h, --help                       Show this message
        --version                    Show version
```

Additional options can be passed to the underlying script, poller.rb in this case, by using a '--' separator.  At the moment the only supported option is the process\_group, mentioned above.

If you start the poller without specifying a process group, then all processors will be subscribed and run.

If you start with a 'script/poller run -- process-group=critical\_group' command line argument, then only the processors added to that group will be run.