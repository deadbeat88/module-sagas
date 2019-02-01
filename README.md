[![Build Status](https://travis-ci.org/php-service-bus/module-sagas.svg?branch=master)](https://travis-ci.org/php-service-bus/module-sagas)
[![Code Coverage](https://scrutinizer-ci.com/g/php-service-bus/module-sagas/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/php-service-bus/module-sagas/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/php-service-bus/module-sagas/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/php-service-bus/module-sagas/?branch=master)

## Table of contents
* [What is Saga?]()
* [Field of use]()
* [Implementation features]()
* [Caveats]()
* [Configuration]()
* [Lifecycle]()
* [Creation]()
* [Example]()

#### What is Saga?
Saga may be interpreted as any documented business process which consists of steps. Speaking technically, Saga is an Event Listener which listens to some [event](https://github.com/mmasiukevich/service-bus/blob/master/doc/en_messages.md#event) and performs an action based on that event. A good example is a flowchart with a decision symbol.

There are synchronous, asynchronous and mixed sagas (where some steps may be performed synchronously and some asynchronously). From personal experience only asynchronous sagas are worth implementing.

A little bit more on [Saga](https://microservices.io/patterns/data/saga.html).

#### Field of use
* Description of complicated business processes. A good example is electronic document management. Transfer of a document may take some time - up to weeks depending on many factors. The process itself consists of a dozen of steps including electronic signatures of 3 parties (3 party is the operator)
* Distributed transactions (emulated Atomicity): either each step would be finished successfully or a step-specific compensating action will be performed.

#### Implementation features
All the sagas in the framework are asynchronous and have their own state which is serialized and stored in database. Any variables expect closures may be used as saga state.
Any non-closed saga may be triggered starting from any of saga's steps if a corresponding event is received.

#### Caveats
The more sagas (and steps within them) you have the more documentation you need for them. Single saga may trigger other sagas heavily increasing complexity of business processes.
[Message based architecture](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Messaging.html) is very difficult to understand at the beginning (especially after "common" PHP programming) and requires a lot of responsibility.

#### Configuration
Sagas are configures through annotations [@SagaHeader]() and [@SagaEventListener]().
Parameters:
 - ```idClass```: Saga identifier class namespace;
 - ```expireDateModifier```: Saga expiry interval;
 - ```containingIdProperty```: Field (in the event) where saga identifier should be found.
 Each event should be bound to specific saga instance (since one message may be received by different sagas) - that's why there should be an identifier in each of saga events. This parameter may be overridden per event listener class.

 Each event listener should be named like ```on{EventName}```, where *on* is a generic prefix and *{EventName}* is short class name.

 #### Lifecycle
 Saga execution starts on call of method [start()](), which will be called automatically (see example below). There are following methods (protected) available inside Saga instance:
- [fire()](): Dispatches a command;
- [raise()](): Dispatches an event;
- [makeCompleted()](): Closes saga marking it as successfully finished.
- [makeFailed()](): Closes saga marking it as failed.

On saga status change next events will be raised:
- [SagaCreated()](): Saga was created (started);
- [SagaStatusChanged()](): Saga status was changed;
- [SagaClosed()](): Saga was closed;

#### Creation
There is a specific provider created for more convenient operations with sagas - [SagaProvider](). Each of its methods returns [Promise](https://github.com/amphp/amp/blob/master/lib/Promise.php) object.
- [start()](): Creates and starts new saga firing a command;
- [obtain()](): Retrieves a saga instance from database;
- [save()](): Saves all changes in saga state and sends all saga events to transport.

#### Example

```php

```

```

```

On saga creation each saga will be started and saved. It is not necessary to save a saga if there were no changes to its state after start. Saga should be saved if it was obtained from database.

*Saga uniqueness is ensured by a composite key which consists of an UUID and identifier class namespace. Identifier class namespace should be used here to allow launching different sagas with the same identifier (usually all of these are parts of a single business process).*

