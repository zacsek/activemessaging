# Summary #

ActiveMQ is an open source Java JMS message queue. It supports the Stomp protocol, which ActiveMessaging uses to send and receive messages. We hope to support more protocols soon, but for the moment ActiveMQ is the transport of choice.

## Get ActiveMQ ##

To get a Stomp-talking ActiveMQ on your developer box, try this
page: [Getting Started with ActiveMQ](http://www.activemq.org/site/getting-started.html)

[James Strachan](http://people.apache.org/~jstrachan/) of Apache ActiveMQ greatness has been kind enough to patch up an issue with Stomp to clean up connections when they end improperly. So use a 5.0 snapshot (previously 4.2 - now 5.0 will be the next release), and one after 1/19/2007.  It needs to include [this change list](http://svn.apache.org/viewvc?view=rev&revision=497898). As of 5/12007, this has not been added to the 4.1 code base, and is absent from the 4.1.1 release.

## Configure ActiveMQ ##

Edit `conf/activemq.xml` and add the following connector, the port number 61613 is important:

```
<connector>
  <serverTransport uri="stomp://localhost:61613"/>
</connector>
```

N.B. that in 4.2, this is already present by default, but make sure it is there in case something changes in future releases.

You don't need to configure your queues/topics ahead of use. ActiveMQ creates queues/topics dynamically, so if you are used to some other more configuration happy JMS provider, don't be put off, there is not a step missing here.

If you choose to in have your own config file for activemq (which I suggest), then make a copy of the activemq.xml under the conf directory, like "my-activemq.xml".  Why under conf? The file needs to be on the classpath for activemq to find it, and conf is an obvious place (I actually soft-link my file here from my rails application's conf directory, just to make install simpler and keep everything together).


## Start ActiveMQ ##

Start ActiveMQ
```
cd activemq
./bin/activemq
```

You should see something like this:
```
ACTIVEMQ_HOME: /usr/local/activemq
ACTIVEMQ_BASE: /usr/local/activemq
Loading message broker from: xbean:activemq.xml
INFO  BrokerService                  - Using Persistence Adaptor org.apache.activemq.store.journal.JournalPersistenceAdapter@93b59
INFO  BrokerService                  - ActiveMQ 4.2-incubator-SNAPSHOT JMS Message Broker (localhost) is starting
INFO  BrokerService                  - For help or more information please see: http://incubator.apache.org/activemq/
INFO  ManagementContext              - JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
INFO  JDBCPersistenceAdapter         - Database driver recognized: [apache_derby_embedded_jdbc_driver]
INFO  DefaultDatabaseLocker          - Attempting to acquire the exclusive lock to become the Master broker
INFO  DefaultDatabaseLocker          - Becoming the master on dataSource: org.apache.derby.jdbc.EmbeddedDataSource@b2f811
INFO  JournalPersistenceAdapter      - Journal Recovery Started from: Active Journal: using 5 x 20.0 Megs at: /usr/local/apache-activemq-4.2-incubator-SNAPSHOT/activemq-data/journal
INFO  JournalPersistenceAdapter      - Journal Recovered: 0 message(s) in transactions recovered.
INFO  TransportServerThreadSupport   - Listening for connections at: tcp://yournamehere.local:61616
INFO  TransportConnector             - Connector openwire Started
INFO  TransportServerThreadSupport   - Listening for connections at: ssl://yournamehere.local:61617
INFO  TransportConnector             - Connector ssl Started
INFO  TransportServerThreadSupport   - Listening for connections at: stomp://yournamehere.local:61613
INFO  TransportConnector             - Connector stomp Started
INFO  NetworkConnector               - Network Connector default-nc Started
INFO  BrokerService                  - ActiveMQ JMS Message Broker (localhost, ID:yournamehere.local-49369-1168358773693-1:0) started
INFO  TransportConnector             - Connector vm://localhost Started
INFO  KahaStore                      - Kaha Store successfully deleted data directory activemq-data/localhost/tmp_storage
```

When starting activemq with a customized config file, specify this file using the following syntax (see the ActiveMQ docs for more info):
```
./bin/activemq xbean:myamq.xml
```

When you want to stop it, do the obvious thing, hit CTRL-C.