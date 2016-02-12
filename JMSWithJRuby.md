# Using JMS with JRuby #

When using ActiveMessaging with JRuby you can use a JMS broker directly.


# Configuration #

```
#
# broker.yml
#
development:
    ############################
    # JMS Adapter Properties #
    ############################
    adapter: jms

    # properties below are all defaults for this adapter
    # login: ""
    # passcode: ""
    # host: localhost
    # port: 61613
    # reliable: true
    # reconnectDelay: 5

    # NEW! enable stomp retry logic
    #  will resend errored out messages to be retried when on_error throws ActiveMessaging::AbortMessageException
    #
    # Max number of times to retry an aborted message, for 0, will not retry (default)
    # retryMax: 0
    #
    # If error still occurs after retryMax, send message to specified dead letter queue
    # deadLetterQueue: '/queue/activemessaging/deadletter'

test:
    adapter: test
    reliable: false

production:
    adapter: jms
    reliable: true
    # properties below are all defaults for this adapter
    # login: ""
    # passcode: ""
    # host: localhost
    # port: 61613
    # reliable: true
    # reconnectDelay: 5
```