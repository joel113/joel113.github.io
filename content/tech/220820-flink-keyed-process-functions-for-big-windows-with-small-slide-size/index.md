+++
author = "Johannes Ehm"
title = "Flink Keyed Process Function"
date = "2022-08-23"
description = "A brief introduction to the keyed process functions of flink"
tags = [
    "flink",
		"streaming"
]
draft = true
+++

A process function is low level stream operation which gives access to events, state and timers. I did discover the keyed process functions when I was reading a [stackoverflow post](https://stackoverflow.com/questions/65681792/optimization-of-flink-windows-with-small-slide-size-and-same-contents/65690116#65690116) which was mentioning keyed process functions as alternatives to [windows](https://nightlies.apache.org/flink/flink-docs-master/docs/learn-flink/event_driven/). It is straightforward to use windows for aggregation across different time windows but comes with the downside of having a duplication of elements. The implementation of windows which assigns each element to the windows by value. If the sliding window setup uses several overlapping windows, the element is assigned to each of them which can lead to a heavy memory overhead to use the windows to calculate the aggregates across the windows.

## Windowing Implementation

In Flink each API has a unique window implementation. [Flink Streaming has a window implementation](https://github.com/apache/flink/tree/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/windowing). [Flink Table has a window implementation](https://github.com/apache/flink/tree/master/flink-table/flink-table-runtime/src/main/java/org/apache/flink/table/runtime/operators/window). In the Streaming API, the [SlidingEventTimeWindows]() and the [SlidingProcessingTimeWindows]() assigns the elements to windows based on the timestamp of the elements, based on the API of the [WindowAssigner](https://github.com/apache/flink/blob/master/flink-table/flink-table-runtime/src/main/java/org/apache/flink/table/runtime/operators/window/assigners/WindowAssigner.java). Hence, independent whether it is streaming or table, both APIs follow the interfaces of the flink-table-runtime. In the Streaming API, [the WindowOperator]() is actually using either the SlidingEventWindows() or the SlidingProcessingTimeWindows() implementation:

```java
final Collection<W> elementWindows =
	windowAssigner.assignWindows(
  	element.getValue(), element.getTimestamp(), windowAssignerContext);

...
windowState.setCurrentNamespace(window);
windowState.add(element.getValue());
```

Independent whether it is a sliding window based on the event time or based on the processing time, [the WindowOperator]() does add the element to the state. Depending on the windows, adding the element to the state does happen more than once.

## Window Aggregation Techniques

An optimization to calculate the aggregates of windows is to store partial aggregates (cmp. [Scotty: Efficient Window Aggregation for out-of-order Stream Processing](). These optimization techniques come with the additional cost of increasing the memory consumption. Using slices can reduce the memory consumption of having partial aggregates.

https://nightlies.apache.org/flink/flink-docs-master/api/java/org/apache/flink/table/runtime/operators/window/slicing/SlicingWindowOperator.html

https://www.ververica.com/blog/high-throughput-low-latency-and-exactly-once-stream-processing-with-apache-flink

https://stackoverflow.com/questions/64566540/apache-flink-incremental-window-computation

https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/operators/overview/

## Keyed Process Functions

According to the [flink documentation](https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/operators/process_function/) the KeyedProcessFunction gives additionally to the key of timers in its onTimer(...) method. The timers are managed by the timer service and do belong to the streaming context. The timers are different from the other time related context information like the current timestamp (watermark in event time, processing time or ingestion time (cmp. [Timers Management in Flink](https://medium.com/@elevate.global/timers-management-in-apache-flink-ca36c0b4c39a). The timer service is only available for keyed streams. Hence, a timer is provided with a specific key. A timer does trigger the onTimer(...) implementetion whenever the timer is due. In a KeyedProcessFunction, the onTimer(...) implementation has access to the key.

With events, the streaming context including state and timers, the process functions and more specifically the keyed process functions are powerful tools to implement streaming operations. Still, I am wondering how a process function might be an alternative to the using sliding windows if aggregation across several overlapping windows should be performed. The flink documentation provides [an example for implementing pseudo windows in a fashion of an event-driven application using a keyed process function](https://nightlies.apache.org/flink/flink-docs-master/docs/learn-flink/event_driven/). The example has a state where the currently aggregated state is kept. It is implemented in the `open()` method. The example has an implementation for arriving elements. The implementation can be found in the `processElement()` method. It updates the state (a sum of tips for taxi driver). The updated sum of tips is passed to the state using the end of the window based on the event time. Hence, each sum of tips as aggregation result is added to a single window only once. Additionally, a timer is set for the completion of a timer. The implementation of the `onTimer`(...)` method does fetch the sum of tips state as the window aggregation result and pushes the result downstream like a regular window would do. Additionally, it drops the window. Hence, the example is a perfectly fine pseudo implementation of a single window. However, there is still the question, whether the keyed process function can also be used as a more efficient implementation of sliding windows. 

In a sliding window setup, several windows overlap each other:

|--- w06 ---|
--|--- w07 ---|
 ---|--- w08 ---|
 w03 ---|--- w09 ---|
-- w04 ---|--- w10 ---|
|--- w05 ---|--- w11 ---|

...x1..x2......x3.....x4.......

In the case of the example, following assignments do take place:

x1 -> w02, w03, w04, w05, w06, w07
x2 -> w03, w04, w05, w06, w07, w08
x3 -> w08, w09, w10, w11
x4 -> w10, w11

Hence, a lot of duplication takes place. The goal of using this setup is to have an aggregation across the most current events in an event driven manner, while having an elemination of outdated events. Hence, multiple events are duplicated to simplify the aggregation excluding outdated events. If there a lot of overlapping windows (i.e. long window size with small slices), there is clearly an operational burden to this approach. As an opposite approach, simply a single window could be used to calculation a single aggregation across the events of the windows and to substract an event if it is supposed to be become retired or to simply recalculate the aggregation depending whether what is possible and what is feasible.

Hence, in a single aggregation window setup, there is only a single window at a single point of time:

|--- w01 ----|

...x1..x2....

In the case of the example, following assignments do take place:

x1 -> w01
x2 -> w02

When w01 is done, the aggregation of w01 can be computed, can be put downstream and w01 can be garbage collected:

--|--- w01 ----|

If an element is older, than the current window it can be remove from the state, it can be removed from the aggregation result or a recomputation can take place. If an event is added to the state using the currently valid end time of a window, when it was added to the window, it should be fairly simple to update the state and having an aggregation result which can be forwarded downstream. Hence, if [the example from the stackoverflow to compute the sum of tips](https://nightlies.apache.org/flink/flink-docs-master/docs/learn-flink/event_driven/) should support sliding windows, following adaptions would be needed.

## The open() method



## The processElement() method

## The onTimer() method

## References

https://stackoverflow.com/questions/65681792/optimization-of-flink-windows-with-small-slide-size-and-same-contents/65690116#65690116

https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/operators/process_function/

https://medium.com/@elevate.global/timers-management-in-apache-flink-ca36c0b4c39a

https://nightlies.apache.org/flink/flink-docs-master/docs/learn-flink/event_driven/

https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/runtime/operators/windowing/WindowOperator.java

https://github.com/apache/flink/blob/master/flink-streaming-java/src/main/java/org/apache/flink/streaming/api/windowing/assigners/WindowAssigner.java

https://www.user.tu-berlin.de/powibol/assets/publications/traub-scotty-icde-2018.pdf
