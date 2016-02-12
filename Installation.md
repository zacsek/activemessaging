# Installation #

## Prerequisites ##

To develop applications using ActiveMessaging you need the following prerequisites:
  1. Ruby and gems
  1. Ruby on Rails (version 1.1+)
  1. Database

## Installing ##

### Broker ###
ActiveMessaging supports several different brokers now.

#### Stomp ####
Select from the [list of brokers that speak Stomp](http://stomp.codehaus.org/Brokers), the recommended choice is ActiveMQ configured with the Stomp listener.
If you do use ActiveMQ, there is a page [dedicated to its install and configuration](ActiveMq.md).  You'll need the stomp gem for this.

#### JMS ####
[StompConnect](http://stomp.codehaus.org/StompConnect) exposes any JMS broker as a Stomp broker, so you can use this JMS broker adapter with ActiveMessaging Stomp support to connect to almost any JMS capable broker.  You'll need the stomp gem for this.

#### Amazon Simple Queue Service ####
This is a service provided by amazon, so you don't run the broker, but you do need to sign up for the service and get your own keys to access it.[Amazon SQS](http://www.amazon.com/Simple-Queue-Service-home-page/b/ref=sc_fe_l_2/103-2654138-3198236?ie=UTF8&node=13584001&no=3435361).  No addition gem required - the adapter for this included with ActiveMessaging is standalone.

#### Websphere MQ ####
Formerly IBM MQ Series, Websphere MQ is a commercial messaging broker.  Sylvain Perez has provided a free adapter for this service based on the wmq gem, but you still need to install Websphere MQ to use it. [Websphere MQ](http://www-306.ibm.com/software/integration/wmq/).

#### ReliableMessaging ####
ReliableMessaging is an all Ruby message broker providing a simple API, transactions, different delivery modes, selectors, and file or mysql based storage.  Easy to install and use, it's an all Ruby broker option.


### Gems ###
With Ruby and gems installed, get rails and daemons.  You will also need to get the stomp or wmq gems depending on which broker you are using.
```
sudo gem install rails
sudo gem install daemons

#optional
sudo gem install stomp
sudo gem install wmq
sudo gem install reliable-msg
```

### Plugin ###
To install the plugin into a Rails app:
```
script/plugin install http://activemessaging.googlecode.com/svn/trunk/plugins/activemessaging
```