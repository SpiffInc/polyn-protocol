# Observability

## Tracing

Polyn clients SHOULD use [OpenTelemetry](https://opentelemetry.io/) to create distributed traces that will connect sent and received events across different services. Clients SHOULD try to follow the [OpenTelemetry Conventions](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/messaging/) for messaging systems as well. The following are additional details about how to enable distributed tracing:

### Publishing

#### Span Name

The span name for a published message should follow the pattern of:

`<event_type> send`

For example:

* `user.created.v1 send`
* `widget.updated.v1 send`

#### Span Kind

The span kind for a published message should be `PRODUCER`

#### Attributes

The span attributes for a published message should contain at least:

```
"messaging.system"                     => "NATS",
"messaging.destination"                => <event_type>,
"messaging.protocol"                   => "Polyn",
"messaging.url"                        => <connected_nats_server_url>,
"messaging.message_id"                 => <event_id>,
"messaging.message_payload_size_bytes" => <byte_size_of_event_data>
```

#### Headers

Eached published message should include a [`traceparent`](https://www.w3.org/TR/trace-context/#traceparent-header) header that can be used by a subscriber to connect the spans together into a single trace. This will likely be accomplishable using an OpenTelemetry library's [TextMapPropagator Module](https://opentelemetry.io/docs/reference/specification/context/api-propagators/#textmap-propagator).

### Subscribing

#### Span Name

The span name for the code that handles a subscribed message should follow the pattern of:

`<event_type> receive`

For example:

* `user.created.v1 receive`
* `widget.updated.v1 receive`

#### Span Kind

The span kind for the code that handles a subscribed message should be `CONSUMER`

#### Attributes

The span attributes for the code that handles a subscribed message should contain at least:

```
"messaging.system"                     => "NATS",
"messaging.destination"                => <event_type>,
"messaging.protocol"                   => "Polyn",
"messaging.url"                        => <connected_nats_server_url>,
"messaging.message_id"                 => <event_id>,
"messaging.message_payload_size_bytes" => <byte_size_of_event_data>
```

#### Span Parent

The span parent for the code that handles a subscribed message should be extracted from the [`traceparent`](https://www.w3.org/TR/trace-context/#traceparent-header) header in the received message. This will likely be accomplishable using an OpenTelemetry library's [TextMapPropagator Module](https://opentelemetry.io/docs/reference/specification/context/api-propagators/#textmap-propagator).

### Batch Processing

#### Span Name

The span name for the code that processes a batch of messages should follow the pattern of:

`<event_type> process`

For example:

* `user.created.v1 process`
* `widget.updated.v1 process`

#### Span Kind

The span kind for code that processes a batch of messages should be `CONSUMER`

#### Links

Each message that is processed should have a "subscription span" that connects the received message to the subscription handler. Additionally, each process message should have a link to the processing span.

For example if you had a batch of 3 messages of the type `user.created.v1` you would do the following:

1. Create a wrapper span of `user.created.v1 process`
2. Extract the 3 messages
3. For each message create a new span called `user.created.v1 receive`
  - Give each `user.created.v1 receive` span a link to the `user.created.v1 process` span
  - Ensure each `user.created.v1 receive` span has a parent_span from the received message's `traceparent` header

```
       SERVICE A
      ┌──────────────────────────┐
      │                          │
      │   user.created.v1 send   │◄─────────────────┐
      │                          │                  │
      └──────────────┬───────────┘                  │
                     │                              │
                     │                              │
                     │                              │
                ┌────▼───┐                          │
                │        │                          │
                │  NATS  │                          │
                │        │                          │
                └────┬───┘                          │
                     │                              │
  SERVICE B          │                              │
┌────────────────────┼──────────────────────────────┼────────────┐
│                    │                              │            │
│    ┌───────────────▼─────────────┐                │            │
│    │                             │                │            │
│    │   user.created.v1 process   │                │            │
│    │                             │                │            │
│    └───────▲─────────────────────┘                │            │
│            │                                      │            │
│            │  ┌──────────────────────────────┐    │parent_span │
│            │  │                              │    │            │
│            ├──┤   user.created.v1 receive    ├────┤            │
│            │  │                              │    │            │
│            │  └──────────────────────────────┘    │            │
│            │                                      │            │
│        link│  ┌──────────────────────────────┐    │            │
│            │  │                              │    │            │
│            ├──┤   user.created.v1 receive    ├────┤            │
│            │  │                              │    │            │
│            │  └──────────────────────────────┘    │            │
│            │                                      │            │
│            │  ┌──────────────────────────────┐    │            │
│            │  │                              │    │            │
│            └──┤   user.created.v1 receive    ├────┘            │
│               │                              │                 │
│               └──────────────────────────────┘                 │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Request-Reply

#### Span Hierarchy

The span hierarchy for the request should be: `<event_type> send` -> `<event_type> receive` -> `(temporary) send` -> `(temporary) receive`

#### Request Publish

The publishing of the requests SHOULD follow the same format as other published messages

#### Request Handler

The subscribing to requests SHOULD follow the same format as other subscriptions.

#### Response Publish

##### Span Name

The span name for the code that publishes a response for a request should be:

`(temporary) send`

This follows the [OpenTelemetry guidance](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/messaging/#span-name) on temporary destinations. When a request is made a temporary "inbox" is created for the `reply_to` header. This "inbox" is the subject that the response publishing code will publish to

##### Span Kind

The span kind for the code that handles a response should be `PRODUCER`

##### Attributes

The span attributes for the code that handles a response message should contain at least:

```
"messaging.system"                     => "NATS",
"messaging.destination"                => "(temporary)",
"messaging.protocol"                   => "Polyn",
"messaging.url"                        => <connected_nats_server_url>,
"messaging.message_id"                 => <event_id>,
"messaging.message_payload_size_bytes" => <byte_size_of_event_data>
```

##### Span Parent

The span parent for publishing a response will be the span that subscribed and received the request

#### Response Handler

##### Span Name

The span name for the code that handles the response of the request should be:

`(temporary) receive`

This follows the [OpenTelemetry guidance](https://opentelemetry.io/docs/reference/specification/trace/semantic_conventions/messaging/#span-name) on temporary destinations. When a request is made a temporary "inbox" is created for the `reply_to` header. This "inbox" is the subject that the response handling code is subscribed to.

##### Span Kind

The span kind for the code that handles a response should be `CONSUMER`

##### Attributes

The span attributes for the code that handles a response message should contain at least:

```
"messaging.system"                     => "NATS",
"messaging.destination"                => "(temporary)",
"messaging.protocol"                   => "Polyn",
"messaging.url"                        => <connected_nats_server_url>,
"messaging.message_id"                 => <event_id>,
"messaging.message_payload_size_bytes" => <byte_size_of_event_data>
```

##### Span Parent

The span parent for the code that handles a response should be extracted from the [`traceparent`](https://www.w3.org/TR/trace-context/#traceparent-header) header in the received message. This will likely be accomplishable using an OpenTelemetry library's [TextMapPropagator Module](https://opentelemetry.io/docs/reference/specification/context/api-propagators/#textmap-propagator). The span hierarchy for the request should be: Request Publish -> Request Handler -> Response Publish -> Response Handler

### Exceptions

If there are any validation errors either when publishing or subscribing, the exception MUST be recorded on the span. Some OpenTelemetry libraries do this automatically, others do not.