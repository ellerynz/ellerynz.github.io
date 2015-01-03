---
layout: post
title: Quick Lessons From Trying Out Angular
category: Coding
tags: Angular JS Coffeescript
year: 2014
month: 08
day: 03
summary: A quick post to help out anyone who runs into the same problems with directives that I did.
---

## The 'controller as' syntax with directives
<tl:dr;>Use the damn 'controller as' name before the

<s> Trying to build a directive and use the controller as syntax
The 'controller as' syntax replaces the need for $scope in some cases. It's a pretty handy way to tidy up your code. Lots of the Angular tutorials are slightly outdated and use the $scope syntax with controllers.

This became an issue when trying to implement a basic directive. Directives have access to the controller and, as seen in examples and documentation, the controller is accessed via scopes. So what happens when you're using the controller as syntax?

<t>
Creating a basic controller and directive. The directive should call a function on the controller when a certain event occurs. The controller function is called in the link function of the directive on the scope. The scope should bubble up the function call to the controller. Should.

<a>
<example without controller as syntax>
This is obviously the case when using a scope but what about when using the controller as syntax? For some reason, I expected the directives to work the same and bubble up the function call to a controller. Something like so:
<example with controller as syntx and no method prefix>

<r>
Which didn't work. Logging out the scope from the link function and you can see why.
<log output>
The scope has the controller name but no access to the methods like it did before. All that's needed is to prefix the controller function with the controller name. Y'know, like you do with every other controller function when using the controller as syntax :|
Nice as simple. Just one of those things that took me while to click on.
