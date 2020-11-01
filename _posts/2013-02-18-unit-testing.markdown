---
layout: post
title: Unit (resource/frontend) testing template
date: '2013-02-18T18:23:00.002+01:00'
author: Koen Serneels
img: 2.png
tags:
- Java SE
modified_time: '2013-02-18T18:30:14.770+01:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-161971091390855449
blogger_orig_url: http://koenserneels.blogspot.com/2013/02/unit-testing.html
---

Unit testing requires a lot of time and investment. One of the important things to safe keep this investment is building on good infrastructure. While we are all used to design production code, designing our tests and test infrastructure is in many cases forgotten or the time needed is simply not there. In many cases the requirements for the test set also grow (just like real requirements). Sometimes testing can even become a project inside the project. I have gathered some testing practices I've became used with over the years and put them in a small project, which you can find here:  <a target="_blank" href="https://github.com/koen-serneels/unit-testing">github unit-testing</a>

It is a Spring/Spring MVC webapp using JPA (Hibernate as provider) for persistence and TestNG, Spring test support, selenium2 for testing. There are 3 types of test covered: basic unit test, unit resource test and front end test (unit resource tests with a flavor). 

The basic unit tests give an idea how you can use Spring within your tests or use its MVC test extensions to test for example application controllers . The unit resources tests give a strategy for testing code that is depended on a resource, but cannot be fully tested without that resource (eg. dummies, mocks or stubs are not the best alternative). For example repositories depending on a RDBMS. In that case we work with an embedded database which allows to test our repositories very close to their production environment. 

The front end tests require some extra features as they need to communicate with the database booted inside the deployed application. Also, to make it interesting I have put a repository that depends on an "not further specified" sub system for which no in memory equivalents exists. A stub is put into place in the deployed application which is remotely controllable from within the test.  This allows you to have the data setup from the test for both resources (the one requiring a database and this one requiring the sub system) *in* the test. The data for both resources is setup and removed in scope of each test method (or tat least each test).  Alternatives likes simulators have the data deployed together with the application which makes it brittle for changes. (eg. the data is automatically shared for all tests and packed together in one place) 

The project is Maven enabled; importing it should be zero hassle. Surefire and failsafe (+cargo) are also configured, so all your tests (including the front end tests) are executed when performing a build with Maven. Since there are no dependencies whatsoever (besides the one defined in the pom) you can add this kind of project directly to your CI hassle-free.

There is also a PDF explaining some of the ideas, but it was put together in short time, so don't expect too much from it. Feel free to share your testing strategies!
