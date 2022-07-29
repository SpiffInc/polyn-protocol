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

When an event is published, the Polyn client validates the event against the pre-defined schema
for that event. The schema is a [CloudEvent JSON](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md)
compliant [JSON Schema](https://json-schema.org/) published by the Polyn application to an internal
repository on startup.

Once validation is complete, the event is published to the bus.

#### Receiving

When an event is received by an application, and before it is passed to the component, the event
is again validated against the schema. Once validated it is passed to the component, where it will
then be acknowledged as being received.

Once the event has been processed, it is acknowledged, making way for the next event.

### Schema Repository

<<<<<<< HEAD
When a Polyn application starts, the client publishes all of the event schema for that application
to the schema repository. Before doing so, it compares the existing schema in the repository for
the events of the same type, against the schema to be published, and will throw an error if the
new schema is not backwards compatible with the old.

### Error Handling

This is an object representing the information about the client that published the event as well as
additional Polyn data. A Polyn client SHOULD NOT however fail to consume the event if the `polyndata`
extension is missing.

Because components are stateless, Polyn clients adopt a "let it fail" approach to error handling.
If an error in processing an event occurs, the clients will simply broadcast a
`polyn.error.v1.<type>` message to the bus, where `type` is the type of failed event.

Polyn applications maintain a schema repository that contains the [CloudEvent JSON]()
compliant [JSON Schema](https://json-schema.org/). Event structure is validated against the
schema both before publishing and after receiving events. The schema represents a contract between
publisher and subscribers, ensuring events do not mutate in a way that prevents subscribers from
consuming the events and accomplishing its task.

## Naming and Versioning

### Event `type`

According to the [Version of Cloud Events section](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/primer.md#versioning-of-cloudevents) any backward-_incompatible_ schema changes need to have a completely different name/URI.

> Consumers who have identified a CloudEvent type will generally expect the data within that type to only change in backwardly-compatible ways

> When a CloudEvent's data changes in a backwardly-incompatible way, the value of the type attribute should generally change. The event producer is encouraged to produce both the old event and the new event for some time (potentially forever) in order to avoid disrupting consumers.

Any changes you make to your dataschema that are backwards-_incompatible_ will require you to change the name of the event `type` and produce both events until no Consumers are using the old `type`. For example if the User Producer was producing an event `type` called `app.spiff.user.created.v1` with a dataschema like this:

```json
{
  "type": "object",
  "properties": {
    "first_name": { "type": "string" },
    "last_name": { "type": "string" }
  },
  "required": ["first_name", "last_name"]
}
```

If the User Producer realizes that `first_name` `last_name` doesn't work for all people and updates their schema to be:

```json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" }
  },
  "required": ["name"]
}
```

You would need to update the name of the event `type` from `app.spiff.user.created.v1` to `app.spiff.user.created.v2`.

Examples of backwards-_incompatible_ changes include:

- changing the name of an existing field
- adding a new required field
- changing the type of an existing field
- removing an existing field

Examples of backwards-_compatible_ changes include:

- Adding a new optional field
- Adding metadata such as comments, examples, descriptions

### `dataschema`

The [Cloud Event spec](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/primer.md#versioning-of-cloudevents) recommends that any kind of change to the schema, backwards compatible or incompatible, should get a change in the `dataschema` attribute even if the `type` stays the same. This means that there is a one-to-many relationship between an event `type` and `dataschema`. One `type` of event could have multiple compatible `dataschema`.

If a Producer updates the schema in a backward-_compatible_ way, there will likely be events of the same `type` with different `dataschema` attributes in the event bus. We don't want this kind of change to break a Consumer. A Consumer should be able to receive events with both `dataschema`, validate against them, and not break. The Consumer should be able to add code to handle data in the new schema at its leisure, or not at all if it's not relevant to the Consumer.

### Event and Schema Naming

There is a one-to-many relationship between an event `type` and `dataschema`. One `type` of event could have multiple compatible `dataschema`. A `dataschema` must follow valid [URI syntax](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)

The core part of the event `type` should correspond to the `core` part of the JSON schema name, but the versions may be different. For example event `type` `app.spiff.user.created.v1` could be compatible with `dataschema`

- `app:spiff:user:created:v1:schema:v1`
- `app:spiff:user:created:v1:schema:v2`
- `app:spiff:user:created:v1:schema:v3`

Anytime a backwards-_incompatible_ change is introduced the event `type` should change and the schema name version numbers will start over. For example changing the event `type` to `app.spiff.user.created.v2` would start the `dataschema` over to `app:spiff:user:created:v2:schema:v1`

While the `dataschema` value should be a valid [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier). The file name should look like this: `app.spiff.user.created.v1.schema.v1.json` with the colons replaced by `.` and the file extension `.json` added to the end

## Validation

Every event will require the `dataschema` property be populated. With a valid [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier) that points to a JSON file that follows the [JSON Schema](https://json-schema.org/) specification. This differs from the [CloudEvents](https://github.com/cloudevents) which says `dataschema` can be optional. Polyn aims to ensure every event is explicit about what format its data takes. Even if the schema is just `{ "type" => "null" }` we want the event to be explicit that it can have no data so that Consumers know that an empty paylod is not an accident. Even events that are not being serialized as JSON (e.g. Avro, Kafka, etc) should use the [JSON Schema](https://json-schema.org/) to validate the format of their `data` property.

### Errors

When validation errors happen, either on the Producer or Consumer side, Polyn implementations will raise an error. The advantage of raising an error when validations fail is that monitoring and alerting tools will have an obvious indication that an event contract has been broken at the exact time that it happens. This makes it much easier to respond to problems quickly and debug them easily. The opposite of this would be to catch the error and allow the Consumer/Producer to decide what to do with it. This gives the Consumer/Producer more of a chance to recover from a broken contract, but increases the likelihood that a failure will happen in a more accidental, unpredictable, harder-to-debug way. If the event isn't adhering to the contract something is probably going to break so we might as well control it so we can address the issue faster.
