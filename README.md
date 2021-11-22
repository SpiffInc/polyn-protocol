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
communicate regardless of the language you use.

## Message Format

```json
{
  "topic": "user.added",
  "eventId": "123e4567-e89b-12d3-a456-426652340000",
  "origin": "some-host/<pid>.frontend",
  "createdAt": "2016-01-01T00:00:00.000Z",
  "client": {
    "type": "ruby",
    "version": "0.1.0",
    "languageVersion": "3.0.2"
  },
  "trace": [
    {
      "eventId": "123e4567-e89b-12d3-a456-426652340000",
      "service": "some-host/pid",
      "createdAt": "2016-01-01T00:00:00.000Z"
    }
  ],
  "payload": {
    "userId": "123e4567-e89b-12d3-a456-426652340000",
    "name": "John Doe",
    "email": "jon@doe.com"
  }
}
```

### Fields
#### topic
The topic of the message. This is the topic or event that actually caused the message 
to be sent.

#### eventId
The unique identifier of the event that caused the message to be sent.

#### origin
The origin of the message. This is the hostname and process id, suffixed with The
name of the service that sent the message.

#### createdAt
The time the event was created.

#### client
The client information of the message originator. 

##### type
The type of client. This is typically the programming language of the client.

##### version
The version of the client.

##### languageVersion
The version of the client's programming language.

#### trace
The trace of the message. This is a list of events that caused the message to be
called in order.

#### payload

The actual payload of the message.

## Client Architecture