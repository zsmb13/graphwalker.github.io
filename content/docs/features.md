---
title: Features of GraphWalker
tags: [introduction, workflow]
keywords: features
sidebar: sidebar
permalink: /features/
toc: false
---

## The feature set of GraphWalker

### Path generation

* From a given set of graphs, generators and stop conditions, GraphWalker will generate a path through the graphs.<br>{: style=""}

  Your Model-Based test design consists of 1 or many graphs. Each graph will have it's own set of generator(s) and stop conditions(s). Only when all stop conditions for all generators in all graphs are fulfilled, the generation of the path is done.
  {: style="color:gray; font-size: 90%"}
  
* The path represents your test, or test case if you will.<br>{: style=""}

  The path is used by your test, to call the corresponding methods or functions in the order that is determined by the path.
  {: style="color:gray; font-size: 90%"}

* The path is series of elements, the elements are pairs of edges and vertices.<br>{: style=""}

  It's like a test script, where an action is always followed by a verification. The path is like:<br>  
  {: style="color:gray; font-size: 90%"}

  |Step|Label           |Element type|
  |----|----------------|------------|
  |1   |Do something    |Edge        |
  |2   |Verify something|Vertex      |
  |3   |Do something    |Edge        |
  |4   |Verify something|Vertex      |
  |5   |Do something    |Edge        |
  |6   |Verify something|Vertex      |
  |:   |:               |:           |
  {: style="color:gray; font-size: 90%"}


* An edge represents an action, a transition.<br>{: style=""}

  An action could be an API call, a button clicked, a timeout. Anything that moves your System Under Test into a new state that you want to verify. But remember, there is no verification going on in the edge. That happens only in the vertex. 
  {: style="color:gray; font-size: 90%"}

* A vertex represents verification(s).<br>{: style=""}

  A verification is where you would have assertions in your code. It' here where you verify that an API call returns the correct values. Or that a button click actually did close a dialog. Or that when the timeout should have occurred, the System Under Test triggered the expected event.*
  {: style="color:gray; font-size: 90%"}


### Offline

<a download="example_1.graphml" href="/content/resources/example_1.graphml"><img src="/images/example_1.png" alt="Model" align="right"/></a>

The path generation is done once. It's not directly connected to any test automation code. The path needs to be stored in some intermediate format, like on a file. Typically, the path is generated from command line, and the output stored on file. The content of the file is then used to drive your test.

**Example**

Given following model [example_1.graphml](/content/resources/example_1.graphml). Let's generate an offline path from it using a random generator and an edge coverage stop condition. The test will start at the element `app_closed`


```
java -jar graphwalker-cli-3.4.0.jar offline --start-element app_closed --model example_1.graphml "random(edge_coverage(100))"
{"currentElementName":"app_closed"}
{"currentElementName":"start_app"}
{"currentElementName":"app_running"}
{"currentElementName":"close_app"}
{"currentElementName":"app_closed"}
{"currentElementName":"start_app"}
{"currentElementName":"app_running"}
{"currentElementName":"open_preferences"}
{"currentElementName":"display_preference"}
{"currentElementName":"close_preferences"}
{"currentElementName":"app_running"}

```

To make the output only print out the element names, we can use the [jq](https://stedolan.github.io/jq/) command, and run following instead:

```
java -jar graphwalker-cli-3.4.0.jar offline --start-element app_closed --model example_1.graphml "random(edge_coverage(100))" | jq -r '.currentElementName'
app_closed
start_app
app_running
close_app
app_closed
start_app
app_running
open_preferences
display_preference
close_preferences
app_running

```

### Online

<a download="ShoppingCart.graphml" href="/content/resources/ShoppingCart.graphml"><img src="/images/amazonShoppingCart_small.png" alt="Model" align="right"/></a>

The path generation is done during the execution of the test, run-time. This means that GraphWalker needs to be embedded in your test automation code. This adds a bit of complexity, but the advantages are:

* No need to handle the intermediate path sequences in files.<br>{: style=""}

  GraphWalker 
  {: style="color:gray; font-size: 90%"}
  
* Your test automation code has direct access to the graphs execution context.<br>{: style=""}

  When GraphWalker generates paths, it keeps all data regarding the generation in an [execution context](https://github.com/GraphWalker/graphwalker-project/blob/b604d282087db9776ebf9c4887a1224dcb642567/graphwalker-core/src/main/java/org/graphwalker/core/machine/ExecutionContext.java). Your test can query for data in your model, and it can even change data in your model.
  {: style="color:gray; font-size: 90%"}
  
  An example is the [Amazon example](https://github.com/GraphWalker/graphwalker-example/tree/master/java-amazon). The code at [ShoppingCartTest.java line 65](https://github.com/GraphWalker/graphwalker-example/blob/1c66dc315fd37ca362e66704d26f194bf3acc6bd/java-amazon/src/main/java/org/graphwalker/ShoppingCartTest.java#L65) shows how the current value for the property `num_of_books` is fetched from the model.
  {: style="color:gray; font-size: 90%"}

## What GraphWalker does not do

### Test execution

GraphWalker does not interact with your system under test. You need some other tool do that. If, for instance, you want to test a web application, you would perhaps use [Selenium](http://www.seleniumhq.org/) to do that. Or, if your target is a mobile app, then [Appium](http://appium.io/) might be you choice.


### Graph editing

Today, the graphs are edited using an external tool called [yEd from yWorks](https://www.yworks.com/products/yed).

A webeditor supporting editing of directed graphs is underway, and is expected to be included in the 4.0 release of GraphWalker.
