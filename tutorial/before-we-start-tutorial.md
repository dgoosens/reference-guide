# Before we start tutorial

## What are we building?

The best way to get started with `Ecotone` is to use it to build something realistic.  
We will build a small back-end for shopping system during this tutorial. The techniques you’ll learn in the tutorial are fundamental to building any application using `Ecotone`. 

In doing so, you'll learn the architectural concepts behind the software and start to learn its capabilities.  You will learn how to build types of applications that may have seemed impossible or really, really hard to make prior to learning _Ecotone_.



The tutorial is divided into several sections:

* Lesson 1, we will learn **the fundamentals** of _Ecotone_: Endpoints, Messages, Channels, Gateways and using Command Query Responsibility Segregation \(CQRS\) on top of that
* Lesson 2,  we will learn using **Tactical** [**Domain Driven Design \(DDD\)**](../modelling/modelling-1.md): Aggregates, Repositories and also Event Handlers
* Lesson 3, we will learn **how to use Converters**
* Lesson 4, we will learn about **Metadata and Method Invocation**
* Lesson 5, we will learn about **Interceptors**, to handle cross cutting concerns
* Lesson 6, we we will learn about **Scheduling And Asynchronous** Endpoints

You don’t have to complete all of the lessons at once to get the value out of this tutorial.   
You may start benefit from the tutorial even if it’s one or two lessons.

## Setup for tutorial

Depending on your preferences, you may choose tutorial using [`Symfony`](https://symfony.com/) or [`Laravel`](https://laravel.com/).  
In places where configuration will differ, there will be two tabs available each with code specific to the framework.

1. Use [git](https://git-scm.com) to download  **a starting point** to follow the tutorial. Tutorial is simple Command Line based application.

{% tabs %}
{% tab title="Symfony" %}
```
git clone git@github.com:ecotoneframework/quickstart-symfony.git
# Go to quickstart-symfony catalog
```
{% endtab %}

{% tab title="Laravel" %}
```bash
git clone git@github.com:ecotoneframework/quickstart-laravel.git
# Go to quickstart-laravel catalog

# Normally you will use "php artisan" for running console commands
# To reduce number of difference "artisan" is changed to "bin/console"
```
{% endtab %}
{% endtabs %}

2. Run command line application. There are two options, run on your _Local Environment_ or using _Docker_

{% tabs %}
{% tab title="Local Environment" %}
```php
/** You need to have atleast PHP 7.3 and Composer installed */
1. Run "composer install" 
2. Run starting command "bin/console ecotone:quickstart"
3. You should see:
"Running example...
Hello World
Good job, scenario ran with success!"
```
{% endtab %}

{% tab title="Docker" %}
```php
/** Ecotone Quickstart ships with docker-compose with preinstalled PHP 7.4 */
1. Run "docker-compose up -d" // It will automatically download composer packages
2. Enter container "docker exec -it ecotone-quickstart /bin/bash"
3. Run starting command "bin/console ecotone:quickstart"
4. You should see:
"Running example...
Hello World
Good job, scenario ran with success!"
```
{% endtab %}
{% endtabs %}

{% hint style="success" %}
Great, we are prepared for Lesson 1!
{% endhint %}

{% page-ref page="lesson-1-messaging-concepts.md" %}

