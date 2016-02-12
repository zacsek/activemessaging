## Exception Handling ##

Part of making ActiveMessaging more reliable and fault tolerent is how you handle exceptions.  Not everyone wants this to work the same way, but chances are most folks don't want the poller to shut down for every error.  So, in true rails form, there is a pretty useful default behavior, and with a little effort you can extend or change it as you like.

### Processor on\_error ###

For any Processor, when there is an error raised from calling 'on\_message', this exception is passed to the processor's 'on\_error' method.

The 'app/processor/application.rb' parent processor has a default implementation for the error handling in all processors that logs all exceptions extending from 'StandardError', which is the default for the 'rescue' expression.  All other Exceptions, probably of a severe and unrecoverable nature, are logged and then raised up where they will be raised up to the 'poller.rb' script.  Here is the current 'on\_error' impl:

```
  def on_error(err)
    if (err.kind_of?(StandardError))
      logger.error "ApplicationProcessor::on_error: #{err.class.name} rescued:\n" + \
      err.message + "\n" + \
      "\t" + err.backtrace.join("\n\t")
    else
      logger.error "ApplicationProcessor::on_error: #{err.class.name} raised: " + err.message
      raise err
    end
  end
```

If you don't like this, just change it in 'app/processor/application.rb', or override it in each processor class that extends from this if a particular processor has a different need.

### Adding email notification on error ###

There are other recipes for this, but here is one way to add emailing exceptions to admins.

Make sure ActionMailer is configured properly in 'environment.rb' and 'environments/**.rb'.
This configuration is different with recent edge Rails changes, so I won't try to document this here.**

Generate an actionmailer, I called mine 'ErrorMailer' with a single 'notify' method.
```
script/generate ErrorMailer notify
```

This gets created under app/model/error\_mailer.rb, and I edited it to the following:
```
class ErrorMailer < ActionMailer::Base

  def notify(message,error)
    #if message is multi-line, make 1st line subject
    eol = error.message.index("\n")
    fl = error.message[0,error.message.index("\n")] if eol
    fl = error.message if not eol
    @subject    = "[my application] error: #{fl}"
    @body       = {:error=>error,:message=>message}
    @recipients = 'admin@activemessaging.com'
    @from       = 'admin@activemessaging.com'
    @sent_on    = Time.now
    @headers    = {}
  end

end
```

For this example, we'll send in the the 'message' which was passed to the processor, and the 'error'.  Then you need to create a better view for the error email using the info passed in the 'body' map.

I have under app/views/error\_mailer/notify.rhtml:
```
my application has experienced an exception while processing a message.

message:
<%= @message.body %>

exception:
<%= @error.class.name %>
<%= @error.message %>:
<% for line in @error.backtrace %>
    <%= line.chomp %><% end %>
```

Then you need to add the hook to send this on each processor exception, which is easier with some of the new a13g changes.

The latest a13g processor generator makes the parent of all processors the ApplicationProcessor, which gets copied to /app/processors/application.rb, so it acts like the ApplicationController.
In that class is a default 'on\_error' method that gets passed any exception that occurs in a processor.  Add the one line to call the mailer:
```
class ApplicationProcessor < ActiveMessaging::Processor
 
  def on_error(err)

    #call the error mailer
    ErrorMailer.deliver_notify(message,err)
    ...

  end

end

```

And that should do it, now all exceptions in any processor will be mailed to the admin.

### Poller script exception handling ###

The 'activemessaging/poller.rb' is the main script (executed by script/poller) that loads and runs all the processors, and loops continuously receiving messages has its own exception handling.

It has a rescue block around the receive messages loop to quietly halt when there is an 'Interrupt', and print out any other error.  More importantly, what this block ensures is that when the poller ends, it always tries to unsubscribe from all current subscriptions in the connection, and then disconnect from stomp.  Recent versions of AMQ do this automatically, but prior to this and with other brokers, it is essential.

So change this at your own risk.