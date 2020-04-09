# Introduction

![](.gitbook/assets/rsz_2vectorstock_21277268.png)

## About

[_Ecotone_](https://github.com/ecotoneframework/ecotone) provides support for well known [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/) \(EIP\), to enable message driven architecture in PHP. On top of that follow an architectural pattern which is based on the principles of Domain-Driven Design \(DDD\) and Command Query Responsibility Segregation \(CQRS\).

It supports routing and transformation of messages so that different transports and different data formats can be integrated without impacting business logic.   
In other words, the messaging and integration concerns are handled by _Ecotone_.   
Business components are further isolated from the infrastructure, so developers can focus on business problems and are relieved of complex integration responsibilities.

{% hint style="info" %}
_Ecotone_ is heavily inspired by _Java_ frameworks [_Spring Integration_](https://spring.io/projects/spring-integration) __and [_Axon Framework_](https://docs.axoniq.io/reference-guide/)_, built around messaging concepts._
{% endhint %}

One of the central tenets of the _Ecotone_ is the idea that you should not be forced to introduce framework-specific classes and interfaces into your business/domain model. However, in some places the Ecotone does give you the option to introduce _Ecotone-specific dependencies_ as it might be just plain easier to read or code some specific piece of functionality in such a way. The _Ecotone_ \(almost\) always offers you the choice though.

## Support Ecotone

If you want to help building and improving `Ecotone` consider becoming a sponsor:

* [Github](https://github.com/sponsors/dgafka)
* [Patreon](https://www.patreon.com/dgafka)

