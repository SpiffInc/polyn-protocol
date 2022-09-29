# Polyn Protocol (v 0.1.0)

Polyn is a dead simple service framework designed to be language agnostic while
and providing a simple, yet powerful, abstraction layer for building reactive events
based services.

According to [Jonas Boner](http://jonasboner.com/), reactive Microservices require
you to:

1. Follow the principle “do one thing, and one thing well” in defining service
   boundaries
2. Isolate the services
3. Ensure services act autonomously
4. Embrace asynchronous message passing
5. Stay mobile, but addressable
6. Design for the required level of consistency

Polyn implements this pattern in a manner that can be applied to multiple programming
languages, such as Ruby, Elixir, or Python, enabling you to build services that can
communicate regardless of the language you user.

## Protocol Documents

* [Protocol](PROTOCOL.md)
* [Naming Conventions](NAMING_CONVENTIONS.md)
* [Testing](TESTING.md)
* [Observability](OBSERVABILITY.md)
* [Extended Cloud Event Schema](cloud-event-schema.json)

## Parts of a Polyn Environment

A Polyn environment is made up of **Applications**, **Nodes**, **Components**, and the
**Message Bus**. Following is a description of each.

### Applications

Polyn applications are processes that implement one or more
[Components (or Services)](https://www.reactivemanifesto.org/glossary#Component). Ideally,
components within an application are in some way related to a specific problem domain.

### Nodes

A node is a specific instance of an application. Applications can run multiple nodes, as
necessary.

### Components

A component is a individual event consumer that executes business logic in reaction to the events.
As a rule of thumb, components should be stateless and self-reliant, allowing them to function
in complete isolation from other components.

### Message Bus

Polyn is built from the ground up to utilize the NATS JetStream message bus.

## Environment Architecture

### Subscriptions

Polyn utilizes JetStream to distribute events to subscribing components. JetStream provides
the ability for components to define policies around event delivery. When a Polyn application is
deployed, a script is typically run that sets up the application component's subscriptions, much
like a database migration.

### Event Lifecycle

#### Publishing

When an event is published, the Polyn client validates the event against the pre-defined schema for that event. The schema is a [CloudEvent JSON](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md) compliant [JSON Schema](https://json-schema.org/) published by the Polyn application to an internal repository on startup.

Once validation is complete, the event is published to the message bus.

#### Receiving

When an event is received by an application, and before it is passed to the component, the event is again validated against the schema. Once validated it is passed to the component, where it will then be acknowledged as being received.

Once the event has been processed, it is acknowledged, making way for the next event.

### Schema Repository

When a Polyn application starts, the client publishes all of the event schema for that application to the schema repository. Before doing so, it compares the existing schema in the repository for the events of the same type, against the schema to be published, and will throw an error if the new schema is not backwards compatible with the old. The schema respository should be called `POLYN_SCHEMAS`

### Error Handling

Because components are stateless, Polyn clients adopt a "let it fail" approach to error handling. If an error occurs when processing an event, the clients will simply broadcast a
`polyn.error.v1.<type>` message to the bus, where `type` is the type of failed event.

When validation errors happen, either on the Producer or Consumer side, Polyn implementations will raise an error. The advantage of raising an error when validations fail is that monitoring and alerting tools will have an obvious indication that an event contract has been broken at the exact time that it happens. This makes it much easier to respond to problems quickly and debug them easily. The opposite of this would be to catch the error and allow the Consumer/Producer to decide what to do with it. This gives the Consumer/Producer more of a chance to recover from a broken contract, but increases the likelihood that a failure will happen in a more accidental, unpredictable, harder-to-debug way. If the event isn't adhering to the contract something is probably going to break. We might as well control it the failure so we can address the issue faster.

Polyn applications maintain a schema repository that contains the [CloudEvent JSON]()
compliant [JSON Schema](https://json-schema.org/). The event structure is validated against the schema both before publishing and after receiving events. The schema represents a contract between publisher and subscribers, ensuring events do not mutate in a way that prevents subscribers from consuming the events and accomplishing its task.

Polyn aims to ensure every event is explicit about what format its `data` takes. Even if the schema is just `{ "type" => "null" }` we want the event to be explicit that the event allows for empty data so that Consumers know that an empty paylod is not an accident. Any time an event is published or consumed it MUST be validated against a [JSON Schema](https://json-schema.org/). Both the CloudEvent schema and the `data` schema must be valid.


