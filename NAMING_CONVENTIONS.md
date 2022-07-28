# Polyn Naming Conventions

## NATS Consumer Name

### Format
`<reverse_domain_name>_<component_name>_<event_name>_<version>`

### Examples
`app_spiff_notifications_user_created_v1`

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

The Cloud Event spec indicates that the event `type` should include the reverse domain name of the organization at the beginning. Most application code should not need to worry about this as the stream subjects and JSON schema files will only have the `event_name` in them.

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

`app:spiff:notifications`
`app:spiff:orders`
`app:spiff:payments`

### References
https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/spec.md#source-1

### Explanation

The `source` of any event MUST be a URI and MUST be unique when combined with the event id. We want to include the Component/Service name that's producing the event so if there are problems its easier to track down where the event originated.

## Stream Subjects

### Format
`<event_name>.<version>`

### Examples
`user.created.v1`

### Explanation
We want there to be a one-to-one relationship between subject and event so it's simple and clear where the events will exist


