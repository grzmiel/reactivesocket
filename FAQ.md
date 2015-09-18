### FAQ

#### Why a protocol? What makes creating a new protocol (a quite low level task that most devs don't really do on a daily basis) the right thing for improving large scale distributed systems?



#### Why not HTTP/2?

HTTP/2 is MUCH better for browsers and request/response document transfer, but unfortunately does not expose interaction models beyond request/response, or support application level flow control. 

Here are some quotes from the HTTP/2 spec and FAQ that are useful to provide context on what HTTP/2 was targeting:

> “HTTP's existing semantics remain unchanged.”

> “… from the application perspective, the features of the protocol are largely unchanged …”

> "This effort was chartered to work on a revision of the wire protocol – i.e., how HTTP headers, methods, etc. are put “onto the wire”, not change HTTP’s semantics."

Additionally, "push promises" are focused on filling browser caches for standard web browsing behavior:

> “Pushed responses are always associated with an explicit request from the client.”
This means we still need SSE or WebSockets (and SSE is a text protocol so requires Base64 encoding to UTF-8) for push.

HTTP/2 was meant as a better HTTP/1.1, primarily for document retrieval in browsers for websites. We can do better than HTTP/2 for applications.

More on the motivations behind Reactive Socket can be found inhttps://github.com/ReactiveSocket/reactivesocket/blob/master/Motivations.md.

#### Why "Reactive Streams" `request(n)` Flow Control?

Without application feedback in terms of work units done (not bytes), you get into a couple problems:

Data is buffered by TCP on the sender and receiver side which means that understanding what is done on the subscriber is not possible.

A sender who needs to send a large work unit (larger than the buffering on the TCP sender or receiver sides) is kinda stuck to a bad behaving scenario where the TCP connection will cycle between full and empty and under utilize the buffering drastically (as well as the throughput)

TCP isn't the only transport that would make sense to use.

TCP handles a single sender/receiver pair and reactive streams allows for multiple senders and/or multiple receivers (somewhat), and

(most importantly) decoupling of data reception at the transport layer from application consumption control. I.e. an application may want to artificially slow down or limit processing separately from pulling off the data from the transport.

It all comes down to what TCP is designed to do (not overrun the receiver OS buffer space or network queues) and what reactive-streams flow control is designed to do (allow for push/pull application work unit semantics, additional dissemination models, and application control of when it is ready for more or not). This clear separation of concerns is necessary for any real system to operate efficiently.

This illustrates why ever single solution that doesn't have built in flow control at the application level (pretty much every solution mentioned aside from MQTT, AMQP, & STOMP) is not well suited for usage.




#### What about Session Continuation across connections?


#### Connection Setup Requirement

This is effectively the same as the HTTP/2 requirement to exchange SETTINGS frames: https://http2.github.io/http2-spec/#ConnectionHeader and https://http2.github.io/http2-spec/#discover-http

HTTP/2 and Reactive Socket both require a stateful connection with an initial exchange. 

#### Transport Layer

HTTP/2 requires TCP: https://http2.github.io/http2-spec/#starting

Reactive Socket requires TCP, WebSockets or Aeron: https://github.com/ReactiveSocket/reactivesocket/blob/master/Protocol.md#terminology

We have no intention of this running over HTTP/1.1. We also do not intend on running over HTTP/2, though that could be explored and conceptually is possible (with the use of SSE).

#### Proxying

Proxies that behave correctly for HTTP/2 will behave correctly for Reactive Socket.

#### Frame Length

On TCP, it will be included. On Aeron or WebSockets it is not needed. 

If there is some reason to include it in Aeron or WebSockets we are fine with changing. 

#### State Spanning Connections

We determine this to be an unnecessary optimization at this protocol layer since the application has to be involved to make it work. Applications maintain state between connections. It is also very complex to implement for negligible gain. Here are many examples of distributed systems failing at these types of problems: https://aphyr.com/tags/jepsen

#### Future Proofing

There is no way to fully future proof something, but we have made the attempt to future proof through the following ways:

- Frame type has a reserved value for extension
- Error code has a reserved value for extension
- Setup has a version field
- All fields have been sized according to given requirements as known currently (such as streamId supporting 4b requests)
- There is plenty of space for additional flags
- Separation of data and metadata
- Use of MimeType in Setup to eliminate coupling with encoding

Additionally, we have stuck within connection-oriented semantics of HTTP/2 and TCP so that connection behavior is not abnormal or special. 

Beyond those factors, TCP has existed since 1977. We do not expect it to be eliminated in the near future. Quic looks to be a legit alternative to TCP in the coming years. Since HTTP/2 is already working over Quic, we see no reason why Reactive Socket will not also work over Quic. 

#### Prioritization, QoS, OOB





#### Why is cancellation required?

####  What are example use cases of cancellation?

#### What are example use cases of topic subscription (and push notification)?

#### What are example use cases of fire-and-forget versus request-response?

#### What are example use cases of request-stream?

#### Why Binary?

https://http2.github.io/faq/#why-is-http2-binary

#### Doesn't binary encoding make debugging harder?

#### What tooling exists for debugging the protocol?

#### Why are these different flow control approaches needed beyond what the transport layer offers?

#### What are example use cases where ReactiveSocket flow control helps?

#### How does ReactiveSocket flow control behave?

#### How does ReactiveSocket benefit a client-side load balancer in a data center?

#### Why is multiplexing more efficient?

https://http2.github.io/faq/#why-is-http2-multiplexed
https://http2.github.io/faq/#why-just-one-tcp-connection

#### Is multiplexing equivalent to pipelining?

#### What is difference between flow control and the reactive-stream backpressure protocol?

#### Why is the "TLS False start" strategy useful for establishing a connection?

#### What are example use cases for payload data on the Setup frame?


#### Why those 5 interaction models?

