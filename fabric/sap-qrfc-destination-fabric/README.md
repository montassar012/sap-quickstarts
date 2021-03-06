FABRIC SAP Queued RFC Destination Endpoint Quick Start
===========================================================
**Demonstrates the sap-qrfc-destination component running in a Fabric camel runtime.**  
![SAP Tool Suite](../../sap_tool_suite.png "SAP Tool Suite")

* * *
Author: William Collins - JBoss Fuse Team  
Level: Advanced  
Technologies: Fabric, Camel, SAP  
Summary: This quick start demonstrates how to configure and use the sap-qrfc-destination component in a JBoss Fuse Fabric environment to invoke remote function modules and BAPI methods within SAP. This component invokes remote function modules and BAPI methods within SAP using the *Queued RFC* (qRFC) protocol.  
Target Product: Fuse  
Source: <https://github.com/jboss-fuse/fuse/tree/master/quickstarts/camel-sap>  

* * *

What is it?  
-----------  

This quick start shows how to integrate Apache Camel with SAP using the JBoss Fuse SAP Queued Remote Function Call Destination Camel component. This component extends the capabilities of the JBoss Fuse Transactional Remote Function Call Destination camel component by adding **IN-ORDER** delivery and processing guarantees to the delivery and processing of requests through its endpoints. This component and its endpoints should be used in cases where a series of requests depend on each other and must be delivered to and processed by the receiving SAP system in the same order that they were sent (**AT-MOST-ONCE** and **IN-ORDER**). The component accomplishes the AT-MOST-ONCE delivery guarantees using the same mechanisms as the JBoss Fuse SAP Transactional Remote Function Call Destination Camel component. The ordering guarantee is accomplished by serializing the requests in the order they are received by the SAP system to an *Inbound Queue*. Inbound queues are processed by the **QIN Scheduler** within SAP. When the inbound queue is *activated*, the QIN Scheduler which execute in order the queue's requests.   

This quick start contains a route with an initial timer endpoint which triggers and executes that route once. The route uses processor beans to build requests to the `CreateFromData` method of the `FlightCustomer` BAPI to create flight customer records in SAP. These requests are routed to `sap-qrfc-destination` endpoints which use the qRFC protocol to send these requests to SAP to invoke the BAPI method. The route logs to the console the serialized contents of the request messages it sends. These requests are serializes within SAP on the inbound queue `QUICKSTARTQUEUE`.  

**NOTE:** The qRFC protocol used by this component is asynchronous and does not return a response and thus the endpoints of this component do not return a response message.    

In studying this quick start you will learn:

* How to define, build and deploy a JBoss Fuse Fabric profile that configures a Fabric container to use the JBoss Fuse SAP Queued Remote Function Call Destination Camel component
* How to define a Camel route containing the JBoss Fuse SAP Queued Remote Function Call Destination Camel component using the Blueprint XML syntax.
* How to use the JBoss Fuse SAP Queued Remote Function Call Destination Camel component to reliably update data in SAP. 
* How to configure connections used by the component.

For more information see:

* <https://access.redhat.com/documentation/en/red-hat-jboss-fuse/6.3/paged/apache-camel-component-reference/chapter-138-sap-component> for more information about the JBoss Fuse SAP Camel components 
* <https://access.redhat.com/documentation/en/red-hat-jboss-fuse/> for more information about using JBoss Fuse

System requirements
-------------------

Before building and running this quick start you will need:

* Maven 3.0.4 or higher
* JDK 1.7 or 1.8
* A JBoss Fuse 6.3.0 container running within a Fabric
* SAP JCo3 and IDoc3 libraries (sapjco3.jar, sapidoc3.jar and JCo native library for your OS platform)
* SAP instance with [Flight Data Application](http://help.sap.com/saphelp_erp60_sp/helpdata/en/db/7c623cf568896be10000000a11405a/content.htm) setup.

Configuring the QuickStart for your environment
-----------------------------------------------

To configure the quick start for your environment:

1. Deploy the JCo3 library jar and native library (for your platform) and IDoc3 library jar to a network accessible location in your environment.
2. Edit the profile configuration file (`src/main/fabric8/io.fabric8.agent.properties`) in the quick start and set the network locations of the SAP libraries in your environment:

>lib.sapjco3.jar=http://host/path/to/library/sapjco3.jar  
>lib.sapidoc3.jar=http://host/path/to/library/sapidoc3.jar  
>lib.sapjco3.nativelib=http://host/path/to/library/\<native-lib\>  

3. Ensure that the **SAP Instance Configuration Configuration Parameters** in the parent pom.xml file (`../../.pom.xml`) of quick starts project has been set to match the connection configuration for your SAP instance.  

Build and Run the Quickstart
----------------------------

To build and run the quick start:

1. Change your working directory to the quick start directory `sap-qrfc-destination-fabric`.  
2. Run `mvn clean install` to build the quick start.  
3. In your JBoss Fuse installation directory run, `./bin/fuse` to start the JBoss Fuse runtime.  
4. It is assumed that you have already created a fabric and logged into a container called `root` from the JBoss Fuse console.  
5. In the quick start directory run `mvn fabric8:deploy` to upload the quick start's profile (`sap-qrfc-destination-fabric`) to the fabric container.  
6. Create a new child container and deploy the `sap-qrfc-destination-fabric` profile in a single step, by entering the
 following command in the JBoss Fuse console:  

        fabric:container-create-child --profile sap-qrfc-destination-fabric root mychild  

7. Wait for the new child container, `mychild`, to start up. Use the `fabric:container-list` command to check the status of the `mychild` container and wait until the `[provision status]` is shown as `success`.  

	**Note:** You may need to restart the `mychild` container to successfully provision it.  

8. Log into the `mychild` container using the `fabric:container-connect` command, as follows:  

		fabric:container-connect mychild  
  
9. In the `mychild` container's JBoss Fuse console, run `log:tail` to monitor the container's log.  
10. In the container's log observe the requests sent by the endpoint.  
11. Execute the queued requests waiting in the inbound queue `QUICKSTARTQUEUE`. Using the SAP GUI, run transaction `SMQ2`, the Inbound Queue qRFC Monitor:  
    a. Select the `QUICKSTARTQUEUE` queue.  
    b. Display the queue contents (Edit > Display Selection).  
    c. Select the entry for your Client connection and activate the queue (Edit > Activate).  
12. Using the SAP GUI, run transaction `SE16`, Data Browser, and display the contents of the table `SCUSTOM`.  
13. Search the table (Edit > Find..) for the newly created Customer records: `Fred Flintstone`, `Wilma Flintstone`, `Barney Rubble`, and `Betty Rubble`.   

Stopping and Uninstalling the Quickstart
----------------------------------------

To uninstall the quick start and stop the JBoss Fuse run-time perform the following in the JBoss Fuse console:

1. Disconnect from the child container by typing `Ctrl-D` at the console prompt.
2. Stop and delete the child container by entering the following command at the JBoss Fuse console:

		fabric:container-stop mychild
		fabric:container-delete mychild

3. Delete the quick start's profile by entering the following command at the JBoss Fuse console: 

		fabric:profile-delete sap-qrfc-destination-fabric

4. Run `osgi:shutdown -f` to shutdown the JBoss Fuse runtime.

