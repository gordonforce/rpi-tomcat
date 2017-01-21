## rpi-tomcat
[![](https://images.microbadger.com/badges/image/gordonff/rpi-tomcat.svg)](https://microbadger.com/images/gordonff/rpi-tomcat "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/gordonff/rpi-tomcat.svg)](https://microbadger.com/images/gordonff/rpi-tomcat "Get your own version badge on microbadger.com")

A stable version of Tomcat 8 and JRE 8 for Raspberry Pi similar to other images on docker hub except this one does not remove apt sources; hence, it is extensible. Seen as a base image for Tomcat dependent applications.

### How to use this image

Use an image tag to select a specific versions of tomcat starting with version 8.5.11.

* 8.5.11, 8.5, 8.5-jre, latest: [Dockerfile](https://github.com/gordonforce/rpi-tomcat/blob/master/Dockerfile)

The usage examples on the [Offical Tomcat image dockerhub  page](https://hub.docker.com/_/tomcat/) are valid for this image after changing the image name. Here are the examples using the update image name and tag. Use control-c to stop each image or stop a container using the docker stop command from another terminal.

```yaml
$ docker run -it --rm -p 8080:8080 gordonff/rpi-tomcat:8.5

$ docker run -it --rm -p 8888:8080 gordonff/rpi-tomcat:8.5
```

The examples directory contains docker-compose examples described below

#### Tomcat Manager Application

Deployers either add some form of authenticated access to the Tomcat manager application or remove it as it can be a security risk if left poorly configured and deployed.

The `$CATALINA_HOME/conf/tomcat-users.xml' file provides a simple form of authenticated access without a third party service such as LDAP or a database server. Note the provided schema file allows one to check their users file against an xml schema.

Tomcat's manager applications contains three interfaces:
* an HTML based HTTP interface for humans
* a text based HTTP interface for automated scripts
* a JMX interface for monitoring applications

When one defines a user login and password they also must declare the role, or interface, the user interacts with. The thinking is a management console should not have gui access as it's not a human and visa versa. While one can define multiple roles or interfaces for a single user, it's not seen as a good practice in production environments.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tomcat-users xmlns="http://tomcat.apache.org/xml"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
              version="1.0">

    <role rolename="manager-gui"/>
    <role rolename="manager-text"/>
    <role rolename="manager-jmx"/>
    <user username="bob" password="abc123" roles="manager-gui"/>
    <user username="otto" password="abc12;" roles="manager-text,manager-jmx"/>

</tomcat-users>
```
`examples/manager/tomcat-users.xml`

The context file for the manager application must be updated to allow access from a host other than the raspberry pi (localhost). When deployed, the context file is `/usr/local/tomcat/conf/Catalina/local/manager.xml`. Here is an example of allowing access from any host to the default Tomcat port 8080.

```xml
<Context  privileged="true">
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         addConnectorPort="true"
         invalidAuthenticationWhenDeny="true"
         allow=".*;8080"/>
</Context>
```
`examples/manager/manager.xml`

The docker compose file to start this example follows

```yaml
version: "2.0"

services:
  tomcat:
    environment:
    - CATALINA_OPTS="-server -Xmx64m"
    image: gordonff/rpi-tomcat:latest
    ports:
    - 8080:8080
    volumes:
    - ./tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.xml:ro
    - ./manager.xml:/usr/local/tomcat/conf/Catalina/localhost/manager.xml:ro
```
`examples/manager/manager.xml`

By manipulating the `$CATALINA_OPTS` environment variable, one can alter Tomcat's JVM configuration. In this example, the JVM is started in server mode and will not use more than 64 megabytes of heap memory. The volumes section configures access to the manager application by overriding files in the docker image with files from this project.

Tomcat takes about 30 seconds or so to boot on a Raspberry Pi 3.
- Once up, navigate to http://pihost/manager/html, where pihost is the address or hostname of your raspberry pi, to access the manager console. Enter _bob_ for the username and _abc123_ for the password when prompted.
- Once at the console, click on the "Server Status" link to view the JVM configuration. Notice, that Tomcat does not use more than 64 MB of heap memory.

Find out more about configuring Tomcat's Manager Application [here](http://pihost:8080/docs/manager-howto.html).

