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
  "dataschema": "app.spiff.polyn.service.started.v1.schema.v1.json",
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
