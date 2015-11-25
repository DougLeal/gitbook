# Kura Camel quickstart

The Kura Camel quickstart can be used to create Camel router OSGi bundle project deployable into the
[Eclipse Kura](https://www.eclipse.org/kura) gateway. Kura is a widely adopted field gateway software for the
IoT solutions. Rhiot supports Kura gateway deployments as a first class citizen and this quickstart is intended to be
used as a blueprint for the Camel deployments for Kura. It uses [Camel Kura component](http://camel.apache.org/kura.html)
under the hood.

#### Creating a Kura Camel project

In order to create the Kura Camel project execute the following commands:

    git clone git@github.com:rhiot/quickstarts.git
    cp -r quickstarts/kura-camel kura-camel
    cd kura-camel
    mvn install

#### Prerequisites

We presume that you have Eclipse Kura already installed on your target device. And that you know the IP address of that device.
If you happen to deploy to a Raspbian-based device, and you would like to find the IP of that Raspberry Pi device connected
to your local network, you can use the Rhiot device scanner, as demonstrated on the snippet below:

    docker run --net=host -it rhiot/deploy-gateway scan

The command above will return an output similar to the one presented below:

    Scanning local networks for devices...

    ======================================
    Device type		IPv4 address
    --------------------------------------
    RaspberryPi2		/192.168.1.100

Keep in mind that `/opt/eclipse/kura/kura/config.ini` file on your target device should have OSGi boot delegation
enabled for packages `sun.*,com.sun.*`. Your `/opt/eclipse/kura/kura/config.ini` should contain the following line then:

    org.osgi.framework.bootdelegation=sun.*,com.sun.*

A boot delegation of `sun` packages is required to make Camel work smoothly in an Equinox.

#### Deployment

In order to deploy Camel application to a Kura server, you have to copy necessary Camel jars and a bundle containing your
 application. Your bundle can be deployed into the target device by executing an `scp` command. For example:

    scp target/rhiot-kura-camel-1.0.0-SNAPSHOT.jar pi@192.168.1.100:/tmp

The command above will copy your bundle to the `/tmp/rhiot-kura-camel-1.0.0-SNAPSHOT.jar` location on a target device.
Use similar `scp` command to deploy Camel jars required to run your project:

    scp ~/.m2/repository/org/apache/camel/camel-core/2.16.0/camel-core-2.16.0.jar pi@192.168.1.100:/tmp
    scp ~/.m2/repository/org/apache/camel/camel-core-osgi/2.16.0/camel-core-osgi-2.16.0.jar pi@192.168.1.100:/tmp
    scp ~/.m2/repository/org/apache/camel/camel-kura/2.16.0/camel-kura-2.16.0.jar pi@192.168.1.100:/tmp

Now log into your target device Kura shell using telnet:

    telnet localhost 5002

And install the bundles you previously scp-ed:

    install file:///tmp/camel-core-2.16.0.jar
    install file:///tmp/camel-core-osgi-2.16.0.jar
    install file:///tmp/camel-kura-2.16.0.jar
    install file:///tmp/rhiot-kura-camel-1.0.0-SNAPSHOT.jar

Finally start your application using the following command:

    start <ID_OF_rhiot-kura-camel-1.0.0-SNAPSHOT_BUNDLE)

Keep in mind that bundles you deployed using the recipe above are not installed permanently and will be reverted
after the server restart. Please read Kura documentation for more details regarding
[permanent deployments](http://eclipse.github.io/kura/doc/deploying-bundles.html#making-deployment-permanent).

#### What the quickstart is actually doing?

This quickstart triggers [Camel timer](http://camel.apache.org/timer.html) event every second and sends it to the
system logger using [Camel Log](http://camel.apache.org/log) component. This is fairy simple functionality, but enough
to demonstrate the Camel Kura project is actually working and processing messages.