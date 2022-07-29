# Polyn Protocol (v 0.1.0)

This documentation describes how Polyn compliant clients should be implemented.

## Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Definitions

- **application** - a single process that implements one or more components.
- **event** - any message being published to the transporter by the clients.
- **message bus** - the NATS JetStream message bus
- **components** - an event consumer that implements one or more event endpoints
- **subscriptions** - any endpoint within a service that implements specific business logic to be triggered as the result of consuming a subscribed event.
- **type** - a reverse domain name representing a unique event.
- **uuid** - a [UUID v4](https://datatracker.ietf.org/doc/html/rfc4122) compliant UUID.
- **Schema Repository** - a NATS KeyValue backed store of all events published by registered Polyn
  components.

## Message Format

A Polyn client MUST publish events that comply with the
[CloudEvents JSON](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md)
specification. In addition a client MUST add a `polyntrace` and `polyndata` extension as described
[here](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md#2-attributes).

While CloudEvents supports multiple formats, for simplicities sake, Polyn clients only support the JSON version of the spec.

### Example

```json
{
  "specversion" : "1.0.1",
  "type" : "<type>",
  "source" : "location or name of service",
  "id" :  "<uuid>",
  "time" : "2018-04-05T17:31:00Z",
  "polyntrace": [
    {
      "type": "<type>",
      "time": "2018-04-05T17:31:00Z",
      "id": "<uuid>"
    }
  ],
  "polyndata": {
    "clientlang": "ruby",
    "clientlangversion": "3.2.1",
    "clientversion": "0.1.0"
  },
  "datacontenttype" : "application/json",
  "data" : {}
  }
}
```

### CloudEvents Extensions

Polyn Clients MUST implement the following extensions.

#### `polyntrace`

This is an array of trace objects that identify the chain of events that lead the current event to
being published. A Polyn client SHOULD NOT fail to consume an event however if `polyntrace` is
missing from the published event.

##### Example

```json
[
  {
    "type": "<type>",
    "time": "2018-04-05T14:31:00Z",
    "id": "<uuid>"
  },
  {
    "type": "<type>",
    "time": "2018-04-05T17:31:00Z",
    "id": "<uuid>"
  }
]
```

A Polyn client MUST provide a way to publish an event as the result of a prior event, adding the
trace of the predecessor to the `polyntrace` array.

##### Example

```ruby
def receive_some_event(context)
  # ... do work
  context.publish("<type>", result_of_work)
end
```

This would add whatever event that caused `receive_some_event` to be fired to the trace when
`context.publish` is called.

#### `polyndata`

This is an object representing the information about the client that published the event as well as additional metadata. A Polyn client SHOULD NOT however fail to consume the event if the `polyndata` extension is missing.

##### Example

```json
{
  "clientlang": "ruby",
  "clientlangversion": "3.2.1",
  "clientversion": "0.1.0"
}
```

### Additional Field Requirements

#### `datacontenttype`

A Polyn client SHOULD add the `datacontenttype` field as defined [here](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md#datacontenttype)
before publishing events that correctly match the content type of the `data` field. It MUST at a minimum be able to serialize and deserialize the `application/json` content type.

If the `datacontenttype` is not present in the event, a Polyn client MUST assume that the content type is `application/json`. If the data cannot be deserialized, the client MUST broadcast an [appropriate error]().

#### `data`

A Polyn client MUST add its event data to the `data` attribute of the CloudEvent. The data format must match the `datacontentype` field.

## Message Validation

### JSON Schema

A Polyn client MUST support loading a [JSON Schema](https://json-schema.org/) document for each event. It MUST wrap the event schema into a valid CloudEvent JSON Schema, and publish that schema to the Schema Repository. The Schema Repository should be a JetStream KeyValue Bucket called `POLYN_SCHEMAS`. Each event's JSON Schema should be a CloudEvent JSON Schema document with the `data` section replaced with the schema specific to the event.

#### Schema Backwards Compatibility

A Polyn client SHOULD check the Schema Repository for event schema of the same name before
publishing its events to the repository. It SHOULD validate that the event schema to be published
are backwards compatible with the schema

Changes considered to break backwards compatibility are considered to be:

- removing or renaming fields, including deep changes within objects
- changing field data types

Adding fields SHOULD be considered to be a backwards compatible change.

### Validation on Publish

A valid Polyn client SHOULD validate events against their respective JSON schema before publishing
the event to the bus.

### Validation on Receive

A valid Polyn client SHOULD validate events received against their respective JSON schema before
processing said events.

## NATS JetStream

### Publishing

A Polyn client MUST publish full CloudEevent messages utilizing [NATS Jetstream](https://docs.nats.io/nats-concepts/jetstream).
The subject should the type of the event. For example if the event type were
`app.widgets.created.v1` the client MUST publish the event to the `app.widgets.created.v1` subject.

### Subscribing

A Polyn client MUST subscribe its components to a [NATS JetStream consumer]()
whose name is the reverse domain name of the application, suffixed with the component name and
event type delimited by underscores (`_`).

For example, if the application reverse domain were `app.widgets`, and the component consuming events
were the `new_widget_notifier` component, and that component was subscribing to `app.widgets.created.v1`,
the `app_widgets_new_widget_notifier_app_widgets_created_v1`.

A Polyn client MUST NOT attempt to set up its own consumers. This is handled by the [Polybn CLI]()
within the [events repository]()
