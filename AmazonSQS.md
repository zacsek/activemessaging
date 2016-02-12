# Introduction #

Amazon Simple Queue Service, as the name implies, is a service based message broker created and supported by Amazon.  There are several ways to integrate with the service, including a simple query based interface.

ActiveMessaging supports using Amazon SQS using its own adapter for REST based access to the service.

# Details #

To use Amazon SQS, [sign up for the service](http://www.amazon.com/Simple-Queue-Service-home-page/b?ie=UTF8&node=13584001) (it is not free, but it is very cheap, 1000 messages = 10 cents).

Once you have signed up, you get an 'access key id' and a 'secret access key'.  Using these, you specify the connection parameters in your 'config/broker.yml' like the following:

```
development:
    adapter: asqs
    access_key_id: XXXXXXXXXXXXXXXXXXXX
    secret_access_key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    
```

That's all you need to specify, and now you'll be connected to Amazon SQS for all messages.


For more advanced configuration options, see the [Configuration](Configuration.md) page.