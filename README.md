# itmp

Internet of Things Messaging Protocol (ITMP)

ITMP is a routable protocol that provides two messaging patterns: Publish & Subscribe and Remote Procedure Calls, and the Service Discovery ability. It is intended to connect application components in distributed applications. ITMP uses WebSocket as its default transport, but can be transmitted via any other protocol that allows for bi-directional, and message-oriented communications.

## 1. Introduction

### 1.1. Background

The Internet of Things Messaging Protocol (ITMP) is intended to provide application developers with the semantics they need to handle messaging between components in distributed applications.
ITMP is a route-able protocol, with all components can connect to a ITMP Router, where the ITMP Router performs message routing between the components.
ITMP provides two messaging patterns: Publish & Subscribe and Remote Procedure Calls.
Publish & Subscribe (PubSub) is an established messaging pattern where a component, the _Subscriber_, informs the router that it wants to receive information on a topic (i.e., it subscribes to a topic).
Another component, a _Publisher_, can then publish messages or events to this topic, and the router distributes those events to all Subscribers.
Remote Procedure Calls (RPCs) rely on the same sort of decoupling that is used by the Publish & Subscribe pattern. A component, the _Callee_, announces to the router that it provides a certain procedure, identified by a procedure name. Other components, _Callers_, can then call the procedure, with the router invoking the procedure on the Callee, receiving the procedure's result, and then forwarding this result back to the Caller. Routed RPCs differ from traditional client-server RPCs in that the router serves as an intermediary between the Caller and the Callee.
The decoupling in routed RPCs arises from the fact that the Caller is no longer required to have knowledge of the Callee; it merely needs to know the identifier of the procedure it wants to call. There is
also no longer a need for a direct connection between the caller and the callee, since all traffic is routed. This enables the calling of procedures in components which are not reachable externally (e.g. on a NATted connection) but which can establish an outgoing connection to the ITMP router.
Combining these two patterns into a single protocol allows it to be used for the entire messaging requirements of an application, thus reducing technology stack complexity, as well as networking overheads. Adding _discovery_ pattern allows both components know facilities each other.

### 1.2. Protocol Overview

The PubSub messaging pattern defines three roles: _Subscribers_ and _Publishers_, which communicate via a _Broker_.
The RPC messaging pattern also defines three roles: _Callers_ and _Callees_, which communicate via a _Dealer_.
ITMP Connections are established by _Clients_ to a _Router_. Connections can use any transport that is message-based and bi-directional, with WebSocket as the default transport.
A Router is a component which implements one or both of the Broker and Dealer roles. A Client is a component which implements any or all of the Subscriber, Publisher, Caller, or Callee roles.
ITMP _Connections_ are established by Clients to a Router. Connections can use any transport which is message-oriented, ordered, reliable and bi-directional, with WebSocket as the default transport.
ITMP _Sessions_ are established over a ITMP Connection. A ITMP Session is joined to a _Realm_ on a Router. Routing occurs only between ITMP Sessions that have joined the same Realm.

#### 1.3.2. Application Code

ITMP is designed for application code to run within Clients, i.e. _Peers_ having the roles Callee, Caller, Publisher, and Subscriber.
Routers, i.e. Peers of the roles Brokers are responsible for *generic call and event routing*. This allows the transparent exchange of Broker implementations without affecting the application and to distribute and deploy application components flexibly.

#### 1.3.3. Language Agnostic

ITMP is language agnostic, i.e. can be implemented in any programming language. At the level of arguments that may be part of a ITMP message, ITMP takes a 'superset of all' approach. ITMP implementations may support features of the implementing language for use in arguments.

#### 1.3.4. Router Implementation Specifics

This specification only deals with the protocol level. Specific ITMP Broker and Dealer implementations may differ in aspects such as support for:
authentication and authorization schemes,
message persistence, and,
management and monitoring.
The definition and documentation of such Router features is outside the scope of this document.

### 1.4. Relationship to WebSocket

The WebSocket protocol brings bi-directional real-time connections to the browser. It defines an API at the message level, requiring users who want to use WebSocket connections in their applications to define their own semantics on top of it.
ITMP uses WebSocket as its default transport binding, and in the future is a registered WebSocket subprotocol.

## 2. Conformance Requirements

All diagrams, examples, and notes in this specification are non-normative, as are all sections explicitly marked non-normative. Everything else in this specification is normative.
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 [RFC2119].
Requirements phrased in the imperative as part of algorithms (such as "strip any leading space characters" or "return false and abort these steps") are to be interpreted with the meaning of the key word ("MUST", "SHOULD", "MAY", etc.) used in introducing the algorithm.

Conformance requirements phrased as algorithms or specific steps MAY be implemented in any manner, so long as the end result is equivalent.

### 2.1. Terminology and Other Conventions

Key terms such as named algorithms or definitions are indicated like _this_ when they first occur, and are capitalized throughout the text.

## 3. Realms, Sessions and Transports

A Realm is a ITMP routing and administrative domain, optionally protected by authentication and authorization. ITMP messages are only routed within a Realm. Realm with empty name is default.
A Session is a transient conversation between two Peers attached to a Realm and running over a Transport.
A Transport connects two ITMP Peers and provides a channel over which ITMP messages for a ITMP Session can flow in both directions.
ITMP can run over any Transport which is message-based, bidirectional, reliable and ordered.
The default transport for ITMP is WebSocket [RFC6455], where ITMP is an officially registered [1] subprotocol.

## 4. Peers and Roles

A ITMP Session connects two Peers. Each ITMP Peer MUST implement one role, and MAY implement more roles.
A Peer MAY implement any combination of the Roles:

* Callee
* Caller
* Publisher
* Subscriber

Some time you need to combine several Roles - you can use

* Broker = [ Callee +  Caller + Publisher + Subscriber ]
* Node = [Calle + Publisher]

This document describes ITMP as in client-to-client communication, client-to-router and router-to-router communication is supported by ITMP by same way.

### 4.1. Symmetric Messaging

It is important to note that though the establishment of a Transport might have an inherent asymmetry (like a TCP client establishing a WebSocket connection to a server), and Clients establish ITMP sessions by attaching to Realms on Routers, ITMP itself is designed to be fully symmetric for application components.
After the transport and a session have been established, any application component may act as Caller, Callee, Publisher and Subscriber at the same time. And Routers provide the fabric on top of which ITMP runs a symmetric application messaging service.

### 4.2. Remote Procedure Call Roles

The Remote Procedure Call messaging pattern involves peers of two different roles:

* Callee
* Caller

A Caller issues calls to remote procedures by providing the procedure URI and any arguments for the call. The Callee will execute the procedure using the supplied arguments to the call and return the result of the call to the Caller.
The Caller and Callee will usually run application code, while the Broker works as a generic router for remote procedure calls decoupling Callers and Callees.

### 4.3. Publish & Subscribe Roles

The Publish & Subscribe messaging pattern involves peers of two different roles:

* Subscriber
* Publisher

A Publishers publishes events to topics by providing the topic URI and any payload for the event. Subscribers of the topic will receive the event together with the event payload. Subscribers subscribe to topics they are interested.

Broker act both as Publisher for Subscribers and as Subscriber for Publishers at the same time. Brokers route events incoming from Publishers to Subscribers that are subscribed to respective topics.

The Publisher and Subscriber will usually run application code, while the Broker works as a generic router for events decoupling Publishers from Subscribers.

### 4.4. Peers with multiple Roles

Note that Peers might implement more than one role: e.g. a Peer might act as Caller, Publisher and Subscriber at the same time. Another Peer might act also as Broker.

## 5. Building Blocks

ITMP is defined with respect to the following building blocks

1. Identifiers
2. Serializations
3. Transports

For each building block, ITMP only assumes a defined set of requirements, which allows to run ITMP variants with different concrete bindings.

### 5.1. Identifiers

#### 5.1.1. URIs

ITMP needs to identify the following persistent resources:

1. Topics
2. Procedures
3. Interfaces

These are identified in ITMP using Uniform Resource Identifiers (URIs) [RFC3986] that MUST be Unicode strings.
When using JSON as ITMP serialization format, URIs (as other strings) are transmitted in UTF-8 [RFC3629] encoding.
_Examples_
"com.myapp.mytopic1"
"com.myapp.myprocedure1"
The URIs are understood to form a single, global, hierarchical namespace for ITMP.
The namespace is unified for topics and procedures - these different resource types do NOT have separate namespaces.
To avoid resource naming conflicts, the package naming convention from Java is used, where URIs SHOULD begin with (reversed) domain names owned by the organization defining the URI.

##### 5.1.1.1. Relaxed/Loose URIs

URI components (the parts between two "."s, the head part up to the first ".", the tail part after the last ".") MUST NOT contain a ".", "#" or whitespace characters and MUST NOT be empty (zero-length strings), except for whole URI that can be an empty string.
The restriction not to allow "." in component strings is due to the fact that "." is used to separate components, and ITMP associates semantics with resource hierarchies, such as in pattern-based subscriptions. The restriction not to allow empty (zero-length) strings as components is due to the fact that this may be used to denote wildcard components with pattern-based subscriptions and registrations in the Advanced Profile. The character "#" is not allowed since this is reserved for internal use by Brokers.
As an example, the following regular expression could be used in Python to check URIs according to the above rules:

```regexp
loose URI check disallowing empty URI components
pattern = re.compile(r"^([^\s\.#]+\.)*([^\s\.#]+)$")
```

When empty URI components are allowed (which is the case for specific messages that are part of the Advanced Profile), this following regular expression can be used (shown used in Python):

```regexp
loose URI check allowing empty URI components
pattern = re.compile(r"^(([^\s\.#]+\.)|\.)*([^\s\.#]+)?$")
```

##### 5.1.1.2. Strict URIs

While the above rules MUST be followed, following a stricter URI rule is recommended: URI components SHOULD only contain lower-case letters, digits and "_".
As an example, the following regular expression could be used in Python to check URIs according to the above rules:

```regexp
strict URI check disallowing empty URI components
pattern = re.compile(r"^([0-9a-z\_]+\.)*([0-9a-z_]+)$")
```

When empty URI components are allowed (which is the case for specific messages), the following regular expression can be used (shown in Python):

```regexp
strict URI check allowing empty URI components
pattern = re.compile(r"^(([0-9a-z_]+\.)|\.)*([0-9a-z_]+)?$")
```

Following the suggested regular expression will make URI components valid identifiers in most languages (modulo URIs starting with a digit and language keywords) and the use of lower-case only will make those identifiers unique in languages that have case-insensitive identifiers. Following this suggestion can allow implementations to map topics, procedures and errors to the language environment in a completely transparent way.

#### 5.1.2. IDs

ITMP needs to identify the following ephemeral entities each in the scope noted:

1. Sessions (_global scope_)
2. Publications (_global scope_)
3. Subscriptions (_router scope_)
4. Requests (_session scope_)

These are identified in ITMP using IDs that are integers between (inclusive) *0* and *2^53* (9007199254740992):
IDs in the _global scope_ MUST be drawn _randomly_ from a _uniform distribution_ over the complete range [0, 2^53]
IDs in the _router scope_ can be chosen freely by the specific router implementation
IDs in the _session scope_ SHOULD be incremented by 1 beginning with 1 (for each direction - _Client-to-Router_ and _Router-to-Client_)
The reason to choose the specific upper bound is that 2^53 is the largest integer such that this integer and _all_ (positive) smaller integers can be represented exactly in IEEE-754 doubles. Some languages (e.g. JavaScript) use doubles as their sole number type. Most languages do have signed and unsigned 64-bit integer types that both can hold any value from the specified range.
The following is a complete list of usage of IDs in the three categories for all ITMP messages. For a full definition of these see Section 6.

##### 5.1.2.1. Global Scope IDs

* "CONNECTED.Session"
* "PUBLISHED.Publication"

##### 5.1.2.2. Router Scope IDs

* "SUBSCRIBED.Subscription"

##### 5.1.2.3. Session Scope IDs

* "PUBLISH.Request"
* "SUBSCRIBE.Request"
* "UNSUBSCRIBE.Request"
* "CALL.Request"

### 5.2. Serializations

ITMP is a message-based protocol that requires serialization of messages to octet sequences to be sent out on the wire.
A message serialization format is assumed that (at least) provides the following types:

* "integer"
* "string" (UTF-8 encoded Unicode)
* "bool"
* "list"
* "dict" (with string keys)

ITMP _itself_ only uses the above types, e.g. it does not use the JSON data types "number" (non-integer) and "null". The _application payloads_ transmitted by ITMP (e.g. in call arguments or event payloads) may use other types a concrete serialization format supports.
There is no required serialization or set of serializations for ITMP implementations (but each implementation MUST, of course, implement at least one serialization format). Routers SHOULD implement more than one serialization format, enabling components using different kinds of serializations to connect to each other.
ITMP defines two bindings for message serialization:

1. JSON
2. CBOR

Other bindings for serialization may be defined in future ITMP versions.

#### 5.2.1. JSON

With JSON serialization, each ITMP message is serialized according to the JSON specification as described in RFC4627.
Further, binary data follows a convention for conversion to JSON strings. For details see the Appendix.

#### 5.2.2. CBOR

With CBOR ([RFC 7049](https://tools.ietf.org/html/rfc7049)) serialization, each ITMP message is serialized according to the CBOR specification.

### 5.3. Transports

ITMP assumes a transport with the following characteristics:

1. message-based
2. bidirectional (full-duplex)

There is no required transport or set of transports for ITMP implementations (but each implementation MUST, of course, implement at least one transport). Routers SHOULD implement more than one transport, enabling components using different kinds of transports to connect in an application.

#### 5.3.1. WebSocket Transport

The default transport binding for ITMP is WebSocket.
In the Basic Profile, ITMP messages are transmitted as WebSocket messages: each ITMP message is transmitted as a separate WebSocket message (not WebSocket frame). The Advanced Profile may define other modes, e.g. a *batched mode* where multiple ITMP messages are transmitted via single WebSocket message.
The ITMP protocol SHOULD BE negotiated during the WebSocket opening handshake between Peers using the WebSocket subprotocol negotiation mechanism.

ITMP uses the following WebSocket subprotocol identifiers for unbatched modes:

* "itmp.json"
* "itmp.cbor"

With "itmp.json", _all_ WebSocket messages MUST BE of type *text* (UTF8 encoded payload) and use the JSON message serialization.
With "itmp.cbor", _all_ WebSocket messages MUST BE of type *binary* and use the CBOR message serialization.
To avoid incompatibilities merely due to naming conflicts with WebSocket subprotocol identifiers, implementers SHOULD register identifiers for additional serialization formats with the official WebSocket subprotocol registry.

#### 5.3.2. Transport and Session Lifetime

ITMP implementations MAY choose to tie the lifetime of the underlying transport connection for a ITMP connection to that of a ITMP session, i.e. establish a new transport-layer connection as part of each new session establishment. They MAY equally choose to allow re-use of a transport connection, allowing subsequent ITMP sessions to be established using the same transport connection.
The diagram below illustrates the full transport connection and session lifecycle for an implementation which uses WebSocket over TCP as the transport and allows the re-use of a transport connection.

## 6. Messages

All ITMP messages are a "list" with a first element "MessageType" followed by one or more message type specific elements:
[MessageType:integer, ... one or more message type specific elements ...]

The notation "Element:type" denotes a message element named "Element" of type "type", where "type" is one of

* "uri": a string URI as defined in Section 5.1.1
* "id": an integer ID as defined in Section 5.1.2
* "integer": a non-negative integer
* "string": a Unicode string, including the empty string
* "bool": a boolean value ("true" or "false") - integers MUST NOT be used instead of boolean value
* "dict": a dictionary (map) where keys MUST be strings, keys MUST be unique and serialization order is undefined (left to the serializer being used)
* "list": a list (array) where items can be again any of this enumeration

### Example

A "SUBSCRIBE" message has the following format

```itmp
[SUBSCRIBE, Request:id, Topic:uri, Options:dict]
```

Here is an example message conforming to the above format

```json
[16, 713845233, "com.myapp.mytopic1", {}]
```

### 6.1. Extensibility

Some ITMP messages contain "Options:dict" elements. This allows for future extensibility and implementations that only provide subsets of functionality by ignoring unimplemented attributes. Keys in "Options" MUST be of type "string" and MUST match the regular expression "[a-z][a-z0-9_]{2,}" for ITMP predefined keys. Implementations MAY use implementation-specific keys that MUST match the regular expression "[a-z0-9_]{3,}". Attributes unknown to an implementation MUST be ignored.

### 6.2. Polymorphism

For a given "MessageType" the expected types are uniquely defined except the arguments. Hence there are polymorphic messages in ITMP.

### 6.3. Routing

If node needs to send message hrough router it adds address fields before command field. Router should test type of first message element and if it is byte or text strings should interpret it as destination address and if second filed also string it is source address.

### 6.4. Message Definitions

ITMP defines the following messages that are explained in detail in the following sections.
All messages are mandatory per role, i.e. in an implementation that only provides a Client with the role of Publisher MUST additionally implement sending "PUBLISH" and receiving "PUBLISHED" and "ERROR" messages.

#### 6.4.1. Session Lifecycle

ITMP not always uses sessions and in some implementation based on datagram transport can work in trusted environment without using sessions

##### 6.4.1.1. CONNECT

Sent by a Client to initiate opening of a ITMP session to a Server.

```itmp
[CONNECT, Request:id, peer_identifier:string, Options:dict]
```

in trusted environment CONNECT can be omitted, then client by default connected without extended features, mostly it is important to small mobile nodes without persistent connection

`Connection` is a random number and can be used to distinguish separate connection attempts and to transmit initial seed to generate session key for securing connection

##### 6.4.1.2. CONNECTED

Sent by a Server to accept a Client. The ITMP session is now open.

```itmp
[CONNECTED, CONNECT.Request:id, peer_identifier:string, Options:dict]
```

Session is a random number and can be used to distinguish separate connection attempts and to transmit initial seed to generate session key for securing connection

##### 6.4.1.4. DISCONNECT

Sent by a Peer to close a previously opened ITMP session. Must be echo'ed by the receiving Peer.

```itmp
[DISCONNECT, Code:integer, Reason:string, Options:dict]
```

#### 6.4.2. Service discovering

##### 6.4.2.1. DESCRIBE

Sent by peer to other peer to get peer/function/event description

```itmp
[DESCRIBE, Request:id, Topic:uri, Options:dict]
```

Sent by peer as answer to DESCRIBE message

```itmp
[RESULT, Request:id, description:list, Options:dict]
```

#### 6.4.2. Publish & Subscribe

##### 6.4.2.1. EVENT

Sent by a Publisher to a Subscriber/Broker to publish an event without acknowledge.

```itmp
[EVENT, Request:id, Topic:uri, Arguments, Options:dict]
```

An event is dispatched to a Subscriber for a given "Subscription:id" only once. On the other hand, a Subscriber that holds subscriptions with different "Subscription:id"s that all match a given event will receive the event on each matching subscription.

##### 6.4.2.1. PUBLISH

Sent by a Publisher to a Subscriber/Broker to publish an event with acknowledge awaiting.

```itmp
[PUBLISH, Request:id, Topic:uri, Arguments, Options:dict]
```

Acknowledge sent by a Broker to a Publisher for acknowledged publications.

```itmp
[RESULT, Request:id, Options:dict]
```

##### 6.4.2.3. SUBSCRIBE

Subscribe request sent by a Subscriber to a Broker to subscribe to a topic.

```itmp
[SUBSCRIBE, Request:id, Topic:uri, Options:dict]
```

Subscribe request was accepted.

```itmp
[RESULT, Request:id, Options:dict]
```

##### 6.4.2.5. UNSUBSCRIBE

Unsubscribe request sent by a Subscriber to a Broker to unsubscribe a subscription.

```itmp
[UNSUBSCRIBE, Request:id, Topic:uri, Options:dict]
```

Acknowledge sent by a Broker to a Subscriber to acknowledge a unsubscription.

```itmp
[RESULT, UNSUBSCRIBE.Request:id, Options:dict]
```

#### 6.4.3. Remote Procedure Calls

##### 6.4.3.1. CALL

Call as originally issued by the Caller.

```itmp
[CALL, Request:id, Procedure:uri, Arguments, Options:dict]
```

##### 6.4.3.2. ARGUMENTS

Provide additional arguments to the call to Callee during call execution.

```itmp
[ARGUMENTS, CALL.Request:id, Arguments, Options:dict]
```

##### 6.4.3.2. PROGRESS

Result of a call progress returned to Caller during call execution.

```itmp
[PROGRESS, CALL.Request:id, Result, Options:dict]
```

##### 6.4.3.2. CANCEL

Cancel the previously called function.

```itmp
[CANCEL, CALL.Request:id, Options:dict]
```

##### 6.4.3.2. RESULT

Result of a call as returned to Caller.

```itmp
[RESULT, CALL.Request:id, Result, Options:dict]
```

##### 6.4.3.2. ERROR

Result of a call as returned to Caller if the error occur during call execution.

```itmp
[ERROR, CALL.Request:id, error code:integer, TextError:string, Options:dict]`
```

#### 6.4.4. List of all messages and codes

| code | Format | desc |
| ---- | ----------- | -------- |
| | __Connection__
| 0 | [CONNECT, Request:id, peer_identifier:string, Options:dict] | open connection |
| 1 | [CONNECTED, CONNECT.Request:id, peer_identifier:string, Options:dict] | confirm connection
| 4 | [DISCONNECT, Code:integer, Reason:string, Options:dict] |  clear finish connection
| | __Error__
| 5 | [ERROR, Request:id, Code:integer, Reason:string, Options:dict] | error notification
| | __Result__
| 9 | [RESULT, Request:id, Result, Options:dict] | response notification
| | __Description__
| 6 | [DESCRIBE, Request:id, Topic:uri, Options:dict] | get description
| | __RPC__
| 8 | [CALL, Request:id, Procedure:uri, Arguments, Options:dict] | call
|  | __RPC Extended__
| 10 | [ARGUMENTS, CALL.Request:id, ARGUMENTS.Sequuence:integer, Arguments, Options:dict] | additional arguments for call
| 11 | [PROGRESS, CALL.Request:id, PROGRESS.Sequuence:integer, Result, Options:dict] | call in progress
| 12 | [CANCEL, CALL.Request:id, Options:dict] | call cancel
| | __publish__
| 13 | [EVENT, Request:id, Topic:uri, Arguments, Options:dict] | event
| 14 | [PUBLISH, Request:id, Topic:uri, Arguments, Options:dict] | event with acknowledge awaiting
| | __subscribe__
| 16 | [SUBSCRIBE, Request:id, Topic:uri, Options:dict] | subscribe
| 18 | [UNSUBSCRIBE, Request:id, Topic:uri, Options:dict]

### 6.6. Extension Messages

ITMP uses type codes from the core range [0, 18]. Implementations MAY define and use implementation specific messages with message type codes from the extension message range [35, 255]. For example, a router MAY implement router-to-router communication by using extension messages.

### 6.7. Empty Options

Implementations SHOULD avoid sending empty "Options" dicts.
E.g. a "CALL" message

```itmp
[CALL, Request:id, Procedure:uri, Arguments, Options:dict]
```

where " Options == {}" SHOULD be avoided, and instead

```itmp
[CALL, Request:id, Procedure:uri, Arguments]
```

SHOULD be sent.

### 6.7. Empty Arguments

Implementations SHOULD avoid sending empty "Arguments" lists if Options is empty.
E.g. a "CALL" message

```itmp
[CALL, Request:id, Procedure:uri, Arguments]
```

where "Arguments == []" SHOULD be avoided, and instead

```itmp
[CALL, Request:id, Procedure:uri]
```

SHOULD be sent.

## 7. Sessions

The message flow between Clients and Routers for opening and closing ITMP sessions involves the following messages:

1. "CONNECT"
2. "CONNECTED"
3. "DISCONNECT"

### 7.1. Session Establishment

#### 7.1.1. CONNECT

After the underlying transport has been established, the opening of a ITMP session is initiated by the Client sending a "CONNECT" message to the Router

```itmp
[CONNECT, Request:id, peer_identifier:string, Options:dict]
```

where

"Options" is a dictionary that allows to provide additional opening information (see below).
The "CONNECT" message MUST be the very first message sent by the Client after the transport has been established.
In the ITMP Basic Profile without session authentication the Router will reply with a "CONNECTED" or "ERROR" message.
A ITMP session starts its lifetime when the Router has sent a "CONNECTED" message to the Client, and ends when the underlying transport closes or when the session is closed explicitly by either peer sending the "DISCONNECT" message (see below).

It is a protocol error to receive a second "CONNECT" message during the lifetime of the session and the Peer must fail the session if that happens.

##### 7.1.1.1. Client: Role and Feature Announcement

ITMP uses _Role & Feature announcement_ instead of _protocol versioning_ to allow implementations only supporting subsets of functionality future extensibility
A Client can announce the roles it supports via "Connect.Options.roles:dict", with a key mapping to a "Connect.Options.roles.\<role\>:dict" where "\<role\>" can be:

* "publisher"
* "subscriber"
* "caller"
* "callee"

A Client can support any combination of the above roles but must support at least one role.
The "\<role\>:dict" is a dictionary describing features supported by the peer for that role.
This MUST be empty for ITMP Basic Profile implementations, and MUST be used by implementations implementing parts of the Advanced Profile to list the specific set of features they support.

_Example: A Client that implements the Publisher and Subscriber roles of the ITMP Basic Profile._

```json
[0, "some", {"roles": {"publisher": {},"subscriber": {}}}]
```

#### 7.1.2. CONNECTED

A Router completes the opening of a ITMP session by sending a "CONNECTED" reply message to the Client.

```itmp
[CONNECTED, CONNECT.Request:id, peer_identifier:string, Options:dict]

```

where
"Options" is a dictionary that allows to provide additional information regarding the open session (see below).
In the ITMP Basic Profile without session authentication, a "CONNECTED" message MUST be the first message sent by the Router, directly in response to a "CONNECT" message received from the Client. Extensions in the Advanced Profile MAY include intermediate steps and messages for authentication.
Note. The behavior if a requested "Realm" does not presently exist is router-specific. A router may e.g. automatically create the realm, or deny the establishment of the session with an "ABORT" reply message.

##### 7.1.2.1. Router: Role and Feature Announcement

Similar to a Client announcing Roles and Features supported in the "CONNECT" message, a Router announces its supported Roles and Features in the "CONNECTED" message.
A Router MUST announce the roles it supports via "Connected.Options.roles:dict", with a key mapping to a "Connected.Options.roles.\<role\>:dict" where "\<role\>" can be:

* "broker"
* "dealer"
* "publisher"
* "subscriber"
* "caller"
* "callee"

A Router must support at least one role, and MAY support all roles.
The "\<role\>:dict" is a dictionary describing features supported by the peer for that role. With ITMP Basic Profile implementations, this MUST be empty, but MUST be used by implementations implementing parts of the Advanced Profile to list the specific set of features they support

_Example: A Router implementing the Broker role of the ITMP Basic Profile._

```json
[2, 9129137332, "server", {"roles": {"broker": {}}}]
```

### 7.2. Session Closing

A ITMP session starts its lifetime with the Router sending a "CONNECTED" message to the Client and ends when the underlying transport disappears or when the ITMP session is closed explicitly by a "DISCONNECT" message sent by one Peer and a "DISCONNECT" message sent from the other Peer in response.

```itmp
[DISCONNECT, Code:integer, Reason:string, Options:dict]
```

where

"Code" MUST be an integer code of error.

"Options" MUST be a dictionary that allows to provide additional, optional closing information (see below).

_Example_. One Peer initiates closing

```json
[6, 503, "The host is shutting down now."]
```

and the other peer replies

```json
[6, 200, "connection closed"]
```

_Example_. One Peer initiates closing

```json
[6, 304, " close realm"]
```

and the other peer replies

```json
[6, 200, "connection closed"]
```

## 8. Agent Identification

When a software agent operates in a network protocol, it often identifies itself, its application type, operating system, software vendor, or software revision, by submitting a characteristic identification string to its operating peer.
Similar to what browsers do with the "User-Agent" HTTP header, both the "CONNECT" and the "CONNECTED" message MAY disclose the ITMP implementation in use to its peer:

CONNECT.Options.agent:string

and

CONNECTED.Options.agent:string

_Example: A Client "CONNECT" message._

```json
[1, "some", {
  "agent": "itmpJS-0.1.14",
  "roles": {"subscriber": {},"publisher": {}}
}]
```

_Example: A Router "CONNECTED" message._

```json
[2, 9129137332, "server", {
  "agent": "itmp.io-0.1.11",
  "roles": {"broker": {}}
}]
```

### 8.1 Node description

```itmp
<field description> ::= <name><description><signature>
<name> ::= string
<signature> ::= <interface> | <func> | <event>
<interface> ::= ':' [ <name> [ , <name> ] ]
<func> ::= '&' [<description>] ‘(‘ <argument type> ’)’ <return type>
<event> ::= '!' [<description>]<type>
<return type> ::= <type>
<argument type> ::= <type>
<type> ::= <simple type> | '['<type list>']'  | '<'<type list>'>' | '{' <name> [ '?' ] : <type> [ , <prop name> : <type> ] '}'
<type list> ::= <simple type> [, <simple type>]
<simple type> ::= <type name> [<description>]
<description>::='`'[<name>] [<version>] [<UniqueId>] [<units>] [<variants>] [<manufacturer>] [<comments>]'`'
<version>::=’%’ string
<UniqueId>::=’#’ string
<units> ::= '$' string
<variants> ::= '<' <value> [ , <value> ] '>' | '[' <minimum> '..' <maximum> ']'
<manufacturer> ::= '@' string
<comments> ::= '/' string
<type char> ::= n | num | number | i | int | s | str | string | b | bool | f | float | i8 | i16 | i32 | i64 | u8 | u16 | u32 | u64 | f32 | d64
```

simple type JSON and CBOR encoded inside array and map denoted by '[]' and '{}'

* n,num,number - float or integer number
* i,int,integer - integer number
* f,float - floating point number
* s,str,string - UTF-8 string
* b, bool - boolean value

fixed types JSON BASE64 string or CBOR byte array encoded as sequence (for small controllers without complex encoders) denoted by '<>'

* u8 - 8-bit unsigned integer (byte)
* i8 - 8-bit signed integer
* i16 - 16-bit signed integer
* u16 - 16-bit unsigned integer
* i32 - 32-bit signed integer
* u32 - 32-bit unsigned integer
* i64 - 64-bit signed integer
* u64 - 64-bit unsigned integer
* f32 - single-precision floating point (IEEE 754)
* d64 - double-precision floating point (IEEE 754)

any sequence of primitive typed values encoded as byte array or base64 encoded string for json

_Example_

```itmp
go & `set pwm for wheels drivers`( <i16`left pwm[-1280..1280]`,i16`right pwm[-1280..1280]`>) : <i32`left position`, i32`right position`, i32`time stamp$us`, i16`echolocator distance$mm`>

to & `set target position` (<i32`left target pos$ticks`, i32`right target pos$ticks`, i32`max speed$ticks per sec`,i32`acceleration$ticks per sec per sec`>) <i32`left position`,i32`right position`,i32`time stamp$us`,i32`echolocator distance$mm`>

sp &`set moving speed` (<i32`left speed$ticks per sec`,i32`right speed$ticks per sec`, i32`acceleration$ticks per sec per sec`>) : <i32`left position`,i32`right position,i32`time stamp$us`, i32`echolocator distance$mm`>

set &`set servos position` (<u16`srvo1$us[1000..2000]`,u16`servo2$us[1000..2000]`>)

stat & () <u16`supplyvoltage`,u8`global status`>
```

## 9. Publish and Subscribe

All of the following features for Publish & Subscribe are mandatory for ITMP Basic Profile implementations supporting the respective roles, i.e. _Publisher_, _Subscriber_.

### 9.1. Topics

A topic is a UTF-8 string, which is used by the broker to filter messages for each connected client. A topic consists of one or more topic levels. Each topic level is separated by a dot (topic level separator).

In comparison to a message queue a topic is very lightweight. There is no need for a client to create the desired topic before publishing or subscribing to it, because a broker accepts each valid topic without any prior initialization.

Here are a few example topics:

    myhome.groundfloor.livingroom.temperature
    USA.California.San Francisco.Silicon Valley
    5ff4a2ce-e485-40f4-826c-b1a5d81be9b6.status
    Germany.Bavaria.car.2382340923453.latitude

Noticeable is that each topic must have at least 1 character to be valid and it can also contain spaces. Also a topic is case-sensitive, which makes myhome.temperature and MyHome.Temperature two individual topics. Additionally the dot alone is a valid topic, too.

### 9.2. Wildcards

When a client subscribes to a topic it can use the exact topic the message was published to or it can subscribe to more topics at once by using wildcards. A wildcard can only be used when subscribing to topics and is not permitted when publishing a message. In the following we will look at the two different kinds one by one: single level and multi-level wildcards.

__Single Level: +__

As the name already suggests, a single level wildcard is a substitute for one topic level. The plus symbol represents a single level wildcard in the topic.

Any topic matches to a topic including the single level wildcard if it contains an arbitrary string instead of the wildcard. For example a subscription to myhome.groundfloor.+.temperature would match or not match the following topics:

```itmp
+  myhome.groundfloor.kitchen.temperature
+  myhome.groundfloor.livingroom.temperature
-  myhome.groundfloor.livingroom.brightness
-  myhome.firstfloor.kitchen.temperature
-  myhome.groundfloor.kitchen.fridge.temperature
```

__Multi-Level: #__

While the single level wildcard only covers one topic level, the multi-level wildcard covers an arbitrary number of topic levels. In order to determine the matching topics it is required that the multi-level wildcard is always the last character in the topic and it is preceded by a dot.

`myhome.groundfloor.#`

```itmp
+  myhome.groundfloor.kitchen.temperature
+  myhome.groundfloor.livingroom.temperature
+  myhome.groundfloor.livingroom.brightness
+  myhome.groundfloor.kitchen.fridge.temperature
-  myhome.firstfloor.kitchen.temperature
```

A client subscribing to a topic with a multi-level wildcard is receiving all messages, which start with the pattern before the wildcard character, no matter how long or deep the topics will get. If you only specify the multilevel wildcard as a topic (#), it means that you will get every message sent over the broker. If you expect high throughput this is an anti-pattern.

#### When topic-based wildcards are not wild

The wildcard characters '+' and '#' have no special meaning when they are mixed with other characters (including themselves) in a topic level.

For ITMP topics, consider the following examples:

    level0.level1.+.level4/#
    level0.level1.#.level4.level+

In the first example, the characters '+' and '#' are treated as a wildcards and are therefore not valid in a topic string that is to be published to, but is valid in a subscription.

In the second example, the character '+' is not the only character in a topic level. The character '#' is not the last character in the topic string. Therefore the topic string cannot be published or subscribed to.

The following table provides examples of topic strings, and shows whether these strings are valid for ITMP publish/subscribe.

Topic string 	| ITMP subscribe |	ITMP publish
-------------|-------------|---------------
\# |    Yes |    No
\+ |    Yes |    No
.# |	Yes |	No
.+ |	Yes |	No
### |	No |	No
++ |	No |	No
#.# |	No |	No
+.+ |	Yes |	No
topic# |	No |	No
topic+ |	No |	No
topic.# |	Yes |	No
topic.+ |	Yes |	No
topic## |	No |	No
topic++ |	No |	No
topic.## |	No |	No
topic.++ |	No |	No
topic#.# |	No |	No
topic+.+ |	No |	No
topic.#.# |	No |	No
topic.+.+ |	Yes |	No
#topic |	No |	No
+topic |	No |	No
#.topic |	No |	No
+.topic |	Yes |	No
.#topic |	No |	No
.+topic |	No |	No
. |	Yes |	Yes
topic.#.topic |	No |	No
top+ic |	No |	No

#### 9.2.1 Best practices

##### Don’t use a leading dot

It is allowed to use a leading dot in ITMP, for example .myhome.groundfloor.livingroom. But that introduces a unnecessary topic level with a zero character at the front. That should be avoided, because it doesn’t provide any benefit and often leads to confusion.

##### Don’t use spaces in a topic

A space is the natural enemy of each programmer, they often make it much harder to read and debug topics, when things are not going the way, they should be. So similar to the first one, only because something is allowed doesn’t mean it should be used. UTF-8 knows many different white space types, it’s pretty obvious that such uncommon characters should be avoided.

##### Keep the topic short and concise

Each topic will be included in every message it is used in, so you should think about making them short and concise. When it comes to small devices, each byte counts and makes really a difference.

##### Use only ASCII characters, avoid non printable characters

Using non-ASCII UTF-8 character makes it really hard to find typos or issues related to the character set, because often they cannot be displayed correctly. Unless it is really necessary we recommend avoid using non-ASCII character in a topic.

##### Embed a unique identifier or the ClientId into the topic

In some cases it is very helpful, when the topic contains a unique identifier of the client the publish is coming from. This helps identifying, who send the message. Another advantage is the enforcement of authorization, so that only a client with the same ClientId as contained in the topic is allowed to publish to that topic. So a client with the id client1 is allowed to publish to client1.status, but not permitted to publish to client2.status.

##### Don’t subscribe to \#

Sometimes it is necessary to subscribe to all messages, which are transferred over the broker, for example when persisting all of them into a database. This should not be done by using a ITMP client and subscribing to the multi level wildcard. The reason is that often the subscribing client is not able to process the load of messages that is coming its way. Especially if you have a massive throughput. Our recommended solution is to implement an extension in the ITMP broker.

##### Don’t forget extensibility

Topics are a flexible concept and there is no need to preallocate them in any kind of way, regardless both the publisher and subscriber need to be aware of the topic. So it is important to think about how they can be extended in case you are adding new features to your product. For example when your smart home solution is extended by some new sensors, it should be possible to add these to your topic tree without changing the whole topic hierarchy.

##### Use specific topics, instead of general ones

When naming topics it is important not to use them like a queue, for example using only one topic for all messages is a anti pattern. You should use as specific topics as possible. So if you have three sensors in your living room, you should use topics myhome.livingroom.temperature, myhome.livingroom.brightness and myhome.livingroom.humidity, instead of sending all values over myhome.livingroom. Also this enables you to use other ITMP features like retained messages, which we cover in one of the next posts.

### 9.3. Subscribing and Unsubscribing

The message flow between Clients implementing the role of Subscriber and Routers implementing the role of Broker for subscribing and unsubscribing involves the following messages:

1. "SUBSCRIBE"
2. "UNSUBSCRIBE"
3. "RESULT"
4. "ERROR"

A Subscriber may subscribe to zero, one or more topics, and a Publisher publishes to topics without knowledge of subscribers.
Upon subscribing to a topic via the "SUBSCRIBE" message, a Subscriber will receive any future events published to the respective topic by Publishers, and will receive those events asynchronously.
A subscription lasts for the duration of a session, unless a Subscriber opts out from a previously established subscription via the "UNSUBSCRIBE" message.
A Subscriber may have more than one event handler attached to the same subscription. This can be implemented in different ways: a) a Subscriber can recognize itself that it is already subscribed and just attach another handler to the subscription for incoming events, b) or it can send a new "SUBSCRIBE" message to broker (as it would be first) and upon receiving a "SUBSCRIBED.Subscription:id" it already knows about, attach the handler to the existing subscription

#### 9.3.1. SUBSCRIBE

A Subscriber communicates its interest in a topic to a Broker by sending a "SUBSCRIBE" message:

```itmp
[SUBSCRIBE, Request:id, Topic:uri, Options:dict]
```

where

"Request" MUST be a random, ephemeral ID chosen by the Subscriber and used to correlate the Broker's response with the request.

"Options" MUST be a dictionary that allows to provide additional subscription request Options in an extensible way. This is described further below.

"Topic" is the topic the Subscriber wants to subscribe to and MUST be an URI.

_Example_

```json
[16, 713845233, "com.myapp.mytopic1", {}]
```

A Broker, receiving a "SUBSCRIBE" message, can fulfill or reject the subscription, so it answers with "SUBSCRIBED" or "ERROR" messages.

If the Broker is able to fulfill and allow the subscription, it answers by sending a "SUBSCRIBED" message to the Subscriber

```itmp
[RESULT, SUBSCRIBE.Request:id, Subscription:id]
```

where

"SUBSCRIBE.Request" MUST be the ID from the original request.

"Subscription" MUST be an ID chosen by the Broker for the subscription.

_Example_

```json
[9, 713845233, 5233]
```

Note. The "Subscription" ID chosen by the broker need not be unique to the subscription of a single Subscriber, but may be assigned to the "Topic", or the combination of the "Topic" and some or all "Options", such as the topic pattern matching method to be used. Then this ID may be sent to all Subscribers for the "Topic" or "Topic" / "Options" combination. This allows the Broker to serialize an event to be delivered only once for all actual receivers of the event.
In case of receiving a "SUBSCRIBE" message from the same Subscriber and to already subscribed topic, Broker should answer with "SUBSCRIBED" message, containing the existing "Subscription:id".

When the request for subscription cannot be fulfilled by the Broker, the Broker sends back an "ERROR" message to the Subscriber

```itmp
[ERROR, SUBSCRIBE.Request:id, Code:integer, Description:string, Options:dict]
```

where

"SUBSCRIBE.Request" MUST be the ID from the original request.

"Code" MUST be an code that gives the error of why the request could not be fulfilled.

"Description" MUST be a string that gives the error of why the request could not be fulfilled.

_Example_

```json
[5, 713845233, 401, "not authorized"]
```

#### 9.3.4. UNSUBSCRIBE

When a Subscriber is no longer interested in receiving events for a subscription it sends an "UNSUBSCRIBE" message

```itmp
[UNSUBSCRIBE, Request:id, Topic:uri, Options:dict]
```

where

"Request" MUST be a random, ephemeral ID chosen by the Subscriber and used to correlate the Broker's response with the request.

"SUBSCRIBED.Subscription" MUST be the ID for the subscription to unsubscribe from, originally handed out by the Broker to the Subscriber.

_Example_

```json
[18, 85346237, "com.myapp.mytopic1"]
```

Upon successful unsubscription, the Broker sends an "UNSUBSCRIBED" message to the Subscriber

```itmp
[RESULT, UNSUBSCRIBE.Request:id]
```

where

"UNSUBSCRIBE.Request" MUST be the ID from the original request.

_Example_

```json
[9, 85346237]
```

When the request fails, the Broker sends an "ERROR"

```itmp
[ERROR, UNSUBSCRIBE.Request:id, Code:integer, Description:string, Options:dict]
```

where

"UNSUBSCRIBE.Request" MUST be the ID from the original request.

"Code" MUST be an code that gives the error of why the request could not be fulfilled.

"Description" MUST be a string that gives the error of why the request could not be fulfilled.

_Example_

```json
[5, 85346237, 404, "No such subscription"]
```

### 9.4. Publishing and Events

The message flow between Publishers, a Broker and Subscribers for publishing to topics and dispatching events involves the following messages:

1. "EVENT"
2. "PUBLISH"
3. "RESULT"
4. "ERROR"

#### 9.4.1. EVENT

When a Publisher requests to publish an event to some topic, it sends an "EVENT" message to a Broker:

```itmp
[EVENT, Request:id, Topic:uri, Arguments, Options:dict]
```

where

"Request" is a random, ephemeral ID chosen by the Publisher and used to correlate the Broker's response with the request.

"Topic" is the topic published to.

"Arguments" is a list or dictionary of application-level event payload elements. The list or dictionary may be of zero length.

"Options" is a dictionary that allows to provide additional publication request details in an extensible way. This is described further below.

That publications are unacknowledged, and the Broker will not respond, whether the publication was successful indeed or not.

_Example_

```json
[13, 239714735, "com.myapp.mytopic1"]
```

_Example_

```json
[13, 239714735, "com.myapp.mytopic1", ["Hello, world!"]]
```

_Example_

```json
[13, 239714735, "com.myapp.mytopic1", {"color": "orange", "sizes": [23, 42, 7]}]
```

##### Best Practice

Use Event when …

* You have a complete or almost stable connection between sender and receiver. A classic use case is when connecting a test client or a front-end application to a MQTT broker over a wired connection.
* You don’t care if one or more messages are lost once a while. That is sometimes the case if the data is not that important or will be send at short intervals, where it is okay that messages might get lost.
* You don’t need any message queuing. Messages are only queued for disconnected clients if they have subscribed for publishable topics.

#### 9.4.1. PUBLISH

When a Publisher requests to publish an event to some topic, it sends a "PUBLISH" message to a Broker:

```itmp
[PUBLISH, Request:id, Topic:uri, Arguments, Options:dict]
```

where

"Request" is a random, ephemeral ID chosen by the Publisher and used to correlate the Broker's response with the request.

"Topic" is the topic published to.

"Arguments" is a list or dictionary of application-level event payload elements. The list or dictionary may be of zero length.

"Options" is a dictionary that allows to provide additional publication request details in an extensible way. This is described further below.

If the Broker is able to fulfill and allowing the publication, the Broker will send the event to all current Subscribers of the topic of the published event.
Publications are acknowledged, and the Broker will respond, depends the publication was successful indeed or not.

_Example_

```json
[14, 239714735, "com.myapp.mytopic1"]
```

_Example_

```json
[14, 239714735, "com.myapp.mytopic1", ["Hello, world!"]]
```

_Example_

```json
[14, 239714735, "com.myapp.mytopic1", {"color": "orange", "sizes": [23, 42, 7]}]
```

##### Best Practice

Use Publish when …

* It is critical to your application to receive all messages exactly once. This is often the case if a duplicate delivery would do harm to application users or subscribing clients. You should be aware of the overhead and that it takes a bit longer to complete the publish flow.

If the Broker is able to fulfill and allowing the publication of PUBLISH message, the Broker replies by sending a "PUBLISHED" message to the Publisher:

```itmp
[RESULT, PUBLISH.Request:id, Publication:id]
```

where

"PUBLISH.Request" is the ID from the original publication request.

"Publication" is an ID chosen by the Broker for the publication.

_Example_

```json
[9, 239714735, 4429313566]
```

#### 9.4.3. Publish ERROR

When the PUBLISH request for publication cannot be fulfilled by the Broker, the Broker sends back an "ERROR" message to the Publisher

```itmp
[ERROR, PUBLISH.Request:id, Code:integer, Description:string, Options:dict]
```

where

"PUBLISH.Request" is the ID from the original publication request.

"Code" is a number that gives the error of why the request could not be fulfilled.

_Example_

```json
[5, 239714735, 401, " not authorized"]
```

## 10. Remote Procedure Calls

All of the following features for Remote Procedure Calls are mandatory for ITMP Basic Profile implementations supporting the respective roles.

### 10.2. Calling

The message flow between Callers, a Dealer and Callees for calling procedures and invoking endpoints involves the following messages:

1. "CALL"
2. "ARGUMENTS"
3. "PROGRESS"
4. "CANCEL"
5. "RESULT"
6. "ERROR"

The execution of remote procedure calls is asynchronous, and there may be more than one call outstanding. A call is called outstanding (from the point of view of the Caller), when a (final) result or error has not yet been received by the Caller.

#### 10.2.1. CALL

When a Caller wishes to call a remote procedure, it sends a "CALL" message to a Dealer:

```itmp
[CALL, Request:id, Procedure:uri, Arguments, Options:dict]
```

where

"Request" is a random, ephemeral ID chosen by the Caller and used to correlate the Dealer's response with the request.

"Options" is a dictionary that allows to provide additional call request details in an extensible way. This is described further below.

"Procedure" is the URI of the procedure to be called.
if uri denote event topic, CALL make polling for event and can be used for get an event without subscribing,in such a way RESULT bring Event if so and ERROR will return if no message.

"Arguments" is a list of positional call arguments (each of arbitrary type) or dictionary of keyword call arguments. The Arguments may be empty or omitted.The Arguments cannot be null because it marks following arguments message present.

_Example_

```json
[8, 7814135, "com.myapp.ping"]
```

_Example_

```json
[8, 7814135, "com.myapp.echo", ["Hello, world!"]]
```

_Example_

```json
[8, 7814135, "com.myapp.add2", [23, 7]]
```

_Example_

```json
[8, 7814135, "com.myapp.user.new", {"firstname": "John", "surname": "Doe"}]
```

#### 10.2.3. RESULT

If the Callee is able to successfully process and finish the execution of the call, it answers by sending a "RESULT" message to the Dealer:

```itmp
[RESULT, CALL.Request:id, Arguments, Options:dict]
```

where

"CALL.Request" is the ID from the original invocation request.

"Arguments" is a list of positional result elements (each of arbitrary type) or dictionary of keyword result elements (each of arbitrary type). The Arguments may be empty or even omitted.

"Options"is a dictionary that allows to provide additional options.

_Example_

```json
[9, 6131533]
```

_Example_

```json
[9, 6131533, ["Hello, world!"]]
```

_Example_

```json
[9, 6131533, [30]]
```

_Example_

```json
[9, 6131533, {"userid": 123, "karma": 10}]
```

#### 10.2.3. ARGUMENTS

If the Caller should send a lot of arguments or produce the arguments for long time or the arguments is too big to send it as single message, it sends several "ARGUMENTS" message to the Dealer after the CALL message:

```itmp
[CALL, Request:id, Procedure:uri, null, Options:dict]
[ARGUMENTS, CALL.Request:id, 0, Arguments, Options:dict]
[ARGUMENTS, CALL.Request:id, 1, Arguments, Options:dict]
[ARGUMENTS, CALL.Request:id, 2]
[RESULT, CALL.Request:id, Arguments, Options:dict]
```

where

"CALL.Request" is the ID from the original invocation request.

"Arguments" is a list of positional result elements (each of arbitrary type) or dictionary of keyword result elements (each of arbitrary type). The Arguments may be empty or even omitted.

"Options"is a dictionary that allows to provide additional options.

_Example_

```json
->[8, 6131533,"fill",null]
->[10, 6131533,0,[5]]
->[10, 6131533,1,[25]]
->[10, 6131533,2,[65]]
->[10, 6131533,3]
<-[9, 6131533]
```

_Example_

```json
->[8, 6131533, "add user", null]
->[10, 6131533, 0, {"userid": 123}]
->[10, 6131533, 1, {"karma": 10}]
->[10, 6131533, 2, {"level": 3}]
->[10, 6131533, 3]
<-[9, 6131533]
```

#### 10.2.3. PROGRESS

If the Callee produce the result for long time or part by part or the result is too big to send it as single message, it answers by sending several "PROGRESS" message to the Dealer before the RESULT message sent:

```itmp
[PROGRESS, CALL.Request:id, 0, Arguments, Options:dict]
[PROGRESS, CALL.Request:id, 1, Arguments, Options:dict]
[PROGRESS, CALL.Request:id, 2, Arguments, Options:dict]
[PROGRESS, CALL.Request:id, 3, Arguments, Options:dict]
[RESULT, CALL.Request:id, Arguments, Options:dict]
```

where

"CALL.Request" is the ID from the original invocation request.

"Arguments" is a list of positional result elements (each of arbitrary type) or dictionary of keyword result elements (each of arbitrary type). The Arguments may be empty or even omitted.

"Options" is a dictionary that allows to provide additional options.

_Example_

```json
<-[11, 6131533,0,["5%"]]
<-[11, 6131533,1,["25%"]]
<-[11, 6131533,2,["65%"]]
<-[11, 6131533,3,["95%"]]
<-[9, 6131533]
```

_Example_

```json
<-[11, 6131533, 0, ["preparing for Hello, world!"]]
<-[9, 6131533, ["Hello, world!"]]
```

_Example_

```json
<-[11, 6131533, 0, [330]]
<-[11, 6131533, 1, [530]]
<-[11, 6131533, 2, [360]]
<-[9, 6131533, [30]]
```

_Example_

```json
<-[11, 6131533, 0, {"userid": 123}]
<-[11, 6131533, 1, {"karma": 10}]
<-[11, 6131533, 2, {"level": 3}]
<-[9, 6131533, {}]
```

#### 10.2.3. CANCEL

If the Callee produce the result using PROGRESS messages Caller can stop the call by sending "CANCEL" message to the Dealer:

`[PROGRESS, CALL.Request:id, 0, Arguments, Options:dict]`

`[PROGRESS, CALL.Request:id, 1, Arguments, Options:dict]`

`[CANCEL, CALL.Request:id, Options:dict]`

`[RESULT, CALL.Request:id, Arguments, Options:dict]`

where

"CALL.Request" is the ID from the original invocation request.

"Arguments" is a list of positional result elements (each of arbitrary type) or dictionary of keyword result elements (each of arbitrary type). The Arguments may be empty or even omitted.

"Options" is a dictionary that allows to provide additional options.

_Example_

`[11, 6131533,0,["5%"]]`

`[11, 6131533,1,["25%"]]`

`[12, 6131533]`

`[9, 6131533]`

_Example_

`[11, 6131533, 0, [330]]`

`[12, 6131533, 1, [530]]`

`[5, 6131533, 409, "Process aborted"]`

#### 10.2.5. ERROR

If the Callee is unable to process or finish the execution of the call, or the application code implementing the procedure raises an exception or otherwise runs into an error, the Callee sends an "ERROR" message to the Dealer:

`[ERROR, CALL.Request:id, Code:integer, Description:string, Options:dict]`

where

"CALL.Request" is the ID from the original "CALL" request previously sent by the Dealer to the Callee.

"Options" is a dictionary with additional error Options.

"Code" is an integer that identifies the error of why the request could not be fulfilled.

_Example_

`[5, 6131533, 504, "time out", {"severity": 3}]`

## 11. Error codes

ITMP pre-defines the following error codes. ITMP peers MUST use only the defined error messages.

* 304 Not Modified
* 306 No Event (if CALL to event topic but no event this time)

* 400 Bad Request
* 401 Unauthorized
* 403 Forbidden
* 404 Not Found
* 405 Method Not Allowed
* 406 Not Acceptable
* 409 Conflict.
* 410 Gone.
* 413 Request Entity Too Large
* 414 Request-URI Too Large
* 418 I'm a teapot
* 419 Format error
* 420 Type error
* 429 Too Many Requests

* 500 Internal Server Error
* 501 Not Implemented
* 507 Insufficient Storage

### 11.1. 400 Bad request

When a Peer provides an incorrect URI for any URI-based attribute of a ITMP message (e.g. realm, topic), then the other Peer MUST respond with an "ERROR" message and give the code 400 for such requests

### 11.2. 401 Unauthorized

When a Peer needs authorization they MUST respond with an "ERROR" message and give the code 401 for any requests except CONNECT
A join, call, register, publish or subscribe failed, since the Peer is not authorized to perform the operation.

A Dealer or Broker could not determine if the Peer is authorized to perform a join, call, register, publish or subscribe, since the authorization operation _itself_ failed. E.g. a custom authorizer did run into an error.

### 11.3. 403 Forbidden

When a Peer restrict using some methods or arguments they MUST respond with an "ERROR" message and give the code 403 for any requests that violate restrictions

### 11.4. 404 Not Found

A Node could not perform a call or other action, since no procedure is currently registered under the given URI.

### 11.5. 405 Method Not Allowed

A Node could not perform a call or other action, since type of uri is not intended to called function (for example CALL for event uri).

### 11.6. 406 Not Acceptable

A Node could not perform a call or other action, since type of uri is not intended to called function (for example CALL for event uri).

### 11.7. 413 Request Entity Too Large

A Node could not perform a call or other action, since message is too large.

### 11.8. 414 Request-URI Too Large

A Node could not perform a call or other action, since message is too large.

### 11.9. 418 I'm a teapot

A Node is teapot.

### 11.10. 429 Too Many Requests

A Node could not perform a call or other action, since message is coming to fast.

### 11.11. 419 Format error

A Node could not perform a call or other action, since message has bad format.

### 11.12. 420 Type error

A Node could not perform a call or other action, since message components has bad type.

### 11.13. 421 Reserved...

reserved code

### 11.14. 500 Internal Server Error

A Node could not perform a call or other action, since internal error.

### 11.15. 501 Not Implemented

A Node could not perform a call or other action, since call is not implemented.

### 11.16. 507 Insufficient Storage

A Node could not perform a call or other action, since no storage space.

### 11.17. 512 Bag argument

A call failed since the given argument types or values are not acceptable to the called procedure. In this case the Callee may throw this error. Alternatively a Router may throw this error if it performed _payload validation_ of a call, call result or publish, and the payload did not conform to the requirements.

### 11.18. 513 Session Close

The Peer is shutting down completely - used as a "DISCONNECT" (or "ABORT") reason.

### 11.19. 513 system shutdown

The Peer is shutting down completely - used as a "DISCONNECT" (or "ABORT") reason.

### 11.21. 514 close realm

A Peer acknowledges ending of a session - used as a "DISCONNECT" reply reason.

### 11.22. 202 goodbye and out

## 12. Examples

### 12.1  Simple example

| Client Send | Server send |
| --- | --- |
| `[DESCRIBE, 0, ""]` | ```[RESULT, 0, ["FireGuard`Fire Alarm and automatic Destiguishing board%1.0.10.435#231268834874553@NSC Communication Siberia`:Node,Fireguard","getState&","StateChanged!"] ]``` |
| `[DESCRIBE, 1, "getState"]` | ```[RESULT, 1, ["getState`Get Area State`&s`Area state`(s`Area name`)"] ]``` |
| `[DESCRIBE, 2, "getState"]` | ```[RESULT, 2, ["StateChanged`Inform about new state of area`!s`Area name`s`Area state`"] ]``` |
| `[CALL, 3, "getState", ["Area1"] ]` | ```[RESULT, 3, ["Norm"]]``` |
| `-` | ```[EVENT, 0, "StateChanged" ["Area1","Alarm"]]``` |
| `-` | ```[PUBLISH, 1, "StateChanged" ["Area1","Norm"]]``` |
| `[RESULT,1]` | `-` |

### 12.2  Temperature sensor example

| Sensor Send | Server send |
| --- | --- |
| ```[CONNECT, 0, "TemperatureSensor`Smart Termometer%1.0#A2DF31CD@NSC Communication Siberia`:Sensor"]``` | ```[CONNECT, 0]``` |
| `-` | ```[SUBSCRIBE, 0, ["Temp"] ]``` |
| `[RESULT, 0, 0]` | `-` |
| ```[EVENT, 0, "Temp" ["A2DF31CD",24]``` | `-`
| ```[PUBLISH, 1, "Temp" ["A2DF31CD",23]``` | `[RESULT,1]` |

## 13. Ordering Guarantees

All ITMP implementations, in particular Routers MUST support the following ordering guarantees.
A ITMP Advanced Profile may provide applications options to relax ordering guarantees, in particular with distributed calls.

### 13.1. Publish & Subscribe Ordering

Regarding *Publish & Subscribe*, the ordering guarantees are as follows:
If _Subscriber A_ is subscribed to both *Topic 1* and *Topic 2*, and _Publisher B_ first publishes an *Event 1* to *Topic 1* and then an *Event 2* to *Topic 2*, then _Subscriber A_ will first receive *Event 1* and then *Event 2*. This also holds if *Topic 1* and *Topic 2* are identical.
In other words, ITMP guarantees ordering of events between any given _pair_ of Publisher and Subscriber.
Further, if _Subscriber A_ subscribes to *Topic 1*, the "SUBSCRIBED" message will be sent by the _Broker_ to _Subscriber A_ before any "EVENT" message for *Topic 1*.
There is no guarantee regarding the order of return for multiple subsequent subscribe requests. A subscribe request might require the _Broker_ to do a time-consuming lookup in some database, whereas another subscribe request second might be permissible immediately.

### 13.2. Remote Procedure Call Ordering

Regarding *Remote Procedure Calls*, the ordering guarantees are as follows:
If _Callee A_ has registered endpoints for both *Procedure 1* and *Procedure 2*, and _Caller B_ first issues a *Call 1* to *Procedure 1* and then a *Call 2* to *Procedure 2*, and both calls are routed to _Callee A_, then _Callee A_ will first receive an invocation corresponding to *Call 1* and then *Call 2*. This also holds if *Procedure 1* and *Procedure 2* are identical.
In other words, ITMP guarantees ordering of invocations between any given _pair_ of Caller and Callee.
There are no guarantees on the order of call results and errors in relation to _different_ calls, since the execution of calls upon different invocations of endpoints in Callees are running independently. A first call might require an expensive, long-running computation, whereas a second, subsequent call might finish immediately.

## 14. Security Model

The following discusses the security model for the Basic Profile. Any changes or extensions to this for the Advanced Profile are discussed further on as part of the Advanced Profile definition.

### 14.1. Transport Encryption and Integrity

ITMP transports may provide (optional) transport-level encryption and integrity verification. If so, encryption and integrity is point-to- point: between a Client and the Router it is connected to.
Transport-level encryption and integrity is solely at the transport- level and transparent to ITMP. ITMP itself deliberately does not specify any kind of transport-level encryption.
Implementations that offer TCP based transport such as ITMP-over- WebSocket or ITMP-over-RawSocket SHOULD implement Transport Layer Security (TLS).
ITMP deployments are encouraged to stick to a TLS-only policy with the TLS code and setup being hardened.
Further, when a Client connects to a Router over a local-only transport such as Unix domain sockets, the integrity of the data transmitted is implicit (the OS kernel is trusted), and the privacy of the data transmitted can be assured using file system permissions (no one can tap a Unix domain socket without appropriate permissions or being root).

### 14.2. Router Authentication

To authenticate Routers to Clients, deployments MUST run TLS and Clients MUST verify the Router server certificate presented. ITMP itself does not provide mechanisms to authenticate a Router (only a Client).
The verification of the Router server certificate can happen

1. against a certificate trust database that comes with the Clients operating system
2. against an issuing certificate/key hard-wired into the Client
3. by using new mechanisms like DNS-based Authentication of Named Entities (DNSSEC)/TLSA

Further, when a Client connects to a Router over a local-only transport such as Unix domain sockets, the file system permissions can be used to create implicit trust. E.g. if only the OS user under which the Router runs has the permission to create a Unix domain socket under a specific path, Clients connecting to that path can trust in the router authenticity.

### 14.3. Client Authentication

Authentication of a Client to a Router at the ITMP level is not part of the basic profile.
When running over TLS, a Router MAY authenticate a Client at the transport level by doing a _client certificate-based authentication_.

#### 14.3.1. Routers are trusted

Routers are _trusted_ by Clients.
In particular, Routers can read (and modify) any application payload transmitted in events, calls, call results and call errors.
Hence, Routers do not provide confidentiality with respect to application payload, and also do not provide authenticity or integrity of application payloads that could be verified by a receiving Client.
Routers need to read the application payloads in cases of automatic conversion between different serialization formats.
Further, Routers are trusted to *actually perform* routing as specified. E.g. a Client that publishes an event has to trust a Router that the event is actually dispatched to all (eligible) Subscribers by the Router.
A rogue Router might deny normal routing operation without a Client taking notice.
