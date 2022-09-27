# Testing

Client libraries SHOULD ensure that application tests that involve Polyn can be reliable and concurrent. The inclusion of Polyn in a library should not contribute to flaky tests or slow test suites.

## Mock NATS

NATS is a messaging system that is not owned by any one service. As such it is an [unmanaged dependency](https://vkhorikov.medium.com/dont-mock-your-database-it-s-an-implementation-detail-8f1b527c78be) and should be mocked. Polyn clients SHOULD provide a path for an application's test suite to use a mocked version of NATS that the Polyn client provides.

### What to Mock

The Polyn client should mock NATS in such a way that any messages published during a test are only available to that test. Subscribers in concurrently running tests should not be impacted by each others' messages.

### What not to Mock

Schema, Stream, and Consumer lookups should not be mocked. This will ensure the application is using schemas, streams, and consumers that actually exist and have the expected properties. As such a test suite should have a NATS server running that has been configured similar to production using Polyn-cli (this might be through a Docker container).
