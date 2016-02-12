# Summary #

ReliableMessaging is an all Ruby implementation of a message broker.  The broker is run as its own process, local or remote, and communication is done using DRb.  Much more about ReliableMessaging is available on its [Trac page](http://trac.labnotes.org/cgi-bin/trac.cgi/wiki/Ruby/ReliableMessaging).

## Get ReliableMessaging ##

ReliableMessaging is distributed as a Ruby Gem. Assuming you have rubygems installed, installing it is a matter of getting the gem and dependencies:
```
sudo gem install reliable-msg
```

## Configure ReliableMessaging ##

Once installed, you can generate a configuration file for reliable-msg. It can either use disk or mysql for storage, here are the 2 commands for these options:
```
sudo queues install disk [<path>]

sudo queues install mysql <host> <user> <password> <database> [--port <port>] [--socket <socket>] [--prefix <prefix>]
```
If you don't generate one, a default config file for using disk storage will be created on first run.


## Start ReliableMessaging ##
To start the manager, once again use the 'queues' program:
```
sudo queues manager start
```

You should see something like this:
```
Loaded queues configuration from: /usr/local/lib/ruby/gems/1.8/gems/reliable-msg-1.1.0/queues.cfg
Using message store: disk
Accepting requests at: druby://localhost:6438
```

When you want to stop it, do the obvious thing, hit CTRL-C.

## Known Issues ##
The only problem encountered so far is that connecting to the manager can sometimes be tricky.  On Mac OS X, sometimes connecting fails with 'connection closed' error from DRb.  The workaround for this is to edit the gem to use '127.0.0.1' instead of 'localhost' for the server to accept requests at.