# Polyn Naming Conventions

## NATS Consumer Name

### Format
`<component_name>_<event_name>_<version>`

### Examples

* `notifications_user_created_v1`
* `orders_backend_user_created_v1`

### Explanation

A NATS Consumer puts a cursor on messages in a stream. Each time a message is delivered the cursor moves. Each Component/Service in the system will need to create its own Consumer for any events it cares about. We want to include the Component that is using the Consumer in the name so it's clear who owns it.

## Cloud Event Type

### Format
`<reverse_domain_name>.<event_name>.<version>`

### Examples
`app.spiff.user.created.v1`

### References
https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md#type

### Explanation

The Cloud Event spec indicates that the event `type` should include the reverse domain name of the organization at the beginning. Most application code should not need to worry about this as the stream subjects and JSON schema files will only have the `<event_name>.<version>` in them.

The `event_name` portion SHOULD be written in past tense ("account registered", "funds transferred", "fraudulent activity detected" etc.)

### Versioning

According to the [Version of Cloud Events section](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/primer.md#versioning-of-cloudevents) any backward-_incompatible_ schema changes need to have a completely different name/URI.

> Consumers who have identified a CloudEvent type will generally expect the data within that type to only change in backwardly-compatible ways

> When a CloudEvent's data changes in a backwardly-incompatible way, the value of the type attribute should generally change. The event producer is encouraged to produce both the old event and the new event for some time (potentially forever) in order to avoid disrupting consumers.

Any changes you make to your dataschema that are backwards-_incompatible_ will require you to change the name of the event `type` and produce both events until no Consumers are using the old `type`. For example if the User Producer was producing an event `type` called `user.created.v1` with a dataschema like this:

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

You would need to update the name of the event `type` from `user.created.v1` to `user.created.v2`.

Examples of backwards-_incompatible_ changes include:

- changing the name of an existing field
- adding a new required field
- changing the type of an existing field
- removing an existing field

Examples of backwards-_compatible_ changes include:

- Adding a new optional field
- Adding metadata such as comments, examples, descriptions

## Schema `$id`

### Format
`<reverse_domain_name>:<event_name>:<version>`

### Examples
`app:spiff:user:created:v1`

### References
https://json-schema.org/understanding-json-schema/structuring.html#id

### Explanation
We want the `$id` of the schema to be the same as the event `type`, but in URI format. These are only intended to be used internally and so they don't need to have a web URL.

## Cloud Event Source

### Format
`<reverse_domain_name>:<component_name>`

### Examples

* `app:spiff:notifications`
* `app:spiff:orders`
* `app:spiff:payments`

### References
https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md#source-1

### Explanation

The `source` of any event MUST be a URI and MUST be unique when combined with the event id. We want to include the Component/Service name that's producing the event so if there are problems its easier to track down where the event originated.

## Stream Name

### Format

`<STREAM_NAME>`

### Examples

* `ORDERS`
* `ORDERS_NEW`
* `WIDGETS`
* `USERS`

### References

https://docs.nats.io/nats-concepts/jetstream/streams

### Explanation

The stream name SHOULD be in all caps with separating underscores. This is the way most of the stream names are shown in the NATS docs so we may as well be consistent with that.

## Stream Subjects

### Format
`<event_name>.<version>`

### Examples
`user.created.v1`

### Explanation
We want there to be a one-to-one relationship between subject and event so it's simple and clear where the events will exist


