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

## Message Format

Polyn publishes events using the [CloudEvents](https://github.com/cloudevents) spec with
a few small extensions.

```json
{
  "specversion" : "1.0.1",
  "type" : "com.some.event",
  "source" : "location or name of service",
  "id" :  "a-unique-id"
  "time" : "2018-04-05T17:31:00Z",
  "polyntrace": [
    {
      "type": "com.some.other.event",
      "time": "2018-04-05T17:31:00Z",
      "id": "a-unique-id"
    }
  ],
  "polynclient": {
    "lang": "ruby",
    "langversion": "3.2.1"
    "version": "0.1.0"
  },
  "datacontenttype" : "application/json",
  "dataschema": "app:spiff:polyn:service:started:v1:schema:v1",
  "data" : {}
  }
}
```

### Extensions

- **polyntrace** - this is a trace of the chain of events that lead to this
  event being published. This helps trace long event chains back to their
  origin
- **polynclient** - this contains details of the client that published this
  event.

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
    "name": { "type": "string" },
  },
  "required": ["name"]
}
```

You would need to update the name of the event `type` from `app.spiff.user.created.v1` to `app.spiff.user.created.v2`.

Examples of backwards-_incompatible_ changes include:
* changing the name of an existing field
* adding a new required field
* changing the type of an existing field
* removing an existing field

Examples of backwards-_compatible_ changes include:
* Adding a new optional field
* Adding metadata such as comments, examples, descriptions

### `dataschema`
The [Cloud Event spec](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/primer.md#versioning-of-cloudevents) recommends that any kind of change to the schema, backwards compatible or incompatible, should get a change in the `dataschema` attribute even if the `type` stays the same. This means that there is a one-to-many relationship between an event `type` and `dataschema`. One `type` of event could have multiple compatible `dataschema`.

If a Producer updates the schema in a backward-_compatible_ way, there will likely be events of the same `type` with different `dataschema` attributes in the event bus. We don't want this kind of change to break a Consumer. A Consumer should be able to receive events with both `dataschema`, validate against them, and not break. The Consumer should be able to add code to handle data in the new schema at its leisure, or not at all if it's not relevant to the Consumer.

### Event and Schema Naming
There is a one-to-many relationship between an event `type` and `dataschema`. One `type` of event could have multiple compatible `dataschema`. A `dataschema` must follow valid [URI syntax](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)

The core part of the event `type` should correspond to the `core` part of the JSON schema name, but the versions may be different. For example event `type` `app.spiff.user.created.v1` could be compatible with `dataschema`
* `app:spiff:user:created:v1:schema:v1`
* `app:spiff:user:created:v1:schema:v2`
* `app:spiff:user:created:v1:schema:v3`

Anytime a backwards-_incompatible_ change is introduced the event `type` should change and the schema name version numbers will start over. For example changing the event `type`  to `app.spiff.user.created.v2` would start the `dataschema` over to `app:spiff:user:created:v2:schema:v1`
