# Polyn Protocol (v 0.1.0)

This documentation describes how Polyn compliant clients should be implemented.

## Conformance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Definitions

- **application** - a single process that implements one or more services.
- **event** - any message being published to the transporter by the clients.
- **message bus** - the NATS Jetstream message bus
- **services** - an event consumer that implements one or more event endpoints
- **subscriptions** - any endpoint within a service that implements specific business logic to be
  triggered as the result of consuming a subscribed event.
- **topic** - a reverse domain name representing a unique event.
- **uuid** - a [UUID v4](https://datatracker.ietf.org/doc/html/rfc4122) compliant UUID.

## Message Format

A Polyn client MUST publish events that comply with the
[CloudEvents JSON](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md)
specification. In addition a client MUST add a`polyntrace` and `polynclient` extension as described
[here](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md#2-attributes).

While CloudEvents supports multiple formats, for simplicities sake, Polyn clients only support the
JSON version of the spec.

### Example

```json
{
  "specversion" : "1.0.1",
  "type" : "<topic>",
  "source" : "location or name of service",
  "id" :  "<uuid>",
  "time" : "2018-04-05T17:31:00Z",
  "polyntrace": [
    {
      "type": "<topic>",
      "time": "2018-04-05T17:31:00Z",
      "id": "<uuid>"
    }
  ],
  "polynclient": {
    "lang": "ruby",
    "langversion": "3.2.1",
    "version": "0.1.0"
  },
  "datacontenttype" : "application/json",
  "dataschema": "app:spiff:polyn:service:started:v1:schema:v1",
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
    "type": "<topic>",

    "time": "2018-04-05T14:31:00Z",
    "id": "<uuid>"
  },
  {
    "type": "<topic>",
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
  context.publish("<topic>", result_of_work)
end
```

This would add whatever event that caused `receive_some_event` to be fired to the trace when
`context.publish` is called.

#### `polynclient`

This is an object representing the information about the client that published the event. A Polyn
client SHOULD NOT however fail to consume the event if the `polynclient` extension is missing.

##### Example

```json
{
  "lang": "ruby",
  "langversion": "3.2.1",
  "version": "0.1.0"
}
```
