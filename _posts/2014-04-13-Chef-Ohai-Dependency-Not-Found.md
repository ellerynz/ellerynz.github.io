---
layout: post
title: Chef Ohai dependency not found
category: Coding
tags: Chef
year: 2014
month: 04
day: 13
summary: Having trouble configuring Knife with Chef version 11.12.2
---

TL;DR Install chef version 11.10.4

Install the [Chef](http://www.getchef.com/chef/) gem; ``` gem install chef ``` . Then try to configure Knife 

``` $ knife configure --initial ``` 

``` > WARNING: No knife configuration file found ``` 

``` > Where should I put the config file? [/Users/ellery/.chef/knife.rb] ``` 

``` > ERROR: Ohai::Exceptions::DependencyNotFound: Can not find a plugin for dependency os ``` 

Weird. It seems that Ohai is the problem. Looking at Chef on [rubygems](https://rubygems.org/gems/chef), we can see that Chef 11.12.2 depends on Ohai 7.0. However, the [Ohai](http://docs.opscode.com/ohai.html) page says that version 6 is the current version before Chef 11.12.0. Now to check if using Ohai 6 will fix the issue.

Back on rubygems, the latest version of Chef that doesn't need Ohai 7 is Chef 11.10.4. So ```gem uninstall chef```, remove the executables, and then ```gem install chef -v 11.10.4```. Once that has run, ```knife configure --initial``` will begin prompting you for the Chef server URL and other settings. In other words, it works!