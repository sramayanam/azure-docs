---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 11/15/2021
ms.author: glenga
---

Use the function trigger to respond to an event sent to an event hub event stream. You must have read access to the underlying event hub to set up the trigger. When the function is triggered, the message passed to the function is typed as a string. 

## Example

::: zone pivot="programming-language-csharp"

# [In-process](#tab/in-process)

The following example shows a [C# function](../articles/azure-functions/functions-dotnet-class-library.md) that logs the message body of the Event Hubs trigger.

```csharp
[FunctionName("EventHubTriggerCSharp")]
public void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] string myEventHubMessage, ILogger log)
{
    log.LogInformation($"C# function triggered to process a message: {myEventHubMessage}");
}
```

To get access to [event metadata](#event-metadata) in function code, bind to an [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) object. You can also access the same properties by using binding expressions in the method signature.  The following example shows both ways to get the same data:

```csharp
[FunctionName("EventHubTriggerCSharp")]
public void Run(
    [EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData myEventHubMessage,
    DateTime enqueuedTimeUtc,
    Int64 sequenceNumber,
    string offset,
    ILogger log)
{
    log.LogInformation($"Event: {Encoding.UTF8.GetString(myEventHubMessage.Body)}");
    // Metadata accessed by binding to EventData
    log.LogInformation($"EnqueuedTimeUtc={myEventHubMessage.SystemProperties.EnqueuedTimeUtc}");
    log.LogInformation($"SequenceNumber={myEventHubMessage.SystemProperties.SequenceNumber}");
    log.LogInformation($"Offset={myEventHubMessage.SystemProperties.Offset}");
    // Metadata accessed by using binding expressions in method parameters
    log.LogInformation($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.LogInformation($"SequenceNumber={sequenceNumber}");
    log.LogInformation($"Offset={offset}");
}
```

To receive events in a batch, make `string` or `EventData` an array.  

> [!NOTE]
> When receiving in a batch you cannot bind to method parameters like in the above example with `DateTime enqueuedTimeUtc` and must receive these from each `EventData` object  

```cs
[FunctionName("EventHubTriggerCSharp")]
public void Run([EventHubTrigger("samples-workitems", Connection = "EventHubConnectionAppSetting")] EventData[] eventHubMessages, ILogger log)
{
    foreach (var message in eventHubMessages)
    {
        log.LogInformation($"C# function triggered to process a message: {Encoding.UTF8.GetString(message.Body)}");
        log.LogInformation($"EnqueuedTimeUtc={message.SystemProperties.EnqueuedTimeUtc}");
    }
}
```
# [Isolated process](#tab/isolated-process)

The following example shows a [C# function](../articles/azure-functions/dotnet-isolated-process-guide.md) that is triggered based on an event hub, where the input message string is written to the logs:

:::code language="csharp" source="~/azure-functions-dotnet-worker/samples/Extensions/EventHubs/EventHubsFunction.cs" range="12-23":::

# [C# Script](#tab/csharp-script)

The following example shows an Event Hubs trigger binding in a *function.json* file and a [C# script function](../articles/azure-functions/functions-reference-csharp.md) that uses the binding. The function logs the message body of the Event Hubs trigger.

The following examples show Event Hubs binding data in the *function.json* file for Functions runtime version 2.x and later versions. 

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Here's the C# script code:

```cs
using System;

public static void Run(string myEventHubMessage, TraceWriter log)
{
    log.Info($"C# function triggered to process a message: {myEventHubMessage}");
}
```

To get access to [event metadata](#event-metadata) in function code, bind to an [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) object. You can also access the same properties by using binding expressions in the method signature.  The following example shows both ways to get the same data:

```cs
#r "Microsoft.Azure.EventHubs"

using System.Text;
using System;
using Microsoft.ServiceBus.Messaging;
using Microsoft.Azure.EventHubs;

public void Run(EventData myEventHubMessage,
    DateTime enqueuedTimeUtc,
    Int64 sequenceNumber,
    string offset,
    TraceWriter log)
{
    log.Info($"Event: {Encoding.UTF8.GetString(myEventHubMessage.Body)}");
    log.Info($"EnqueuedTimeUtc={myEventHubMessage.SystemProperties.EnqueuedTimeUtc}");
    log.Info($"SequenceNumber={myEventHubMessage.SystemProperties.SequenceNumber}");
    log.Info($"Offset={myEventHubMessage.SystemProperties.Offset}");

    // Metadata accessed by using binding expressions
    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"SequenceNumber={sequenceNumber}");
    log.Info($"Offset={offset}");
}
```

To receive events in a batch, make `string` or `EventData` an array:

```cs
public static void Run(string[] eventHubMessages, TraceWriter log)
{
    foreach (var message in eventHubMessages)
    {
        log.Info($"C# function triggered to process a message: {message}");
    }
}
```
---

::: zone-end 
::: zone pivot="programming-language-javascript"  

The following example shows an Event Hubs trigger binding in a *function.json* file and a [JavaScript function](../articles/azure-functions/functions-reference-node.md) that uses the binding. The function reads [event metadata](#event-metadata) and logs the message.

The following example shows an Event Hubs binding data in the *function.json* file, which is different for version 1.x of the Functions runtime compared to later versions. 

# [Functions 2.x+](#tab/functionsv2)

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

# [Functions 1.x](#tab/functionsv1)

```json
{
  "type": "eventHubTrigger",
  "name": "myEventHubMessage",
  "direction": "in",
  "path": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```
---

Here's the JavaScript code:

```javascript
module.exports = function (context, myEventHubMessage) {
    context.log('Function triggered to process a message: ', myEventHubMessage);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('SequenceNumber =', context.bindingData.sequenceNumber);
    context.log('Offset =', context.bindingData.offset);

    context.done();
};
```

To receive events in a batch, set `cardinality` to `many` in the *function.json* file, as shown in the following examples.

# [Functions 2.x+](#tab/functionsv2)

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

# [Functions 1.x](#tab/functionsv1)

```json
{
  "type": "eventHubTrigger",
  "name": "eventHubMessages",
  "direction": "in",
  "path": "MyEventHub",
  "cardinality": "many",
  "connection": "myEventHubReadConnectionAppSetting"
}
```
---

Here's the JavaScript code:

```javascript
module.exports = function (context, eventHubMessages) {
    context.log(`JavaScript eventhub trigger function called for message array ${eventHubMessages}`);

    eventHubMessages.forEach((message, index) => {
        context.log(`Processed message ${message}`);
        context.log(`EnqueuedTimeUtc = ${context.bindingData.enqueuedTimeUtcArray[index]}`);
        context.log(`SequenceNumber = ${context.bindingData.sequenceNumberArray[index]}`);
        context.log(`Offset = ${context.bindingData.offsetArray[index]}`);
    });

    context.done();
};
```

::: zone-end  
::: zone pivot="programming-language-powershell" 
 
Complete PowerShell examples are pending.
::: zone-end 
::: zone pivot="programming-language-python"  

The following example shows an Event Hubs trigger binding in a *function.json* file and a [Python function](../articles/azure-functions/functions-reference-python.md) that uses the binding. The function reads [event metadata](#event-metadata) and logs the message.

The following examples show Event Hubs binding data in the *function.json* file.

```json
{
  "type": "eventHubTrigger",
  "name": "event",
  "direction": "in",
  "eventHubName": "MyEventHub",
  "connection": "myEventHubReadConnectionAppSetting"
}
```

Here's the Python code:

```python
import logging
import azure.functions as func


def main(event: func.EventHubEvent):
    logging.info(f'Function triggered to process a message: {event.get_body().decode()}')
    logging.info(f'  EnqueuedTimeUtc = {event.enqueued_time}')
    logging.info(f'  SequenceNumber = {event.sequence_number}')
    logging.info(f'  Offset = {event.offset}')

    # Metadata
    for key in event.metadata:
        logging.info(f'Metadata: {key} = {event.metadata[key]}')
```

::: zone-end
::: zone pivot="programming-language-java"

The following example shows an Event Hubs trigger binding which logs the message body of the Event Hubs trigger.

```java
@FunctionName("ehprocessor")
public void eventHubProcessor(
  @EventHubTrigger(name = "msg",
                  eventHubName = "myeventhubname",
                  connection = "myconnvarname") String message,
       final ExecutionContext context )
       {
          context.getLogger().info(message);
 }
```

 In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the `EventHubTrigger` annotation on parameters whose value comes from the event hub. Parameters with these annotations cause the function to run when an event arrives.  This annotation can be used with native Java types, POJOs, or nullable values using `Optional<T>`.


The following example illustrates extensive use of `SystemProperties` and other Binding options for further introspection of the Event along with providing a well-formed `BlobOutput` path that is Date hierarchical.

```java
package com.example;
import java.util.Map;
import java.time.ZonedDateTime;

import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

/**
 * Azure Functions with Event Hub trigger.
 * and Blob Output using date in path along with message partition ID
 * and message sequence number from EventHub Trigger Properties
 */
public class EventHubReceiver {

    @FunctionName("EventHubReceiver")
    @StorageAccount("bloboutput")
                                
    public void run(
            @EventHubTrigger(name = "message",
                eventHubName = "%eventhub%",
                consumerGroup = "%consumergroup%",
                connection = "eventhubconnection",
                cardinality = Cardinality.ONE)
            String message,
            
            final ExecutionContext context,
            
            @BindingName("Properties") Map<String, Object> properties,
            @BindingName("SystemProperties") Map<String, Object> systemProperties,
            @BindingName("PartitionContext") Map<String, Object> partitionContext,
            @BindingName("EnqueuedTimeUtc") Object enqueuedTimeUtc,

            @BlobOutput(
                name = "outputItem",
                path = "iotevents/{datetime:yy}/{datetime:MM}/{datetime:dd}/{datetime:HH}/" +
                       "{datetime:mm}/{PartitionContext.PartitionId}/{SystemProperties.SequenceNumber}.json")
            OutputBinding<String> outputItem) {

        var et = ZonedDateTime.parse(enqueuedTimeUtc + "Z"); // needed as the UTC time presented does not have a TZ
                                                             // indicator
        context.getLogger().info("Event hub message received: " + message + ", properties: " + properties);
        context.getLogger().info("Properties: " + properties);
        context.getLogger().info("System Properties: " + systemProperties);
        context.getLogger().info("partitionContext: " + partitionContext);
        context.getLogger().info("EnqueuedTimeUtc: " + et);

        outputItem.setValue(message);
    }
}

```

::: zone-end

::: zone pivot="programming-language-csharp"
## Attributes

Both [in-process](../articles/azure-functions/functions-dotnet-class-library.md) and [isolated worker process](../articles/azure-functions/dotnet-isolated-process-guide.md) C# libraries use attribute to configure the trigger. C# script instead uses a function.json configuration file.

# [In-process](#tab/in-process)

In [C# class libraries](../articles/azure-functions/functions-dotnet-class-library.md), use the [EventHubTriggerAttribute], which supports the following properties.

| Parameters | Description|
|---------|----------------------|
|**EventHubName** | The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. Can be referenced in [app settings](../articles/azure-functions/functions-bindings-expressions-patterns.md#binding-expressions---app-settings), like `%eventHubName%` |
|**ConsumerGroup** | An optional property that sets the [consumer group](../articles/event-hubs/event-hubs-features.md#event-consumers) used to subscribe to events in the hub. When omitted, the `$Default` consumer group is used. |
|**Connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

# [Isolated process](#tab/isolated-process)

Use the `EventHubTriggerAttribute` to define a trigger on an event hub, which supports the following properties.

| Parameters | Description|
|---------|----------------------|
|**EventHubName** | The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. Can be referenced in [app settings](../articles/azure-functions/functions-bindings-expressions-patterns.md#binding-expressions---app-settings), like `%eventHubName%` |
|**ConsumerGroup** | An optional property that sets the [consumer group](../articles/event-hubs/event-hubs-features.md#event-consumers) used to subscribe to events in the hub. When omitted, the `$Default` consumer group is used. |
|**Connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. To learn more, see [Connections](#connections).|

# [C# Script](#tab/csharp-script)

|function.json property | Description|
|---------|----------------------|
|**type** |  Must be set to `eventHubTrigger`. This property is set automatically when you create the trigger in the Azure portal.|
|**direction** |  Must be set to `in`. This property is set automatically when you create the trigger in the Azure portal. |
|**name** |  The name of the variable that represents the event item in function code. |
|**eventHubName** | Functions 2.x and higher. The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. Can be referenced via [app settings](../articles/azure-functions/functions-bindings-expressions-patterns.md#binding-expressions---app-settings) `%eventHubName%`. In version 1.x, this property is named `path`. |
|**consumerGroup** |An optional property that sets the [consumer group](../articles/event-hubs/event-hubs-features.md#event-consumers) used to subscribe to events in the hub. If omitted, the `$Default` consumer group is used. |
|**connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. See [Connections](#connections).|

---

::: zone-end  
::: zone pivot="programming-language-java"  
## Annotations

In the [Java functions runtime library](/java/api/overview/azure/functions/runtime), use the [EventHubTrigger](/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger) annotation, which supports the following settings:

+ [name](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.name)
+ [dataType](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.datatype)
+ [eventHubName](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.eventhubname)
+ [connection](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput.connection)
+ [cardinality](/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger.cardinality)
+ [consumerGroup](/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger.consumergroup)

::: zone-end 
::: zone pivot="programming-language-javascript,programming-language-python,programming-language-powershell"  

## Configuration

The following table explains the trigger configuration properties that you set in the *function.json* file, which differs by runtime version.

# [Functions 2.x+](#tab/functionsv2)

|function.json property | Description|
|---------|----------------------|
|**type** |  Must be set to `eventHubTrigger`. This property is set automatically when you create the trigger in the Azure portal.|
|**direction** |  Must be set to `in`. This property is set automatically when you create the trigger in the Azure portal. |
|**name** |  The name of the variable that represents the event item in function code. |
|**eventHubName** | The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. Can be referenced via [app settings](../articles/azure-functions/functions-bindings-expressions-patterns.md#binding-expressions---app-settings) `%eventHubName%` |
|**consumerGroup** |An optional property that sets the [consumer group](../articles/event-hubs/event-hubs-features.md#event-consumers) used to subscribe to events in the hub. If omitted, the `$Default` consumer group is used. |
|**cardinality** | Set to `many` in order to enable batching.  If omitted or set to `one`, a single message is passed to the function.|
|**connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. See [Connections](#connections).|

# [Functions 1.x](#tab/functionsv1)

|function.json property | Description|
|---------|----------------------|
|**type** |  Must be set to `eventHubTrigger`. This property is set automatically when you create the trigger in the Azure portal.|
|**direction** |  Must be set to `in`. This property is set automatically when you create the trigger in the Azure portal. |
|**name** |  The name of the variable that represents the event item in function code. |
|**path** | The name of the event hub. When the event hub name is also present in the connection string, that value overrides this property at runtime. |
|**consumerGroup** |An optional property that sets the [consumer group](../articles/event-hubs/event-hubs-features.md#event-consumers) used to subscribe to events in the hub. If omitted, the `$Default` consumer group is used. |
|**cardinality** | Set to `many` in order to enable batching.  If omitted or set to `one`, a single message is passed to the function.|
|**connection** | The name of an app setting or setting collection that specifies how to connect to Event Hubs. See [Connections](#connections).|

---

::: zone-end

[!INCLUDE [app settings to local.settings.json](../articles/azure-functions/../../includes/functions-app-settings-local.md)]

## Usage

To learn more about how Event Hubs trigger and IoT Hub trigger scales, see [Event Hubs trigger](../articles/azure-functions/event-driven-scaling.md#event-hubs-trigger).

::: zone pivot="programming-language-csharp"  
The parameter type supported by the Event Hubs output binding depends on the Functions runtime version, the extension package version, and the C# modality used. 

# [Extension v5.x+](#tab/extensionv5/in-process)

In-process C# class library functions supports the following types:

+ [Azure.Messaging.EventHubs.EventData](/dotnet/api/azure.messaging.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

This version of [EventData](/dotnet/api/azure.messaging.eventhubs.eventdata) drops support for the legacy `Body` type in favor of [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody).

# [Extension v3.x+](#tab/extensionv3/in-process)

In-process C# class library functions supports the following types:

+ [Microsoft.Azure.EventHubs.EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

# [Extension v5.x+](#tab/extensionv5/isolated-process)

Requires you to define a custom type, or use a string. 

# [Extension v3.x+](#tab/extensionv3/isolated-process)

Requires you to define a custom type, or use a string.

# [Extension v5.x+](#tab/extensionv5/csharp-script)

C# script functions support the following types:

+ [Azure.Messaging.EventHubs.EventData](/dotnet/api/azure.messaging.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

This version of [EventData](/dotnet/api/azure.messaging.eventhubs.eventdata) drops support for the legacy `Body` type in favor of [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody).

# [Extension v3.x+](#tab/extensionv3/csharp-script)

C# script functions support the following types:

+ [Microsoft.Azure.EventHubs.EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata)
+ String
+ Byte array
+ Plain-old CLR object (POCO)

---

::: zone-end
::: zone pivot="programming-language-java"  
The parameter type can be one of the following:

+ Any native Java types such as int, String, byte[].
+ Nullable values using Optional.
+ Any POJO type.

To learn more, see the [EventHubTrigger](/java/api/com.microsoft.azure.functions.annotation.eventhubtrigger) reference.

::: zone-end

## Event metadata

The Event Hubs trigger provides several [metadata properties](../articles/azure-functions/./functions-bindings-expressions-patterns.md). Metadata properties can be used as part of binding expressions in other bindings or as parameters in your code. The properties come from the [EventData](/dotnet/api/microsoft.servicebus.messaging.eventdata) class.

|Property|Type|Description|
|--------|----|-----------|
|`PartitionContext`|[PartitionContext](/dotnet/api/microsoft.servicebus.messaging.partitioncontext)|The `PartitionContext` instance.|
|`EnqueuedTimeUtc`|`DateTime`|The enqueued time in UTC.|
|`Offset`|`string`|The offset of the data relative to the event hub partition stream. The offset is a marker or identifier for an event within the Event Hubs stream. The identifier is unique within a partition of the Event Hubs stream.|
|`PartitionKey`|`string`|The partition to which event data should be sent.|
|`Properties`|`IDictionary<String,Object>`|The user properties of the event data.|
|`SequenceNumber`|`Int64`|The logical sequence number of the event.|
|`SystemProperties`|`IDictionary<String,Object>`|The system properties, including the event data.|

See [code examples](#example) that use these properties earlier in this article.

[EventHubTriggerAttribute]: /dotnet/api/microsoft.azure.webjobs.eventhubtriggerattribute
