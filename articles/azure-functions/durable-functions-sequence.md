---
title: Function chaining in Durable Functions - Azure
description: Learn how to run a Durable Functions sample that executes a sequence of functions.
services: functions
author: cgillum
manager: cfowler
editor: ''
tags: ''
keywords:
ms.service: functions
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 09/29/2017
ms.author: cgillum
---

# Function chaining in Durable Functions - Hello sequence sample

Function chaining refers to the pattern of executing a sequence of functions in a particular order. Often the output of one function needs to be applied to the input of another function. This article explains a sample that uses [Durable Functions](durable-functions-overview.md) to implement function chaining.

## Prerequisites

* Follow the instructions in [Install Durable Functions](durable-functions-install.md) to set up the sample.

## The functions

This article explains the following functions in the sample app:

* `E1_HelloSequence`: An orchestrator function that calls `E1_SayHello` multiple times in a sequence. It stores the outputs from the `E1_SayHello` calls and records the results.
* `E1_SayHello`: An activity function that prepends a string with "Hello".

The following sections explain the configuration and code that are used for Azure portal development. The code for Visual Studio development is shown at the end of the article.
 
## function.json file

If you use the Azure portal for development, here's the content of the *function.json* file for the orchestrator function. Most orchestrator *function.json* files look almost exactly like this.

[!code-json[Main](~/samples-durable-functions/samples/csx/E1_HelloSequence/function.json)]

The important thing is the `orchestrationTrigger` binding type. All orchestrator functions must use this trigger type.

> [!WARNING]
> To abide by the "no I/O" rule of orchestrator functions, don't use any input or output bindings when using the `orchestrationTrigger` trigger binding.  If other input or output bindings are needed, they should instead be used in the context of `activityTrigger` functions.

## C# script

Here is the source code:

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E1_HelloSequence/run.csx)]

All C# orchestration functions must have a `DurableOrchestrationContext` parameter, which exists in the `Microsoft.Azure.WebJobs.Extensions.DurableTask` assembly. If you're using C# script, the assembly can be referenced using the `#r` notation. This context object lets you call other *activity* functions and pass input parameters using its [CallActivityAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CallActivityAsync_) method.

The code calls `E1_SayHello` three times in sequence with different parameter values. The return value of each call is added to the `outputs` list, which is returned at the end of the function.

The *function.json* file for the activity function `E1_SayHello` is similar to that of `E1_HelloSequence` except that it uses an `activityTrigger` binding type instead of an `orchestrationTrigger` binding type.

[!code-json[Main](~/samples-durable-functions/samples/csx/E1_SayHello/function.json)]

> [!NOTE]
> Any function called by an orchestration function must use the `activityTrigger` binding.

The implementation of `E1_SayHello` is a relatively trivial string formatting operation.

[!code-csharp[Main](~/samples-durable-functions/samples/csx/E1_SayHello/run.csx)]

This function has a [DurableActivityContext](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableActivityContext.html)parameter, which it uses to get input that was passed to it by the orchestrator function's call to [CallActivityAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html#Microsoft_Azure_WebJobs_DurableOrchestrationContext_CallActivityAsync_)>.

## Running the orchestration

To execute the `E1_HelloSequence` orchestration, make the following HTTP call.

```
POST http://{app-name}.azurewebsites.net/orchestrators/E1_HelloSequence
```
The result is an HTTP 202 response, like this (trimmed for brevity):

```
HTTP/1.1 202 Accepted
Content-Length: 719
Content-Type: application/json; charset=utf-8
Location: http://{host}/admin/extensions/DurableTaskExtension/instances/96924899c16d43b08a536de376ac786b?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}

(...trimmed...)
```

At this point, the orchestration is queued up and begins to run immediately. The URL in the `Location` header can be used to check the status of the execution.

```
GET http://{host}/admin/extensions/DurableTaskExtension/instances/96924899c16d43b08a536de376ac786b?taskHub=DurableFunctionsHub&connection=Storage&code={systemKey}
```

The result is the status of the orchestration. It runs and completes quickly, so you see it in the *Completed* state with a response that looks like this (trimmed for brevity):

```
HTTP/1.1 200 OK
Content-Length: 179
Content-Type: application/json; charset=utf-8

{"runtimeStatus":"Completed","input":null,"output":["Hello Tokyo!","Hello Seattle!","Hello London!"],"createdTime":"2017-06-29T05:24:57Z","lastUpdatedTime":"2017-06-29T05:24:59Z"}
```

As you can see, the `runtimeStatus` of the instance is *Completed* and the `output` contains the JSON-serialized result of the orchestrator function execution.

> [!NOTE]
> The HTTP POST endpoint that started the orchestrator function is implemented in the sample app as an HTTP trigger function named "HttpStart". You can implement similar starter logic for other trigger types, like `queueTrigger`, `eventHubTrigger`, or `timerTrigger`.

Look at the function execution logs. The `E1_HelloSequence` function started and completed multiple times due to the replay behavior described in the [overview](durable-functions-overview.md). On the other hand, there were only three executions of `E1_SayHello` since those function executions do not get replayed.

## Visual Studio sample code

Here is the orchestration as a single C# file in a Visual Studio project:

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HelloSequence.cs)]

## Next steps

At this point, you have a basic understanding of the core mechanics for Durable Functions. This sample was trivial and only showed a few of the features available. Subsequent samples are more "real world" and display a greater breadth of functionality.

> [!div class="nextstepaction"]
> [Run the Fan-out/fan-in sample](durable-functions-cloud-backup.md)
