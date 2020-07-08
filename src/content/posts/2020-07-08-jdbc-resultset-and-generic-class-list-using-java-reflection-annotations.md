---
template: blog-post
title: JDBC ResultSet and Generic Class List Using Java Reflection Annotations
slug: java-jdbc
date: 2020-07-08 14:08
description: java jdbc
---
When working with web projects, the frequency required to query the database and get results is quite high. This involves a lot of code in your DAO classes and will affect readability and connection backlogs.

If we observe closely, its code is static, except for the `Model`<span style="box-sizing: border-box;"> </span>class type and query in it.

For example, if we need Employee details, we will create the code with `Connection`, `Statement`, `ResultSet`, and closing blocks. If we require `Department` details, we need to do the same as before, except we change the `List`<span style="box-sizing: border-box;"> </span>type to `Department`<span style="box-sizing: border-box;"> </span>and the query we run through it.

Based on this experience, I came across one article by [Roberto Benitez](http://baseprogramming.com/blog1/author/administrator/) on this [blog](http://baseprogramming.com/blog1/2017/08/24/automating-jdbc-crud-operations-with-reflection/).

Please go through the above article and come back here to see how I used it to serve the previously discussed problems.

To implement, check out the steps below:

1.  Create a custom annotation
2.  Create a `Model`<span style="box-sizing: border-box;"> </span>class, which contains mapping fields to the `ResultSet` column names with the created annotation.
3.  Call the `ResultSet`<span style="box-sizing: border-box;"> </span>
4.  Load the `ResultSet` for each value into the object
5.  Check for the `Primitive` type
6.  Auto-box the `Primitive`<span style="box-sizing: border-box;"> </span>type class.

### **Create the of Custom Annotation**

Here is how we created the custom annotation:

<div class="CodeMirror cm-s-default" style="box-sizing: border-box; border: 1px solid rgb(217, 220, 221); background: white; font-family: monospace; height: auto !important; color: black; position: relative; overflow: hidden; font-size: 13px; clear: both; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

<div class="CodeMirror-scroll" style="box-sizing: content-box; margin-bottom: 0px; margin-right: -30px; padding-bottom: 30px; height: 98px; outline: none; position: relative; overflow: visible !important; font-family: monospace; font-size: 13px;">

<div class="CodeMirror-sizer" style="box-sizing: content-box; position: relative; border-right: 30px solid transparent; font-family: monospace; font-size: 13px; min-height: 26px; margin-left: 29px; margin-bottom: 0px; min-width: 319px; padding-right: 0px; padding-bottom: 0px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; top: 0px;">

<div class="CodeMirror-lines" style="box-sizing: border-box; padding: 4px 0px; cursor: text; min-height: 1px; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; outline: none;">

<div class="CodeMirror-code" style="box-sizing: border-box; outline: none; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-meta" style="box-sizing: border-box; color: rgb(85, 85, 85);">@Retention</span>(<span class="cm-variable" style="box-sizing: border-box;">RetentionPolicy</span>.<span class="cm-variable" style="box-sizing: border-box;">RUNTIME</span>)</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">@interface</span> <span class="cm-def" style="box-sizing: border-box; color: rgb(0, 0, 255);">DBTable</span> {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">String</span> <span class="cm-variable" style="box-sizing: border-box;">columnName</span>();</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

### **Creation of the Model Class**

Here is how we created the `Model`<span style="box-sizing: border-box;"> </span>class, which contains mapping fields for the `ResultSet` column names with the created annotation:

<div class="CodeMirror cm-s-default" style="box-sizing: border-box; border: 1px solid rgb(217, 220, 221); background: white; font-family: monospace; height: auto !important; color: black; position: relative; overflow: hidden; font-size: 13px; clear: both; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

<div class="CodeMirror-scroll" style="box-sizing: content-box; margin-bottom: 0px; margin-right: -30px; padding-bottom: 30px; height: 170px; outline: none; position: relative; overflow: visible !important; font-family: monospace; font-size: 13px;">

<div class="CodeMirror-sizer" style="box-sizing: content-box; position: relative; border-right: 30px solid transparent; font-family: monospace; font-size: 13px; min-height: 26px; margin-left: 29px; margin-bottom: 0px; min-width: 319px; padding-right: 0px; padding-bottom: 0px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; top: 0px;">

<div class="CodeMirror-lines" style="box-sizing: border-box; padding: 4px 0px; cursor: text; min-height: 1px; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; outline: none;">

<div class="CodeMirror-code" style="box-sizing: border-box; outline: none; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-variable" style="box-sizing: border-box;">Class</span> <span class="cm-variable" style="box-sizing: border-box;">Employee</span>{</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-meta" style="box-sizing: border-box; color: rgb(85, 85, 85);">@DBTable</span>(<span class="cm-variable" style="box-sizing: border-box;">columnName</span> <span class="cm-operator" style="box-sizing: border-box;">=</span><span class="cm-string" style="box-sizing: border-box; color: rgb(170, 17, 17);">"emp_id"</span>)</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">private</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">int</span> <span class="cm-variable" style="box-sizing: border-box;">empId</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-meta" style="box-sizing: border-box; color: rgb(85, 85, 85);">@DBTable</span>(<span class="cm-variable" style="box-sizing: border-box;">columnName</span> <span class="cm-operator" style="box-sizing: border-box;">=</span><span class="cm-string" style="box-sizing: border-box; color: rgb(170, 17, 17);">"emp_name"</span>)</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">private</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">String</span> <span class="cm-variable" style="box-sizing: border-box;">empName</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-comment" style="box-sizing: border-box; color: rgb(170, 85, 0);">//Getters and setters</span></span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-comment" style="box-sizing: border-box; color: rgb(170, 85, 0);">//Default constructor // mandatory</span></span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

### **Calling the ResultSet**

 `String query =”select emp_id,emp_name from employee” `<span style="box-sizing: border-box;"> </span>  
In the above query, column names should be equal to the `@DBTable` annotation column names.

<div class="CodeMirror cm-s-default" style="box-sizing: border-box; border: 1px solid rgb(217, 220, 221); background: white; font-family: monospace; height: auto !important; color: black; position: relative; overflow: hidden; font-size: 13px; clear: both; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

<div class="CodeMirror-scroll" style="box-sizing: content-box; margin-bottom: 0px; margin-right: -30px; padding-bottom: 30px; height: 296px; outline: none; position: relative; overflow: visible !important; font-family: monospace; font-size: 13px;">

<div class="CodeMirror-sizer" style="box-sizing: content-box; position: relative; border-right: 30px solid transparent; font-family: monospace; font-size: 13px; min-height: 26px; margin-left: 29px; margin-bottom: 0px; min-width: 319px; padding-right: 0px; padding-bottom: 0px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; top: 0px;">

<div class="CodeMirror-lines" style="box-sizing: border-box; padding: 4px 0px; cursor: text; min-height: 1px; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; outline: none;">

<div class="CodeMirror-code" style="box-sizing: border-box; outline: none; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-operator" style="box-sizing: border-box;"><</span><span class="cm-variable" style="box-sizing: border-box;">T</span><span class="cm-operator" style="box-sizing: border-box;">></span> <span class="cm-variable" style="box-sizing: border-box;">List</span><span class="cm-operator" style="box-sizing: border-box;"><</span><span class="cm-variable" style="box-sizing: border-box;">T</span><span class="cm-operator" style="box-sizing: border-box;">></span> <span class="cm-def" style="box-sizing: border-box; color: rgb(0, 0, 255);">selectQuery</span>(<span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><</span><span class="cm-variable" style="box-sizing: border-box;">T</span><span class="cm-operator" style="box-sizing: border-box;">></span> <span class="cm-variable" style="box-sizing: border-box;">type</span>, <span class="cm-variable" style="box-sizing: border-box;">query</span>) <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">throws</span> <span class="cm-variable" style="box-sizing: border-box;">SQLException</span> {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">List</span><span class="cm-operator" style="box-sizing: border-box;"><</span><span class="cm-variable" style="box-sizing: border-box;">T</span><span class="cm-operator" style="box-sizing: border-box;">></span> <span class="cm-variable" style="box-sizing: border-box;">list</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">new</span> <span class="cm-variable" style="box-sizing: border-box;">ArrayList</span><span class="cm-operator" style="box-sizing: border-box;"><</span><span class="cm-variable" style="box-sizing: border-box;">T</span><span class="cm-operator" style="box-sizing: border-box;">></span>();</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">try</span> (<span class="cm-variable" style="box-sizing: border-box;">Connection</span> <span class="cm-variable" style="box-sizing: border-box;">conn</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">dataSource</span>.<span class="cm-variable" style="box-sizing: border-box;">getConnection</span>()) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">try</span> (<span class="cm-variable" style="box-sizing: border-box;">Statement</span> <span class="cm-variable" style="box-sizing: border-box;">stmt</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">conn</span>.<span class="cm-variable" style="box-sizing: border-box;">createStatement</span>()) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">try</span> (<span class="cm-variable" style="box-sizing: border-box;">ResultSet</span> <span class="cm-variable" style="box-sizing: border-box;">rst</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">stmt</span>.<span class="cm-variable" style="box-sizing: border-box;">executeQuery</span>(<span class="cm-variable" style="box-sizing: border-box;">query</span>)) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">while</span> (<span class="cm-variable" style="box-sizing: border-box;">rst</span>.<span class="cm-variable" style="box-sizing: border-box;">next</span>()) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">T</span> <span class="cm-variable" style="box-sizing: border-box;">t</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">type</span>.<span class="cm-variable" style="box-sizing: border-box;">newInstance</span>();</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">loadResultSetIntoObject</span>(<span class="cm-variable" style="box-sizing: border-box;">rst</span>, <span class="cm-variable" style="box-sizing: border-box;">t</span>);<span class="cm-comment" style="box-sizing: border-box; color: rgb(170, 85, 0);">// Point 4</span></span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">list</span>.<span class="cm-variable" style="box-sizing: border-box;">add</span>(<span class="cm-variable" style="box-sizing: border-box;">t</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">catch</span> (<span class="cm-variable" style="box-sizing: border-box;">InstantiationException</span> <span class="cm-operator" style="box-sizing: border-box;">|</span> <span class="cm-variable" style="box-sizing: border-box;">IllegalAccessException</span> <span class="cm-variable" style="box-sizing: border-box;">e</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">throw</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">new</span> <span class="cm-variable" style="box-sizing: border-box;">RuntimeException</span>(<span class="cm-string" style="box-sizing: border-box; color: rgb(170, 17, 17);">"Unable to get the records: "</span> <span class="cm-operator" style="box-sizing: border-box;">+</span> <span class="cm-variable" style="box-sizing: border-box;">e</span>.<span class="cm-variable" style="box-sizing: border-box;">getMessage</span>(), <span class="cm-variable" style="box-sizing: border-box;">e</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-variable" style="box-sizing: border-box;">list</span>;</span></pre>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

### Loading the ResultSet for Each Value Into the Object

This looks quite interesting. Here's how Java annotations help us:

<div class="CodeMirror cm-s-default" style="box-sizing: border-box; border: 1px solid rgb(217, 220, 221); background: white; font-family: monospace; height: auto !important; color: black; position: relative; overflow: hidden; font-size: 13px; clear: both; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

<div class="CodeMirror-scroll" style="box-sizing: content-box; margin-bottom: 0px; margin-right: -30px; padding-bottom: 30px; height: 278px; outline: none; position: relative; overflow: visible !important; font-family: monospace; font-size: 13px;">

<div class="CodeMirror-sizer" style="box-sizing: content-box; position: relative; border-right: 30px solid transparent; font-family: monospace; font-size: 13px; min-height: 26px; margin-left: 29px; margin-bottom: 0px; min-width: 319px; padding-right: 0px; padding-bottom: 0px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; top: 0px;">

<div class="CodeMirror-lines" style="box-sizing: border-box; padding: 4px 0px; cursor: text; min-height: 1px; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; outline: none;">

<div class="CodeMirror-code" style="box-sizing: border-box; outline: none; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">static</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">void</span> <span class="cm-def" style="box-sizing: border-box; color: rgb(0, 0, 255);">loadResultSetIntoObject</span>(<span class="cm-variable" style="box-sizing: border-box;">ResultSet</span> <span class="cm-variable" style="box-sizing: border-box;">rst</span>, <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Object</span> <span class="cm-variable" style="box-sizing: border-box;">object</span>)</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">throws</span> <span class="cm-variable" style="box-sizing: border-box;">IllegalArgumentException</span>, <span class="cm-variable" style="box-sizing: border-box;">IllegalAccessException</span>, <span class="cm-variable" style="box-sizing: border-box;">SQLException</span> {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><?></span> <span class="cm-variable" style="box-sizing: border-box;">zclass</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">object</span>.<span class="cm-variable" style="box-sizing: border-box;">getClass</span>();</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">for</span> (<span class="cm-variable" style="box-sizing: border-box;">Field</span> <span class="cm-variable" style="box-sizing: border-box;">field</span> : <span class="cm-variable" style="box-sizing: border-box;">zclass</span>.<span class="cm-variable" style="box-sizing: border-box;">getDeclaredFields</span>()) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">field</span>.<span class="cm-variable" style="box-sizing: border-box;">setAccessible</span>(<span class="cm-atom" style="box-sizing: border-box; color: rgb(34, 17, 153);">true</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">DBTable</span> <span class="cm-variable" style="box-sizing: border-box;">column</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">field</span>.<span class="cm-variable" style="box-sizing: border-box;">getAnnotation</span>(<span class="cm-variable" style="box-sizing: border-box;">DBTable</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Object</span> <span class="cm-variable" style="box-sizing: border-box;">value</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">rst</span>.<span class="cm-variable" style="box-sizing: border-box;">getObject</span>(<span class="cm-variable" style="box-sizing: border-box;">column</span>.<span class="cm-variable" style="box-sizing: border-box;">columnName</span>());</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><?></span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">field</span>.<span class="cm-variable" style="box-sizing: border-box;">getType</span>();</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">isPrimitive</span>(<span class="cm-variable" style="box-sizing: border-box;">type</span>)) {<span class="cm-comment" style="box-sizing: border-box; color: rgb(170, 85, 0);">//check primitive type(Point 5)</span></span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><?></span> <span class="cm-variable" style="box-sizing: border-box;">boxed</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">boxPrimitiveClass</span>(<span class="cm-variable" style="box-sizing: border-box;">type</span>);<span class="cm-comment" style="box-sizing: border-box; color: rgb(170, 85, 0);">//box if primitive(Point 6)</span></span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">value</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-variable" style="box-sizing: border-box;">boxed</span>.<span class="cm-variable" style="box-sizing: border-box;">cast</span>(<span class="cm-variable" style="box-sizing: border-box;">value</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-variable" style="box-sizing: border-box;">field</span>.<span class="cm-variable" style="box-sizing: border-box;">set</span>(<span class="cm-variable" style="box-sizing: border-box;">object</span>, <span class="cm-variable" style="box-sizing: border-box;">value</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

### Check for Primitive Types

This will return a `Primitive`<span style="box-sizing: border-box;"> </span>type:

<div class="CodeMirror cm-s-default" style="box-sizing: border-box; border: 1px solid rgb(217, 220, 221); background: white; font-family: monospace; height: auto !important; color: black; position: relative; overflow: hidden; font-size: 13px; clear: both; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

<div class="CodeMirror-scroll" style="box-sizing: content-box; margin-bottom: 0px; margin-right: -30px; padding-bottom: 30px; height: 80px; outline: none; position: relative; overflow: visible !important; font-family: monospace; font-size: 13px;">

<div class="CodeMirror-sizer" style="box-sizing: content-box; position: relative; border-right: 30px solid transparent; font-family: monospace; font-size: 13px; min-height: 26px; margin-left: 29px; margin-bottom: 0px; min-width: 319px; padding-right: 0px; padding-bottom: 0px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; top: 0px;">

<div class="CodeMirror-lines" style="box-sizing: border-box; padding: 4px 0px; cursor: text; min-height: 1px; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; outline: none;">

<div class="CodeMirror-code" style="box-sizing: border-box; outline: none; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">static</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">boolean</span> <span class="cm-def" style="box-sizing: border-box; color: rgb(0, 0, 255);">isPrimitive</span>(<span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><?></span> <span class="cm-variable" style="box-sizing: border-box;">type</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">int</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span> <span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">long</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span> <span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">double</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span> <span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">float</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span></span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">boolean</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span> <span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">byte</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span> <span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">char</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span> <span class="cm-operator" style="box-sizing: border-box;">||</span> <span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">short</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

### Auto-Boxing to Primitive Type Class

Here is the code for how we implemented this:

<div class="CodeMirror cm-s-default" style="box-sizing: border-box; border: 1px solid rgb(217, 220, 221); background: white; font-family: monospace; height: auto !important; color: black; position: relative; overflow: hidden; font-size: 13px; clear: both; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;">

<div class="CodeMirror-scroll" style="box-sizing: content-box; margin-bottom: 0px; margin-right: -30px; padding-bottom: 30px; height: 404px; outline: none; position: relative; overflow: visible !important; font-family: monospace; font-size: 13px;">

<div class="CodeMirror-sizer" style="box-sizing: content-box; position: relative; border-right: 30px solid transparent; font-family: monospace; font-size: 13px; min-height: 26px; margin-left: 29px; margin-bottom: 0px; min-width: 319px; padding-right: 0px; padding-bottom: 0px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; top: 0px;">

<div class="CodeMirror-lines" style="box-sizing: border-box; padding: 4px 0px; cursor: text; min-height: 1px; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative; outline: none;">

<div class="CodeMirror-code" style="box-sizing: border-box; outline: none; font-family: monospace; font-size: 13px;">

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">public</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">static</span> <span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><?></span> <span class="cm-def" style="box-sizing: border-box; color: rgb(0, 0, 255);">boxPrimitiveClass</span>(<span class="cm-variable" style="box-sizing: border-box;">Class</span><span class="cm-operator" style="box-sizing: border-box;"><?></span> <span class="cm-variable" style="box-sizing: border-box;">type</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">int</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Integer</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">long</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Long</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">double</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Double</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">float</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Float</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">boolean</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Boolean</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">byte</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Byte</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">char</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Character</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">if</span> (<span class="cm-variable" style="box-sizing: border-box;">type</span> <span class="cm-operator" style="box-sizing: border-box;">==</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">short</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>) {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">return</span> <span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">Short</span>.<span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">class</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">} <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">else</span> {</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-type" style="box-sizing: border-box; color: rgb(0, 136, 85);">String</span> <span class="cm-variable" style="box-sizing: border-box;">string</span> <span class="cm-operator" style="box-sizing: border-box;">=</span> <span class="cm-string" style="box-sizing: border-box; color: rgb(170, 17, 17);">"class '"</span> <span class="cm-operator" style="box-sizing: border-box;">+</span> <span class="cm-variable" style="box-sizing: border-box;">type</span>.<span class="cm-variable" style="box-sizing: border-box;">getName</span>() <span class="cm-operator" style="box-sizing: border-box;">+</span> <span class="cm-string" style="box-sizing: border-box; color: rgb(170, 17, 17);">"' is not a primitive"</span>;</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;"><span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">throw</span> <span class="cm-keyword" style="box-sizing: border-box; color: rgb(119, 0, 136);">new</span> <span class="cm-variable" style="box-sizing: border-box;">IllegalArgumentException</span>(<span class="cm-variable" style="box-sizing: border-box;">string</span>);</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"> <span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

<div style="box-sizing: border-box; font-family: monospace; font-size: 13px; position: relative;">

<pre style="box-sizing: border-box; overflow: visible; font-family: monospace; font-size: 13px; display: block; padding: 0px 4px; margin: 0px; line-height: inherit; word-break: break-all; overflow-wrap: normal; color: inherit; background: transparent; border: 0px solid rgb(204, 204, 204); border-radius: 0px; white-space: pre; z-index: 2; position: relative; -webkit-tap-highlight-color: transparent; font-variant-ligatures: contextual;"><span style="box-sizing: border-box; padding-right: 29px;">}</span></pre>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

</div>

We have made all arrangements to call our magic method with the help of [Roberto Benitez](http://baseprogramming.com/blog1/author/administrator/). Now, we will try to call this method and see the magic behind it all.

## Use Cases

By using`selectQuery`<span style="box-sizing: border-box;">, this annotation can </span>provide whatever class name you want from the list. You just need to provide the query that needs to run against the DB.

**Use case 1**: I want all the details of the student: `selectQuery(Student.class, "SELECT * FROM STUDENT")` 

**Use case 2**: I want all of the details of the Department: `selectQuery(Department.class, "SELECT * FROM DEPARTMENT")`<span style="box-sizing: border-box;"> </span> 

**Use case 3**: I want all of the students in the Department and their enrolled courses. Quite a bit overhead, right? Dont worry too much. The solution is as simple as those mentioned above. Just make the `Model`<span style="box-sizing: border-box;"> </span>class with as many fields as you want, but make sure your query (bit joins) return all fields in the `Model` class.

**Note**: Please make sure the `@columnName` annotation value name of the fields for your Model class and column names of the DB result are the same. Keep whatever name you want for the fields.

**PS**: I am not concerned about the performance, as of now, because I use this as a `Helper` method, which I want to use frequently with less code.

Happy coding!