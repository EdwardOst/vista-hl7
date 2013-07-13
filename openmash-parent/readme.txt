demo-parent

Common parent pom for all demo projects.  Should ideally allow to build the
project for karaf, tomcat, jetty, or spring by justing changing package type and
setting profiles.

Profiles are controlled using file flags in the profiles folder,
e.g. spring.profile, tomcat.profile, jetty.profile, karaf.profile,  There is one
non-platform profile for logging.  The properties used by the project are 
designed so that they can be overridden at various stages in the development 
cycle via the container specific properties file.  This is done via coordinated
use of Maven filters and Spring / OSGI / Camel placeholders.

Property file conventions are as follows.  First, there must be a platform 
property file in the project directory, e.g. springContainer.properties.  
This file must follow the convention of <platform>Container.properties.  In 
addition, there must be an application properties file following the convention 
<artifactId>.properties.

The actual properties "contract" is specified in the 
src/main/resources/container.properties file.  This template is instantiated by
Maven filters with platform specific properties.  The platform specific property
files are located in the project root directory, e.g. springContainer.properties
and karafContainer.properties.

When unit tests are run the surefire plugin is configured to use the
spring container container.xml via the containerConfig system property.  However, 
it is still the target platform's properties that are being applied.  These can
be overridden via the container.test.properties.

Likewise, the  camel-maven-bundle plugin is configured to use the spring container
via the same containerConfig system property.  However, the platform properties
will still be applied.


