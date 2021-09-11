---
title: Domain-Driven Design Building Blocks
date: 2020-05-02 15:23:26 Z
categories: [Domain-Driven-Design]
tags: [value-object, entity, aggregate-root, repository, factory, domain-service]
cover: https://raw.githubusercontent.com/BhuwanUpadhyay/ddd-building-blocks/master/assets/featured.png
---

Domain-driven design (DDD), is an approach used to build systems that have a complex business domain.
<!-- more -->

So you wouldn’t apply DDD to, say, infrastructure software or building routers, proxies, or caching layers, 
but instead to business software that solves real-world business problems.
It’s a great technique for separating the way the business is modeled from the plumbing code that ties it all together.
Separating these two in the software itself makes it easier to design, model, build and evolve an implementation over time.
        
In tactical DDD, the building blocks play an important role in how business is modeled into the code.
In this article, I will take you through the best available options for building blocks in object-oriented principles.

## Value Object

> An object that represents a descriptive aspect of the domain with no conceptual identity is called a Value Object. Value Objects are instantiated to represent elements of the design that we care about only for what they are, not who or which they are.
> — **Eric Evans**

In other words, value objects don’t have their own identity. The value object possess concept of structural equality 
— if two objects are equal then they have equivalent content. 
Also, If two value objects have the same set of attributes we can treat them interchangeably.

### Attributes
- No Identity - value objects are identity-less.
- Immutable - value object can be replaced by another value object with same content. To make sure, equality by structure for value object is by using Immutable design especially in multi-thread scenarios (immutable objects are threadsafe by design).  
- Lifespan - can’t exist without a parent entity (should not have separate table in a database).
- Business Constraints - value object is always valid (should need to validate business rules on creation). 
I always prefer to validate business constraints on the `constructor` for the value object.

### Code Example

Equality logic implementation for value object is very important to ensure they are equals by its content.
It’s too easy to forget to override `equals()` and `hashCode()` in the value object.
It's very important to make sure all properties of value object should be part of equality logic. 

To enforce equality logic for value object I used following class as a base class for every value object.  

```java
public abstract class ValueObject {
  @Override
  public abstract int hashCode();

  @Override
  public abstract boolean equals(Object o);
}
```

## Entity
> Many objects are not fundamentally defined by their attributes, but rather by a thread of continuity and identity.
> — **Eric Evans**

The entity possess concept of identifier equality — Two instances of entity would be equal if they have the same identifiers. 
We can change everything related to an entity (except its identifier), and after modification also it remains the same entity.

### Code Example

Identifier equality means that entity class has a field for an identifier. 
Following class can be used as a base class for entity class.

```java
import java.util.Objects;

public abstract class Entity<ID extends ValueObject> {

  public static final String ENTITY_ID_IS_REQUIRED = "EntityIdIsRequired";
  private final ID id;

  public Entity(ID id) {
    DomainAsserts.begin(id).notNull(DomainError.create(this, ENTITY_ID_IS_REQUIRED)).end();
    this.id = id;
  }

  public ID getId() {
    return this.id;
  }

  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || this.getClass() != o.getClass()) return false;
    Entity<?> entity = (Entity<?>) o;
    return Objects.equals(this.id, entity.id);
  }

  @Override
  public int hashCode() {
    return Objects.hash(this.id);
  }
}
```

## Domain Event
> The essence of a Domain Event is that you use it to capture things that can trigger a change to the state of the application you are developing.
> — **Martin Fowler**

A domain event is, something that happened in the domain that you want notify to other parts of the same domain. 
The important benefit of domain events is that side effects can be expressed explicitly.

![](https://raw.githubusercontent.com/BhuwanUpadhyay/ddd-building-blocks/master/assets/aggregate_transaction.png)

### Code Example

According to nature side effects, the published domain event can be listened inside same bounded context or another bounded context.
Following class can be used as a base class for domain event.

```java
import java.util.Objects;
import java.util.UUID;

public abstract class DomainEvent {

  private final String eventId = UUID.randomUUID().toString();

  private final String eventClassName = getClass().getName();

  private final String domainEventType;

  public DomainEvent(DomainEventType domainEventType) {
    DomainAsserts.begin(domainEventType)
        .notNull(DomainError.create(this, "DomainEventTypeIsRequired"))
        .end();
    this.domainEventType = domainEventType.name();
  }

  public String getEventId() {
    return eventId;
  }

  public String getEventClassName() {
    return eventClassName;
  }

  private String getDomainEventType() {
    return domainEventType;
  }

  public boolean isInsideContext() {
    return Objects.equals(DomainEventType.BOTH.name(), this.getDomainEventType())
        || Objects.equals(DomainEventType.INSIDE.name(), this.getDomainEventType());
  }

  public boolean isOutsideContext() {
    return Objects.equals(DomainEventType.BOTH.name(), this.getDomainEventType())
        || Objects.equals(DomainEventType.OUTSIDE.name(), this.getDomainEventType());
  }

  public enum DomainEventType {
    /** Represents domain event is inside same bounded context only. */
    INSIDE,
    /** Represents domain event is only for other bounded context. */
    OUTSIDE,
    /**
     * Represents domain event is available for both i.e. inside same bounded context and other
     * bounded context.
     */
    BOTH
  }
}
```

## Aggregate Root
> An AGGREGATE is a cluster of associated objects that we treat as a unit for the purpose of data changes. Each aggregate has a root and a boundary.
> — **Eric Evans**

- Group the entities and value objects into aggregates and define boundaries around each. 
- Control all access to the objects inside the boundary through the root. 
- Allow external objects to hold references to the root only. 
- Register domain events into the aggregate root. 

### Code Example

The example of base class for aggregate root in my tactical ddd implementation as below.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public abstract class AggregateRoot<ID extends ValueObject> extends Entity<ID> {

  private final transient List<DomainEvent> domainEvents = new ArrayList<>();

  public AggregateRoot(ID id) {
    super(id);
  }

  /**
   * Register domain event
   *
   * @param event domain event
   */
  protected void registerEvent(DomainEvent event) {
    this.domainEvents.add(event);
  }

  /** Clear registered domain events. */
  public void clearDomainEvents() {
    this.domainEvents.clear();
  }

  /** @return list of domain events */
  public List<DomainEvent> getDomainEvents() {
    return Collections.unmodifiableList(domainEvents);
  }
}
```

 
## Repositories

A repository is a service that uses a global interface to provide access to all entities and value objects that are within a particular aggregate collection.

### Code Example
From a repository, we have to publish all domain events when persist the aggregate root and then detached domain events from the aggregate root.
You can use following code to extend your domain repository in your project.

```java
import java.util.Optional;

public abstract class DomainRepository<T extends AggregateRoot<ID>, ID extends ValueObject> {

  private final DomainEventPublisher publisher;

  protected DomainRepository(DomainEventPublisher publisher) {
    this.publisher = publisher;
  }

  public abstract Optional<T> findOne(ID id);

  public void save(T entity) {
    DomainAsserts.begin(entity).notNull(DomainError.create(this, "EntityIsRequired")).end();
    this.persist(entity);
    entity.getDomainEvents().forEach(publisher::publish);
    entity.clearDomainEvents();
  }

  protected abstract void persist(T entity);
}
```

## Domain Service

> Some concepts from the domain aren’t natural to model as objects…a service tends to be named for an activity, rather than an entity — a verb rather than a noun.
> — **Eric Evans**

A service that expresses a business logic that is not part of any Aggregate Root.

> When an operation does not conceptually belong to any object. Following the natural contours of the problem, you can implement these operations in services.
> — **Wikipedia**

Some business rules don't make sense to be part of an Aggregate.  If something is 'outside' an Aggregate, then it's probably is a Domain Service.

## Factories

Factories are often used to create Aggregates.
Aggregates provide encapsulation and a consistent boundary around a group of objects.
Aggregates are important because they enforce the internal consistency of the object they are responsible for.

A Factory can be useful when creating a new Aggregate because it will encapsulate the knowledge required to create an Aggregate in a consistent state and with all invariants enforced.

You can find example on github: [<i class="fab fa-github"></i> Source Code](https://github.com/bhuwanupadhyay/ddd-building-blocks)
