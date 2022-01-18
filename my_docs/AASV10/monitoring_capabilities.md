# Monitoring Capabilities

Monitoring is the process of reviewing the statistics of a system to improve performance or solve problems. By monitoring the state of components and services that are deployed in the Apusic Server, system administrators can identify performance bottlenecks, predict failures, perform root cause analysis, and ensure that everything is functioning as expected. Monitoring data can also be useful in performance tuning and capacity planning.

## Defining Statistics That Are to Be Monitored

To provide statistics to Apusic Server, your component must define events for the operations that generate these statistics. At runtime, your component must send these events when performing the operations for which the events are defined. For example, to enable the number of received requests to be monitored, a component must send a "request received" event each time that the component receives a request.

Defining statistics that are to be monitored involves the following
tasks:

* Defining an Event Provider
* Sending an Event

### Defining an Event Provider

An event provider defines the types of events for the operations that generate statistics for an add-on component.

#### Defining an Event Provider by Writing a Java Class

To identify your class as an event provider, annotate the declaration of the class with the `org.glassfish.external.probe.provider.annotations.ProbeProvider` annotation.

To create a name space for event providers and to uniquely identify an event provider to the monitoring infrastructure of Apusic Server, set the elements of the `@ProbeProvider` annotation as follows:

`moduleProviderName`::
  Your choice of text to identify the application to which the event  provider belongs. The value of the `moduleProviderName` element is not  required to be unique.  For example, for event providers from Apusic Server Open Source  Edition, `moduleProviderName` is `aas`.
`moduleName`::
  Your choice of name for the module for which the event provider is  defined. A module provides significant functionality of an
  application. The value of the `moduleName` element is not required to  be unique. In Apusic Server, examples of module names are `web-container`,  `ejb-container`, `transaction`, and `webservices`.
`probeProviderName`::
  Your choice of name to identify the event provider. To uniquely  identify the event provider, ensure that `probeProviderName` is unique  for all event providers in the same module.  In Apusic Server, examples of event—provider names are `jsp`,
  `servlet`, and `web-module`.

##### Defining Event Types in an Event Provider Class

To define event types in an event provider class, write one method for each type of event that is related to the component. The requirements for each method are as follows:

* The return value of the callback methods must be void.
* The method body must be empty. You instantiate the event provider class in the class that invokes the method to send the event.

Annotate the declaration of each method with the`org.glassfish.external.probe.provider.annotations.Probe` annotation.

By default, **the type of the event is the method name**. If you overload a method in your class, you must uniquely identify the event type for each form of the method. To uniquely identify the event type, set the `name` element of the `@Probe` annotation to the name of the event type.

##### Specifying Event Parameters

To enable methods in an event listener to select a subset of values, annotate each parameter in the method signature with the
`org.glassfish.external.probe.provider.annotations.ProbeParam` annotation. Set the `value` element of the `@ProbeParam` annotation to the name of the parameter.

##### Example of Defining an Event Provider by Writing a Java Class

This example shows the definition of the `TxManager` class. This class
defines events for the start and end of transactions that are performed
by a transaction manager.

The methods in this class are as follows:

`onTxBegin`::
  This method sends an event to indicate the start of a transaction. The
  name of the event type that is associated with this method is `begin`.
  A parameter that is named `txId` is passed to the method.
`onCompletion`::
  This method sends an event to indicate the end of a transaction. The
  name of the event type that is associated with this method is `end`. A
  parameter that is named `outcome` is passed to the method.

```java
import org.glassfish.external.probe.provider.annotations.Probe;
import org.glassfish.external.probe.provider.annotations.ProbeParam;
import org.glassfish.external.probe.provider.annotations.ProbeProvider;
@ProbeProvider(moduleProviderName="examplecomponent", moduleName="transaction", probeProviderName="manager")
public class TxManager {

    @Probe("begin")
    public void onTxBegin(
        @ProbeParam("{txId}") String txId
    ){}
    
    @Probe ("end")
    public void onCompletion(
        @ProbeParam("{outcome}") boolean outcome
    ){}
 }
```

##### Packaging a Component's Event Providers

Packaging a component's event providers enables the monitoring infrastructure of Apusic Server to discover the event providers automatically.

To package a component's event providers, add an entry to the component's `META-INF/MANIFEST.MF` file that identifies all of the component's event providers. The format of the entry depends on how the event providers are defined:

* If the event providers are defined as Java classes, the entry is a list of the event providers' class names as follows: 
```
probe-provider-class-names : class-list
```
The `class-list` is a comma-separated list of the fully qualified Java class names of the component's event providers.

### Sending an Event

At runtime, your add-on component might perform an operation that generates statistics. To provide statistics about the operation to Apusic Server, your component must send an event of the correct type when performing the operation.

To send an event, **instantiate your event provider class and invoke the method** of the event provider class for the type of the event. Instantiate the class and invoke the method in the class that represents your add-on component. Ensure that the method is invoked when your component performs the operation for which the event was defined. One way to meet this requirement is to invoke the method for sending the event in the body of the method for performing the operation.

Example 5-4 Sending an Event
This example shows the code for instantiating the `TxManager` class and invoking the `onTxBegin` method to send an event of type `begin`. This event indicates that a component is about to begin a transaction.
The `TxManager` class is instantiated in the constructor of the `TransactionManagerImpl` class. To ensure that the event is sent at the correct time, the `onTxBegin` method is invoked in the body of the `begin` method, which starts a transaction.

The declaration of the `onTxBegin` method in the event provider interface is shown .

```java
public class TransactionManagerImpl {
...
     public TransactionManagerImpl() {
         TxManager txProvider = new TxManager();
         ...
     }
    ...
    public void begin() {
        String txId = createTransactionId();
        ....
        txProvider.onTxBegin(txId); //emit
      }
...
}
```

## Updating the Monitorable Object Tree
A monitorable object is a component, subcomponent, or service that can be monitored. Apusic Server uses a tree structure to track monitorable objects.

Because the tree is dynamic, the tree changes as components of the Apusic Server instance are added, modified, or removed. Objects are also added to or removed from the tree in response to configuration changes. For example, if monitoring for a component is turned off, the component's monitorable object is removed from the tree.

To enable system administrators to access statistics for all components in the same way, you must provide statistics for an add-on component by updating the monitorable object tree. Statistics for the add-on component are then available through the Apusic Server administrative commands. These commands locate an object in the tree through the object's dotted name.

To make an add-on component a monitorable object, you must add the add-on component to the monitorable object tree.

To update the statistics for an add-on component, you must add the statistics to the monitorable object tree, and create event listeners to gather statistics from events that represent these statistics. At runtime, these listeners must update monitorable objects with statistics that these events contain. The events are sent by event provider classes. 

Updating the monitorable object tree involves the following tasks:

* link:#ghpni[Creating Event Listeners]
* link:#ghptp[Representing a Component's Statistics in an Event Listener Class]
* link:#ghpml[Subscribing to Events From Event Provider Classes]
* link:#ghppo[Registering an Event Listener]

### Creating Event Listeners

An event listener gathers statistics from events that an event provider sends. To enable an add-on component to gather statistics from events, create listeners to receive events from the event provider. The listener can receive events from the add-on component in which the listener is created and from other components.

To create an event listener, write a Java class to represent the listener. The listener can be any Java object.

An event listener also represents a component's statistics. To enable the Application Server Management Extensions (AMX) to expose the statistics to client applications, annotate the declaration of the class with the `org.glassfish.gmbal.ManagedObject` annotation.

Ensure that the class that you write meets these requirements:

* The return value of all callback methods in the listener must be void.
* Because the methods of your event provider class may be entered by multiple threads, the listener must be thread safe. However,Apusic Server provides utility classes to perform some common operations such as `count`, `avg`, and `sum`.
* The listener must have the same restrictions as a Java Platform,
Enterprise Edition (Java EE) application. For example, the listener **cannot open server sockets, or create threads.**

A listener is called in the same thread as the event method. As a result, the listener can use thread locals. If the monitored system allows access to thread locals, the listener can access thread locals of the monitored system.

### Representing a Component's Statistics in an Event Listener Class

Represent each statistic as the property of a JavaBeans specification getter method of your listener class. Methods in the listener class for processing events can then access the property through the getter method. 

To enable AMX to expose the statistic to client applications, annotate the declaration of the getter method with the
`org.glassfish.gmbal.ManagedAttribute` annotation. Set the `id` element of the `@ManagedAttribute` annotation to the property name all in lowercase.

The data type of the property that represents a statistic must be a class that provides methods for computing the statistic from event data.
 The `org.glassfish.external.statistics.impl` package provides the following classes to gather and compute statistics data:

`AverageRangeStatisticImpl`::
  Provides standard measurements of the lowest and highest values that  an attribute has held and the current value of the attribute.

`BoundaryStatisticImpl`::
  Provides standard measurements of the upper and lower limits of the  value of an attribute.

`BoundedRangeStatisticImpl`::
  Aggregates the attributes of `RangeStatisticImpl` and  `BoundaryStatisticImpl` and provides standard measurements of a range  that has fixed limits.

`CountStatisticImpl`::

  Provides standard count measurements.

`RangeStatisticImpl`::
  Provides standard measurements of the lowest and highest values that  an attribute has held and the current value of the attribute.

`StatisticImpl`::
  Provides performance data.

`StringStatisticImpl`::
  Provides a string equivalent of a counter statistic.

`TimeStatisticImpl`::
  Provides standard timing measurements.


Example 5-5 Representing a Component's Statistics in an Event Listener
Class

This example shows the code for representing the `txcount` statistic in
the `TxListener` class.

```java
...
import org.glassfish.external.statistics.CountStatistic;
import org.glassfish.external.statistics.impl.CountStatisticImpl;
...
import org.glassfish.gmbal.ManagedAttribute;
import org.glassfish.gmbal.ManagedObject;

...
@ManagedObject
public class TxListener {

    private CountStatisticImpl txCount = new ountStatisticImpl("TxCount",
        "count", "Number of completed transactions");
...
    @ManagedAttribute(id="txcount")
    public CountStatistic  getTxCount(){
         return txCount;
    }
}
```



### Subscribing to Events From Event Provider Classes
 To receive events from event provider classes, a listener must subscribe to the events. Subscribing to events also specifies the provider and the type of events that the listener will receive.

 To subscribe to events from event provider classes, write one method in your listener class to process each type of event. To specify the provider and the type of event, annotate the method with the
`org.glassfish.external.probe.provider.annotations.ProbeListener` annotation. In the `@ProbeListener` annotation, specify the provider and the type as follows:

```
"module-providername:module-name:probe-provider-name:event-type"
```

module-providername::
  The application to which the event provider belongs. This parameter  must be the value of the `moduleProviderName` element or attribute in   the definition of the event provider.

module-name::
  The module for which the event provider is defined. This parameter  must match be the value of the `moduleName` element or attribute in  the definition of the event provider. 

probe-provider-name::
  The name of the event provider. This parameter must match be the value  of the `probeProviderName` element or attribute in the definition of   the event provider. 

event-type::
  The type of the event. This type is defined in the event provider   class. 

Annotate each parameter in the method signature with the `@ProbeParam`annotation. Set the `value` element of the `@ProbeParam` annotation to the name of the parameter.

In the method body, provide the code to update monitoring statistics in response to the event.



**Example 5-6 Subscribing to Events From Event Provider Classes**

This example shows the code for subscribing to events of type `begin` from the `tx` component. The provider of the component is `TxManager`. The body of the `begin` method contains code to increase the transaction count txcount by 1 each time that an event is received.

```java
... 
import org.glassfish.external.probe.provider.annotations.ProbeListener; 
import org.glassfish.external.probe.provider.annotations.ProbeParam; 
import org.glassfish.gmbal.ManagedObject;
...
@ManagedObject 
public class TxListener {    
    private CountStatisticImpl txCount = new ountStatisticImpl("TxCount",
        "count", "Number of completed transactions");
	...
    @ManagedAttribute(id="txcount")
    public CountStatistic  getTxCount(){
         return txCount;
    }
    ...;    
    @ProbeListner("examplecomponent:transaction:manager:begin")
    public void begin(@ProbeParam("{txId}") String txId) {
      txCount.increment();
    }
  }
```



### Registering an Event Listener

Registering an event listener enables the listener to receive callbacks from the Apusic Server event infrastructure. The listener can then collect data from events and update monitorable objects in the object tree. These monitorable objects form the basis for monitoring statistics.

Registering an event listener also makes a component and its statistics monitorable objects by adding statistics for the component to the monitorable object tree.

At runtime, the Apusic Server event infrastructure registers listeners for an event provider when the event provider is started and unregisters them when the event provider is shut down. As a result, listeners have no dependencies on other components.

 To register a listener, invoke the static
`org.glassfish.external.probe.provider.StatsProviderManager.register` method in the class that represents your add-on component. In the method invocation, pass the following information as parameters:

* The name of the configuration element with which all statistics in the event listener are to be associated. System administrators use this element for enabling or disabling monitoring for the event listener.
* The node in the monitorable object tree under which the event listener is to be registered. To specify the node, pass one of the following constants of the
`org.glassfish.external.probe.provider.PluginPointPluginPoint` enumeration:

** To register the listener under the `server/applications` node, pass the `APPLICATIONS` constant.

** To register the listener under the `server` node, pass the `SERVER` constant.
* The path through the monitorable object tree from the node under which the event listener is registered down to the statistics in the event listener. The nodes in this path are separated by the slash (`/`) character.
* The listener object that you are registering.

 

Example 5-7 Registering an Event Listener

This example shows the code for registering the event listener `TxListener` for the add-on component that is represented by the class `TransactionManagerImpl`. The statistics that are defined in this listener are associated with the `web-container` configuration element. The listener is registered under the `server/applications` node. The path from this node to the statistics in the event listener is `tx/txapp`.

 Code for the constructor of the `TxListener` class is beyond the scope of this example.

```java
... 
import org.glassfish.external.probe.provider.StatsProviderManager; 
import org.glassfish.external.probe.provider.PluginPoint
... 
public class TransactionManagerImpl {
...
  StatsProviderManager.register("web-container", PluginPoint.APPLICATIONS, "tx/txapp", new TxListener());
...
}
```
