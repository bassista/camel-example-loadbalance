Using Fuse ESB to Manage Controller Failover
============================================

Created: June 28, 2012
<a href="https://github.com/markbrooks">Mark Brooks</a>

Updated: August 8, 2012
<a href="https://github.com/scranton">Scott Cranton</a>

This document describes a simple prototype that demonstrates the use of Fuse ESB to manage controller failover.

Here is the design:

![Design Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/design.png "Design Image")

Application A sends JMS messages to a ROUTER-INPUT queue. A failover-capable router pickups up messages and sends them
to Controller 1 using a RESTful service call. Controller 1 stamps the message as having been processed and returns it
to the router, which then sends the message to a ROUTER-OUTPUT queue. Application B consumes messages from that queue.
If Controller 1 falls over, the router fails over to Controller 2. For purposes of a fast prototype, Controller 1 and
Controller 2 were run as simple JAX-RS services run from the command line (external to the ESB) so they were easy to
kill to demonstrate failover. In practice, additional Fuse ESB containers would be ideal to host controller services.

Software Requirements

* Fuse ESB
* Fuse IDE (optional)
* JDK 1.6
* Maven 2.2 or 3.0

Download and Install Fuse ESB

* Download Fuse ESB at http://fusesource.com/products/fuse-esb-enterprise/
* I recommend downloading the .zip or .tar.gz archives rather than the full platform installers.
* Unzip the archive in a directory without spaces in its path.

Download and Install Fuse IDE (optional, not required for this demo)

* Download Fuse IDE at http://fusesource.com/products/fuse-ide/
* Unzip the archive in a directory without spaces in its path.

Project Layout
--------------

There are three modules: applications, controllers and router. Here is the layout in the file system:

* FuseESB-RouterDemo
    * applications
    * controllers
    * router

All of the projects’ build and deploy steps can be performed from Fuse IDE or from the command line; I’ll show it from
the command line here as it is the simplest to describe, and then show Fuse IDE afterwards.

Start Fuse ESB
--------------

Run the command `bin/fuseesb` in the root directory of Fuse ESB. Here is what the console looks like:

![Console Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/console.png "Console Image")

Within the Fuse ESB session, run the command `features:install activemq-web-console` to install the message broker’s
console. Then run the command `features:list | grep activemq` to make sure the message broker is available. Make sure
the activemq and activemq-web-console features have a status of `[installed  ]` as in the screenshot below:

![Screenshot Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/screenshot.png "Screenshot Image")

Connect to the ActiveMQ Web Console
-----------------------------------

Launch a browser and connect to the ActiveMQ Web Console at http://localhost:8181/activemqweb/queues.jsp

![ActiveMQ Web Console Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/amq-web-console.png "ActiveMQ Web Console Image")

No Queues exist yet; they will be dynamically created by the router as needed.

Building the projects
---------------------

Launch a new terminal session, and run the command `mvn clean install` in the root directory of top level project.
Make sure all projects build successfully.

Deploy the Router into Fuse ESB
-------------------------------

In the Fuse ESB terminal session run the command:

    FuseESB:karaf@root> install mvn:com.fusesource.demo/router/1.0.0-SNAPSHOT

You should see output like this, with a Bundle ID generated by the deployment:

    FuseESB:karaf@root> install mvn:com.fusesource.demo/router/1.0.0-SNAPSHOT
    Bundle ID: 225
    FuseESB:karaf@root>

Launch the Controllers
----------------------

Open two new terminal sessions and switch to the root of the controllers project in both. In one session, execute this
command to launch Controller 1:

    controllers$ mvn -P controller1-server

And in the other execute the command to launch Controller 2:

    controllers$ mvn -P controller2-server

You should see messages that show the Controllers are both up and running and listening on different ports:

    Controller 1 is ready and listening on port 9001

    Controller 2 is ready and listening on port 9002

Launch Application B (the message consumer)
-------------------------------------------

Open a new terminal sessions and switch to the root of the applications project. In the session, execute this command
to launch Application B  which is the message consumer:

    applications$ mvn -P app-b-consumer

You should see a message like this:

    12:01:08 INFO  ******************************
    12:01:08 INFO  Connecting to Fuse MQ Broker using URL: tcp://localhost:61616
    12:01:08 INFO  ******************************
    12:01:09 INFO  Start consuming messages from queue://ROUTER-OUTPUT with 120000ms timeout

Launch Application A (the message producer)
-------------------------------------------

Open a new terminal sessions and switch to the root of the applications project. In the session, execute this command
to launch Application B, which is the message consumer:

    applications$ mvn -P app-a-producer

You should see Application A sending messages, like this:

    12:05:49 INFO  Sending message 318 with data: <data><messageNumber>318</messageNumber><timestamp>1340910349531</timestamp></data>
    12:05:50 INFO  Sending message 319 with data: <data><messageNumber>319</messageNumber><timestamp>1340910350033</timestamp></data>
    12:05:50 INFO  Sending message 320 with data: <data><messageNumber>320</messageNumber><timestamp>1340910350536</timestamp></data>
    12:05:51 INFO  Sending message 321 with data: <data><messageNumber>321</messageNumber><timestamp>1340910351040</timestamp></data>
    12:05:51 INFO  Sending message 322 with data: <data><messageNumber>322</messageNumber><timestamp>1340910351543</timestamp></data>
    12:05:52 INFO  Sending message 323 with data: <data><messageNumber>323</messageNumber><timestamp>1340910352046</timestamp></data>
    12:05:52 INFO  Sending message 324 with data: <data><messageNumber>324</messageNumber><timestamp>1340910352549</timestamp></data>
    12:05:53 INFO  Sending message 325 with data: <data><messageNumber>325</messageNumber><timestamp>1340910353052</timestamp></data>
    12:05:53 INFO  Sending message 326 with data: <data><messageNumber>326</messageNumber><timestamp>1340910353555</timestamp></data>
    12:05:54 INFO  Sending message 327 with data: <data><messageNumber>327</messageNumber><timestamp>1340910354058</timestamp></data>
    12:05:54 INFO  Sending message 328 with data: <data><messageNumber>328</messageNumber><timestamp>1340910354561</timestamp></data>
    12:05:55 INFO  Sending message 329 with data: <data><messageNumber>329</messageNumber><timestamp>1340910355065</timestamp></data>
    12:05:55 INFO  Sending message 330 with data: <data><messageNumber>330</messageNumber><timestamp>1340910355568</timestamp></data>
    12:05:56 INFO  Sending message 331 with data: <data><messageNumber>331</messageNumber><timestamp>1340910356071</timestamp></data>
    12:05:56 INFO  Sending message 332 with data: <data><messageNumber>332</messageNumber><timestamp>1340910356574</timestamp></data>

And in the Application B session you should see it consuming those messages, stamped with the Controller 1 ID:

    12:07:14 INFO  Got 486. message: <data><messageNumber>486</messageNumber><timestamp>1340910434095</timestamp><controller>Controller 1</controller></data>
    12:07:14 INFO  Got 487. message: <data><messageNumber>487</messageNumber><timestamp>1340910434598</timestamp><controller>Controller 1</controller></data>
    12:07:15 INFO  Got 488. message: <data><messageNumber>488</messageNumber><timestamp>1340910435102</timestamp><controller>Controller 1</controller></data>
    12:07:15 INFO  Got 489. message: <data><messageNumber>489</messageNumber><timestamp>1340910435605</timestamp><controller>Controller 1</controller></data>
    12:07:16 INFO  Got 490. message: <data><messageNumber>490</messageNumber><timestamp>1340910436108</timestamp><controller>Controller 1</controller></data>
    12:07:16 INFO  Got 491. message: <data><messageNumber>491</messageNumber><timestamp>1340910436611</timestamp><controller>Controller 1</controller></data>
    12:07:17 INFO  Got 492. message: <data><messageNumber>492</messageNumber><timestamp>1340910437114</timestamp><controller>Controller 1</controller></data>
    12:07:17 INFO  Got 493. message: <data><messageNumber>493</messageNumber><timestamp>1340910437617</timestamp><controller>Controller 1</controller></data>
    12:07:18 INFO  Got 494. message: <data><messageNumber>494</messageNumber><timestamp>1340910438120</timestamp><controller>Controller 1</controller></data>
    12:07:18 INFO  Got 495. message: <data><messageNumber>495</messageNumber><timestamp>1340910438623</timestamp><controller>Controller 1</controller></data>
    12:07:19 INFO  Got 496. message: <data><messageNumber>496</messageNumber><timestamp>1340910439127</timestamp><controller>Controller 1</controller></data>

Kill the Controller 1 session any way you like, and you should see uninterrupted message flow, with Controller 2
handling the messages:

    12:10:22 INFO  Got 860. message: <data><messageNumber>860</messageNumber><timestamp>1340910622239</timestamp><controller>Controller 1</controller></data>
    12:10:22 INFO  Got 861. message: <data><messageNumber>861</messageNumber><timestamp>1340910622741</timestamp><controller>Controller 1</controller></data>
    12:10:23 INFO  Got 862. message: <data><messageNumber>862</messageNumber><timestamp>1340910623244</timestamp><controller>Controller 1</controller></data>
    12:10:23 INFO  Got 863. message: <data><messageNumber>863</messageNumber><timestamp>1340910623747</timestamp><controller>Controller 1</controller></data>
    12:10:24 INFO  Got 864. message: <data><messageNumber>864</messageNumber><timestamp>1340910624250</timestamp><controller>Controller 1</controller></data>
    12:10:24 INFO  Got 865. message: <data><messageNumber>865</messageNumber><timestamp>1340910624752</timestamp><controller>Controller 1</controller></data>
    12:10:25 INFO  Got 866. message: <data><messageNumber>866</messageNumber><timestamp>1340910625254</timestamp><controller>Controller 1</controller></data>
    12:10:25 INFO  Got 867. message: <data><messageNumber>867</messageNumber><timestamp>1340910625757</timestamp><controller>Controller 1</controller></data>
    12:10:26 INFO  Got 868. message: <data><messageNumber>868</messageNumber><timestamp>1340910626260</timestamp><controller>**Controller 1**</controller></data>
    12:10:26 INFO  Got 869. message: <data><messageNumber>869</messageNumber><timestamp>1340910626762</timestamp><controller>**Controller 2**</controller></data>
    12:10:27 INFO  Got 870. message: <data><messageNumber>870</messageNumber><timestamp>1340910627265</timestamp><controller>Controller 2</controller></data>
    12:10:27 INFO  Got 871. message: <data><messageNumber>871</messageNumber><timestamp>1340910627768</timestamp><controller>Controller 2</controller></data>
    12:10:28 INFO  Got 872. message: <data><messageNumber>872</messageNumber><timestamp>1340910628270</timestamp><controller>Controller 2</controller></data>
    12:10:28 INFO  Got 873. message: <data><messageNumber>873</messageNumber><timestamp>1340910628773</timestamp><controller>Controller 2</controller></data>
    12:10:29 INFO  Got 874. message: <data><messageNumber>874</messageNumber><timestamp>1340910629275</timestamp><controller>Controller 2</controller></data>
    12:10:29 INFO  Got 875. message: <data><messageNumber>875</messageNumber><timestamp>1340910629777</timestamp><controller>Controller 2</controller></data>
    12:10:30 INFO  Got 876. message: <data><messageNumber>876</messageNumber><timestamp>1340910630280</timestamp><controller>Controller 2</controller></data>

Restart Controller 1 and you’ll see it take control back from Controller 2 (this is user-defined behavior in the
configuration of the failover-load balancer)

    12:14:57 INFO  Got 1406. message: <data><messageNumber>1406</messageNumber><timestamp>1340910897042</timestamp><controller>Controller 2</controller></data>
    12:14:57 INFO  Got 1407. message: <data><messageNumber>1407</messageNumber><timestamp>1340910897547</timestamp><controller>Controller 2</controller></data>
    12:14:58 INFO  Got 1408. message: <data><messageNumber>1408</messageNumber><timestamp>1340910898060</timestamp><controller>Controller 2</controller></data>
    12:14:58 INFO  Got 1409. message: <data><messageNumber>1409</messageNumber><timestamp>1340910898577</timestamp><controller>Controller 2</controller></data>
    12:14:59 INFO  Got 1410. message: <data><messageNumber>1410</messageNumber><timestamp>1340910899080</timestamp><controller>Controller 2</controller></data>
    12:14:59 INFO  Got 1411. message: <data><messageNumber>1411</messageNumber><timestamp>1340910899586</timestamp><controller>**Controller 2**</controller></data>
    12:15:00 INFO  Got 1412. message: <data><messageNumber>1412</messageNumber><timestamp>1340910900089</timestamp><controller>**Controller 1**</controller></data>
    12:15:00 INFO  Got 1413. message: <data><messageNumber>1413</messageNumber><timestamp>1340910900591</timestamp><controller>Controller 1</controller></data>
    12:15:01 INFO  Got 1414. message: <data><messageNumber>1414</messageNumber><timestamp>1340910901094</timestamp><controller>Controller 1</controller></data>
    12:15:01 INFO  Got 1415. message: <data><messageNumber>1415</messageNumber><timestamp>1340910901597</timestamp><controller>Controller 1</controller></data>

Refresh the ActiveMQ Web Console and we can see the number of messages processed:

![ActiveMQ Web Console 2 Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/amq-web-console2.png "ActiveMQ Web Console 2 Image")

To inspect the projects using Fuse IDE, launch Fuse IDE and import the three projects as “Existing Maven Projects”:

![Import Maven Projects Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/import-maven-projects.png "Import Maven Projects Image")

Navigate to the router project’s camel-context.xml file to see a visual editor for the integration:

![Camel Context Image](https://github.com/scranton/camel-example-loadbalance/blob/master/images/camel-context.png "Camel Context Image")

Switch to the Source tab to see the definition of the integration:

```xml
<camelContext xmlns="http://camel.apache.org/schema/spring">
    <route id="route1">
        <from uri="activemq:queue:ROUTER-INPUT"/>
        <loadBalance>
            <failover maximumFailoverAttempts="-1" roundRobin="false"/>
            <to uri="direct:controller1"/>
            <to uri="direct:controller2"/>
        </loadBalance>
    </route>
    <route id="c1">
        <from uri="direct:controller1"/>
        <to uri="http:localhost:9001/controller/process"/>
        <to uri="activemq:queue:ROUTER-OUTPUT?jmsMessageType=Text"/>
    </route>
    <route id="c2">
        <from uri="direct:controller2"/>
        <to uri="http:localhost:9002/controller/process"/>
        <to uri="activemq:queue:ROUTER-OUTPUT?jmsMessageType=Text"/>
    </route>
</camelContext>
```

Links to additional documentation
---------------------------------

* http://fusesource.com/documentation/
* http://fusesource.com/docs/esbent/7.0/camel_eip/IntroToEIP-OP.html
* http://fusesource.com/docs/esbent/7.0/camel_comp_ref/Intro-ComponentsList.html