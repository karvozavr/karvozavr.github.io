---
layout: post
title:  "Clean Architecture Practical Tips"
date:   2021-01-18 15:00:00 +0300
author: dmitrii-abramov
categories: clean-arhitecture software-development
image: assets/images/posts/clean-architecture-diagram.jpg
---

Big Ball of Mud and spagetti-code "*architectural approach*" increases software maintanance and development costs exponentially. 

There is a salvation — **Clean Architecture** — a wonderful approach to prepare the software to requirements changes, so it would be easy to change and joy to maintain and extend.  

## Domain 

Entities, no frameworks, pure business information.

## Use Cases

A lot of effort comes to create a proper boundary for the use case.

As use cases might be responsible for a very complicated logic, a lot of abstractions should be created. But it's worth it. It's DRY because it's SOLID. It's not YAGNI because it's *required* for the system to be resilient to changes.

```kotlin

/**
 * An abstraction to be used by the controllers to 
 * initiate request and presenters to get it's result. 
 */
interface UseCase {
    fun handleRequest(request: UseCaseRequest): UseCaseResponse
}
```

```kotlin
class UseCaseImpl(private val gateway: SomeEntiryGateway): UseCase {
    fun handleRequest(request: UseCaseRequest): UseCaseResponse {
        // all the buisness logic goes here
    }

    // and here
}
```

Business logic goes also to the classes needed to decompose or generalise the use case logic. These classes MUST NOT depend on the database or the UI or controllers.

Use gateways and repository abstractions to manipulate data. 

Return the result in **the most convinient way for the business logic**. Presenting the result in a nice manner for the UI, HTTP response or whatever else is a concern of a **Presenter**.

## Presenters, Controllers, Gateways

### Gateways 

Gateways implement the entity manipulation interfaces declared in the Domain or Use Cases layer.

Gatways might be not only to the database, but to some external system, using which has a particular business significance, i.e. it's reasonable to include the information about that system to the Use Case layer.

```kotlin
interface OrderGateway {

    fun saveOrder(order: Order): UUID

    fun getOrder(orderId: UUID): Order?

    fun deleteUnpaidOrdersOlderThan(DateTime date)
}
```

### Controllers

Controllers are plug-ins, that receive the input from outside of the application and pass it in appropriate to the Use Cases form throught the Input Boundaries like `RequestHandler<Request, Response>`.

For example, using common framework abstractions, like `@RestController` from Spring Framework, as a Controller in your application might be a pragmatic way, if you follow the dependency rule.

```kotlin
@RestController("/api/v2/orders")
class GetOrderController(
    private val GetOrderUseCaseFactory useCaseFactory;
    private val GetOrderResponsePresenter orderPresenter;
) {

    @GetMapping("/{orderId}")
    fun getOrder(@PathParam orderId: String): OrderResponse {
        validateParams(orderId);

        val requestHandler = getOrderHandlerFactory.createRequestHandler()
        val request = GetOrderRequest(orderId)
        val response = requestHandler.handleRequest(request)

        val orderViewModel = orderPresenter.present(response)
        return OrderResponse(orderViewModel)
    }
}
```

### Presenters

Presenters transform the result of the Use Case to the form of presentation, i.e. the information that needs to be shown on UI or returned in HTTP response body.

> Important! Presenters should tranform `UseCaseResult` to a *ViewModel*. Not to the resulting presentation format. Presenters present response in a form of data structure, not in a form of XML, or HTML, or JSON (if it's not JS/TS app), or Protobuf.

## View (UI, Web, etc.)

View is responsible for presenting information to application users, wether it's web or mobile client, another web service, or the desktop app UI.

View utilises the ViewModel objects to get the data from. View should not care about how to compute or get this data. View just uses convinient data structure of a ViewModel and presents this data to the users in appropriate form (JSON / HTML / Protobuf /Swing component).

Use cases should not know which protocol is used to communicate over WEB (or even not a WEB at all), which UI framework is used to show pretty forms, graphs and images, which database is used to store data.

## Database

The database-centric approach is definitely not the way of a clean architecture.

Data manipulations are abstracted into Gateways, which are then implemented on a DB layer via Mongo driver, JDBC, ORMs etc.

That way there would be a clear answer to the question:

> How do I test delivery price calculation withought my Relational DB running and without guessing the correct responses for my DB Mock calls?

Easily, the delivery price calculation UseCase class would have all the Gateway dependencies in a constructor (Dependency Inversions from SOLID). You wouldn't even need a mocking solution like Mockito at all! Implementing a gateway interface would be easier. So testing the use case would not run any executable code rather then the Use Case implementation itself.

Such test is much more stable over time and reqirement changes. You would not need to change anything in your test, as well as with your use case implementation, if you someday swap this Relational DB with a no-sql cloud solution.

## External systems

Same case as the with the databases. They should be abstracted for the Use Cases to use.

If the external system has an API which requires you to send a request in a form of XML, the should not be any xml libraries in your use case. The layers should communicate in language convinient for the inner layer. 

And transforming the request to the external system that the use case logic created, is a concern of this external system adapter implementation.

___

## Sources
- Robert C Martin - Clean Architecture]]