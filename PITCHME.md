# Making the Web Conversational

## How Web protocols like HTTP have evolved to support modern Web applications.

---

*This talk is non-normative.*

Fuzzy Buzz-words: Real-time Interactive Collaborative Bidirectional

---

## about.me/vipulmathur

**Coordinates**

* Twitter: [@VipulMathur](https://twitter.com/VipulMathur)
* LinkedIn: https://in.linkedin.com/in/VipulMathur

---

## Topics

---

### Why

- Various forms of communication
- Web protocols

---

### What

- Overview using plain HTTP
- [RFC6202](https://tools.ietf.org/html/rfc6202): Known Issues and Best Practices for the Use of Long Polling and Streaming in Bidirectional HTTP
- [RFC6455](https://tools.ietf.org/html/rfc6455): The WebSocket Protocol

---

### How

- A quick demo of WebSockets
- Pointers to frameworks/ libraries
- Not the focus for #RFCsWeLove

---

## Forms of Communication

1. Unicast (1 to 1, simplex)
    - e.g. letter (snail-mail)
2. Multicast (1 to many, simplex)
    - e.g. email
3. Broadcast (1 to any, simplex)
    - e.g. TV/ radio
4. Request-response (1 to 1, half duplex)
    - e.g. Q&A like in an interview
5. Conversation (1 to 1, full duplex)
    - e.g. chat between friends
6. Omnidirectional (many to many)
    - e.g. honking in Bangalore traffic :)

---

### Opinion
- Web protocols are mostly 4, increasingly 5
- Web applications are of all types, built on top of Web protocols

---

## HTTP

### Relevant Recap
- request-response based
- plain-text headers, body
- pipelining (multiple requests without waiting for response on same TCP connection)
- persistent connections (multiple request-response pairs on same TCP connection)
- chunked transfer encoding (allow response to be broken into chunks, allow headers after body)

---

### Bi-directional communication over HTTP

**Approaches**

- polling
- long poll
- streaming
- HTTP/2

**Pointers**

- BOSH
- Comet
- Bayou

---

## HTTP Long Polling

### Flow

1.  The client makes an **initial request** and then waits for a response.
2.  The server **defers its response** until an update is available or
    until a particular status or timeout has occurred.
3.  When an update is available, the server **sends a complete response**
    to the client.
4.  The client typically **sends a new long poll request**, either
    immediately upon receiving a response or after a pause to allow
    an acceptable latency period.

---

### Issues

1.  Header overhead
2.  Unnecessary maximal latency (RTT)
3.  Frequent TCP connections (persistence helps)
4.  Resource overhead at client and server
5.  Unneeded buffering during higher loads
6.  Timeouts
7.  Caching

---

## HTTP Streaming

### Flow

1.  The client makes an **initial request** and then waits for a
    response.
2.  The **server defers the response** to a poll request until an update
    is available, or until a particular status or timeout has
    occurred.
3.  Whenever an update is available, the **server sends response** back to the
    client.
4.  The data sent by the server does not terminate the request or the
    connection.  The **server returns to step 3**.

---

### Issues

1.  Network intermediaries (proxies, gateways) may buffer response.
2.  Unnecessary maximal latency (RTT)
3.  Client buffering
4.  Framing requriements not met by chunking (due to re-chunking)

---

## Server-Sent Events (SSE)

- Proposed ~2009
- W3C Recommendation ~2015
- JavaScript [EventSource API](https://www.w3.org/TR/eventsource/) part of HTML5
- Built over HTTP streaming
- Good support by modern browsers
    * Not supported by IE
    * 'Under Consideration' for Edge 16
- Uses `Content-Type: text/event-stream`

---

## Pushlets

- Old ~2002
- Publish/ subscribe mechanism
- Push JavaScript snippets from server to client using HTTP streaming
- [whitepaper](http://www.pushlets.com/doc/whitepaper.html)

---

## WebSockets

> Historically, creating web applications that need bidirectional communication between a client and a server (e.g., instant messaging and gaming applications) has required **an abuse of HTTP** to poll the server for updates while sending upstream notifications as distinct HTTP calls
>
>~ RFC6455, referring to RFC6202

(emphasis mine)

---

### Compared to Long Polling/ Streaming

#### Problems

- Server resources: multiple TCP connections per client (up/ down)
- Protocol overhead: long header per HTTP message
- Client complexity: track pair of connections and state to one server

#### Solution

Use a single TCP connection for traffic in both directions.

---

#### Advantages

* Low latency, high throughput
* Bi-directionaly communication
* Either client or server can send a message anytime
* Compatible

---

### WebSocket Protocol and API

- WebSocket protocol
    - Defined in [RFC6455](https://tools.ietf.org/html/rfc6455)
    - Status: Proposed Standard ~2011
- JavaScript [WebSocket API](https://www.w3.org/TR/websockets)
    - Part of HTML5
    - W3C Candidate Recommendation ~2012

**Pointer**

- Watch the excellent talk [Inside WebSockets](https://youtu.be/9FqjRN4VYUU) by Leah Hanson.

---

### WebSocket Highlights

- Designed to work well with existing Web infrastructure
- Start with HTTP and 'upgrade' to WebSocket
- Uses ports 80 and 443 for WS and WSS
- Can tunnel through HTTP proxies via HTTP CONNECT

---

### Basic steps

- Opening handshake
- Data exchange
- Closing handshake

---

### Supports

- Text (UTF-8) and binary data
- Messages (note: no interleaving)
- Sub-protocols (e.g. chat)
- Extensions (e.g. deflate compression)

---

### Closer Look: Demo

#### [Echo test and simple echo client](https://www.websocket.org/echo.html)
    - https://www.websocket.org/echo.html

#### Inspect WS with Firefox/ Chrome developer tools
    - Ctrl/Cmd + Shift + I
    - Network > WS

#### Firefox [WebSocket Monitor](https://addons.mozilla.org/en-US/firefox/addon/websocket-monitor) extension
    - Extend developer tools with better WS support
    - Visualize WS sessions
    - https://github.com/firebug/websocket-monitor

---

### Message Framing

```
      0                   1                   2                   3
      0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
     +-+-+-+-+-------+-+-------------+-------------------------------+
     |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
     |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
     |N|V|V|V|       |S|             |   (if payload len==126/127)   |
     | |1|2|3|       |K|             |                               |
     +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
     |     Extended payload length continued, if payload len == 127  |
     + - - - - - - - - - - - - - - - +-------------------------------+
     |                               |Masking-key, if MASK set to 1  |
     +-------------------------------+-------------------------------+
     | Masking-key (continued)       |          Payload Data         |
     +-------------------------------- - - - - - - - - - - - - - - - +
     :                     Payload Data continued ...                :
     + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
     |                     Payload Data continued ...                |
     +---------------------------------------------------------------+
```

---

### Other Notable Points

- Masking
- Ping, Pong
- Behaviour with proxies and caches
- HTTP CONNECT

---

### Two Interesting Frameworks/ Libraries

#### [Socket.IO](https://socket.io/): Realtime JavaScript framework

    - HTTP long-poll first
    - Later upgrade to WebSocket if possible
    - Handles disconnects

---

#### [SockJS](http://sockjs.org/): WebSocket emulation

    - WebSocket-like API even without WebSocket transport
    - Focus on cross-browser compatibility
    - Try native WebSocket first
    - Automatically fall back to other transports (WS > streaming > polling)
    - Multiple languages (JavaScript, Python, Java, Scala, Ruby, Go, ...)

---

# Thank You!

## Open Discussion

### My Coordinates

* Twitter: [@VipulMathur](https://twitter.com/VipulMathur)
* LinkedIn: https://in.linkedin.com/in/VipulMathur

### Links to this material

I used [GitPitch](https://gitpitch.com) to prepare slides for the meetup. Use the following links to get access to the slides:
- Slideshow: https://gitpitch.com/VipulMathur/rfc-meetup-sep2017?grs=gitlab
- Source: https://gitlab.com/VipulMathur/rfc-meetup-sep2017
