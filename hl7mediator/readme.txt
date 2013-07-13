HL7 Mediator Routes

This bundle demonstrates mediation of HL7 messages using both JMS and MLLP
transport.  It uses the camel-hl7 component http://camel.apache.org/hl7.html.
The camel-hl7 component in turn uses the HAPI library as part of its HL7 codec.
The HAPI API's can be found at http://hl7api.sourceforge.net/ .  

HAPI is dual licensed under the Mozilla Public License 1.1 and the GPLv3.  This
means HAPI can be used with either license.  The MPL 1.1 license is ApacheV2
compatible so that is the license used here.  See the HAPI license here
http://hl7api.sourceforge.net/license.html.  See notes on the ApacheV2
compatible licenses here http://www.apache.org/legal/3party.html.

This module is built with Maven v3.0.4 and requires Java v7 and Maven to be
installed on your system.  

The instructions below are based on using Talend's
ESB Standard Edition which is ApacheV2 licensed.  It has an specific
configuration of Apache Karaf with pre-defined and in some cases pre-installed
features.


Building the Code

The build environment requires only java and maven.  It will run in Eclipse or
Netbeans, and probably any maven compatible IDE.  Commands below are for the
commandline and assume maven and jdk are on the path.

Dependencies
This module depends on the util-spring module.  See the readme for that module
on how to build and compile it.

Steps
To build this code, run unit tests, and create an OSGI bundle use the command
below.  Note that since this is an integration module, the "unit" tests will
need to connect to a local activemq server as well as Mirth.  In addition, if
building in the same environment with other similar routes on other machines,
be sure that their consumers on ActiveMQ do not interfere with this routes or
unit tests may fail (false positive).

Sample mirth channel configurations are included in the test resources.  They
can be imported into mirth channels.  Mirth (or some other hl7 service) must 
also be running in order to pass the "unit" tests.  When actually running in the
VA environment, these mirth channels may need to be different depending on the
location of servers and ports relative to the production mirth server.  A set
of mirth channels has been saved in the test/resources/mirth/va directory.

> mvn clean install

To run the code in a local Spring container with Camel during development:

> mvn camel:run

To deploy the resulting bundle to a Nexus repository edit the
distributionManagement element of the pom to match your repository and then
issue the maven deploy command.  Note that this is not the same as deploying
the bundle to the actual esb node.  This step is not necessary if all of the
builds are being done on the same machine as the target runtime.

> mvn deploy




Host Configuration

In order to make everything a bit more portable between environments, some of
the properties and routes will use hostnames which are aliased via the good old
fashioned etc/hosts file.  See the parent readme to ensure that these are
correctly configured.

Deployment

Before deploying the service, be sure to deploy the default configuration in
the etc folder of the esb.  The properties in this file control the port and
host names used by the route.  A sample is found in the src/main/resources/etc
directory of the git build and it has been configured for the VA environment.

There are two primary mechanisms for deployment: hot deploy and via Karaf's
maven deployment integration.

To deploy any bundle or feature to Karaf using hot deploy, simply copy the
jar or feature files to the deploy folder of the target Karaf runtime.

In order to deploy the bundle to Karaf using maven, Karaf must be pointing to a
repository to which the OSGI bundle has been installed.  In practice, this means
either Karaf is running on the same machine in  which this bundle was compiled
(with Maven), or the Karaf instance is pointing at a Maven repository such as
Nexus to which this bundle has been "deployed" via maven.

In the latter case, the org.ops4j.pax.url.mvn.cfg in the esb etc directory
must have the org.ops4j.pax.url.mvn.repositories property correctly pointing to
the nexus repository.  This is already done in the existing VistA install.

The runtime dependencies are expressed via the features.xml file.  This file is
generated as part of the build process.  Resolution of transitive dependencies
on other features and bundles will be managed and installed as necessary by the
Karaf container.  

This hl7mediator module depends on the util-spring module.  The transitive
dependency management of the karaf features file will handle it.  But the
util-spring module must have been deployed to the nexus repository in order for
this feature to be available to the karaf runtime.  

karaf> features:addurl mvn:com.talend.se.vista.openmash/hl7mediator/1.0-SNAPSHOT/xml/features
karaf> features:install hl7mediator


Manual Deployment via Bundles using Maven

Alternative, if a manual installation using bundles rather than features is
desired (not recommended), then here is the procedures.

karaf> features:install camel-mina
karaf> features:install activemq
karaf> features:install activemq-spring
karaf> features:install activemq-camel
karaf> install -s mvn:ca.uhn.hapi/hapi-osgi-base/2.1
karaf> install -s mvn:org.apache.camel/camel-hl7/2.10.2
karaf> install -s mvn:com.talend.se.demo/util-spring/1.0-SNAPSHOT
karaf> install -s mvn:com.talend.se.vista.openmash/hl7mediator/1.0-SNAPSHOT

A scripted version of these commands are available in hl7mediator.install.script
which is located in the resource folder.


Managing the Route

You can show information about the camel context and the camel routes within
this bundle from using the Karaf console with the camel command extensions to
the karaf shell.  The example below shows only the monitoring commands, but
there are also commands to start, stop, paus, and resume routes; as well as
commands to start and stop the entire context.  Like all Karaf commands these
can be included in script files like the one above used for installation.

# get a list of camel commands
karaf@trun> camel:
camel:context-info     camel:context-list     camel:context-start    camel:context-stop     camel:endpoint-list
camel:route-info       camel:route-list       camel:route-resume     camel:route-show       camel:route-start
camel:route-stop       camel:route-suspend
karaf@trun> camel:context-list
  Name      Status      Uptime
[ camel ] [ Started ] [ 2 minutes ]
karaf@trun> camel:context-info camel
Camel Context camel
        Name: camel
        Version: 2.10.2
        Status: Started
        Uptime: 3 minutes

Statistics
        Exchanges Total: 0
        Exchanges Completed: 0
        Exchanges Failed: 0
        Min Processing Time: 0ms
        Max Processing Time: 0ms
        Mean Processing Time: 0ms
        Total Processing Time: 0ms
        Last Processing Time: 0ms
        Load Avg: 0.00, 0.00, 0.00
        First Exchange Date:
        Last Exchange Completed Date:
        Number of running routes: 1
        Number of not running routes: 0

Advanced
        Auto Startup: true
        Starting Routes: false
        Suspended: false
        Tracing: false

Properties

Components
        mina
        properties
        amq
        log
        spring-event

Endpoints
        amq://mirth-hl7-out
        log://mirthHL7MessageReceived
        log://hapipojo
        mina://tcp://iscache.openmashvista.net:11961?codec=%23hl7codec&sync=true
        spring-event://default

Routes
        consumeFromMirth

Used Languages
karaf@trun> camel:route-list
Route Id             Context Name         Status
[consumeFromMirth  ] [camel             ] [Started           ]
karaf@trun> camel:route-info consumeFromMirth
Camel Route consumeFromMirth
        Camel Context: camel

Properties
                id = consumeFromMirth
                parent = 45b5217e

Statistics
        Exchanges Total: 0
        Exchanges Completed: 0
        Exchanges Failed: 0
        Min Processing Time: 0ms
        Max Processing Time: 0ms
        Mean Processing Time: 0ms
        Total Processing Time: 0ms
        Last Processing Time: 0ms
        Load Avg: 0.00, 0.00, 0.00
        First Exchange Date:
        Last Exchange Completed Date:

Definition
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<route id="consumeFromMirth" xmlns="http://camel.apache.org/schema/spring">
    <from uri="amq:{{activemq.hl7in}}"/>
    <to uri="log:mirthHL7MessageReceived" id="to1"/>
    <unmarshal ref="hl7" id="unmarshal1"/>
    <to uri="log:hapipojo" id="to2"/>
    <marshal ref="hl7" id="marshal1"/>
    <to ref="hl7producer" id="to3"/>
</route>

karaf@trun>

