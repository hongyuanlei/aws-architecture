## Streams vs Queues

Unlike message queues and buses, messages don't move from the streams they're written to. They can be read by any number of services that they were not originally sent to so that those services can catch up on what has already happened, and either choose to process those messages or ignore them.

Once messages are written to a stream, they stay there. Conversely, with message queues or message buses, a message is removed from a queue and then passed to a service's consumer.

Under certain conditions, this removal of messages from queues to send them to service endpoints results a message being lost before it is processed. More commonly, the loss of the acknowledgment that the message was indeed processed leads to the re-processing of the message, which is why idempotent logic is a necessity no matter the "guarantees" offered by the service bus software. These conditions eventually lead to ever more elaborate message bus features that complicate their use and operation.

No matter how much of the complexity of message buses is hidden by ever more elaborate freatures, they cannot be made to guarantee "only-once" message delivery, and they cannot be made to be as simple as message streams.

Stream eliminate the inherent complexity of message buses. Messages don't move from the streams services. Rather, the service maintains a pointer or cursor to the message in the stream that it is processing. The cursor moves across the messages in the stream, advancing each time a message is processed. Once a message is processed, the service's cursor moves to the next message. But the messages are not remove from a stream in order to process them.

The most important difference between traditional Service-Oriented Architecture(SOA) and Microservices Architecture is the replacement of service buses with the simplicity of streams.

SOA services are described as Smart Pipes, Dumb Endpoints. In the age of Microservice Architecture, the axiom is reversed: Dumb Pipes, Smart Endpoints.

In the age of Microservices, the pipe is dumb and the endpoint is smart. Meaning, the service consumer, ranther than the bus, keeps track of which message to read next. Because of this, the risk of losing messages is eliminated, and the application code does not have to account for the potential loss of transactions. And neither does the team writing the application code.