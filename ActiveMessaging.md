# ActiveMessaging #

ActiveMessaging is an attempt to bring the simplicity and elegance of rails development to the world of messaging. Messaging, (or event-driven architecture) is widely used for enterprise integration, with frameworks such as Java's JMS, and products such as ActiveMQ, Tibco, IBM MQSeries, etc.

ActiveMessaging is a generic framework to ease using messaging, but is not tied to any particular messaging system - in fact, it now has support for Stomp, Amazon Simple Queue Service (SQS), Beanstalk, JMS (using StompConnect or [direct on JRuby](JMSWithJRuby.md)), WebSphere MQ, and the all-Ruby ReliableMessaging.

Here's a sample of a processor class that handles incoming messages:
```
class HelloWorldProcessor < ActiveMessaging::Processor
  
  subscribes_to :hello_world
  
  def on_message(message)
    puts "received: " + message
  end
  
end
```

# Getting Started #

Here are the best beginner guides:
  * [Installation](Installation.md) - get ActiveMessaging, stomp, and activemq
  * [TenMinuteIntroduction](TenMinuteIntroduction.md) - quick intro on sending and receiving
  * [UsingActiveMessagingAsGemInRails](UsingActiveMessagingAsGemInRails.md) - how to install ActiveMessaging in Rails as a gem.

Then after you get through that, learn more wih these:
  * [Configuration](Configuration.md) - more advanced configuration described
  * [ExceptionHandling](ExceptionHandling.md) - advice and recipes for better exception handling


# ActiveMessaging articles elsewhere #
  * [Processing Pipeline with Amazon S3, SQS, EC2, Ruby, Rails and ActiveMessaging](http://developer.amazonwebservices.com/connect/entry.jspa?externalID=1151) By Philip@AWS, a terrific example of using Amazon SQS with ActiveMessaging for applications that require a large amount of 'heavy-lifting' be handled asynchronously and in a scalable manner on EC2 instances.
  * [Intro to ActiveMessaging on InfoQ](http://www.infoq.com/articles/intro-active-messaging-rails) - Still mostly current/complete intro.
  * [Asynchronous Messaging with Rails](http://beechbonanza.blogspot.com/2007/06/asynchronous-messaging-with-rails.html) - example and explanation of using a13g with AR observers, discussion of a13g as alternative to REST integration.
  * [Integrating Rails and ActiveMQ with ActiveMessaging/REST](http://notdennisbyrne.blogspot.com/2007/06/integrating-rails-and-activemq-with.html) - uses ActiveMQ's demo chat app on jetty interacting with a rails app using a13g.
  * [How to build a mini youtube using activemessaging plugin and amazon s3](http://blog.snowonrails.com/articles/2007/05/31/how-to-build-a-mini-youtube-using-activemessaging-plugin-and-amazon-s3) - title says it all.
  * [Publish\Subscribe Messaging with Flex and Rails using Apache ActiveMQ, ActiveMessaging, and STOMP](http://flexonrails.net/?p=83) - very cool example of using flex to communicate via stomp.

# Developers #

[Source code](http://code.google.com/p/activemessaging/source) is available via subversion here on google code:

```
svn checkout http://activemessaging.googlecode.com/svn/trunk/ activemessaging
```

# Support #

Best bet is the google groups mailing list:

http://groups.google.com/group/activemessaging-discuss