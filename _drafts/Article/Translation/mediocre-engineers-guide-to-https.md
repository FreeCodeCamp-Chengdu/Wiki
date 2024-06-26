---
title: Mediocre Engineer’s guide to HTTPS
date: 2024-05-28T04:38:43.572Z
authorURL: ""
originalURL: https://devonperoutky.super.site/blog-posts/mediocre-engineers-guide-to-https
translator: ""
reviewer: ""
---

![image](https://files.oaiusercontent.com/file-SyCerofEEDgemcHfPpquHWxP?se=2024-05-26T17%3A15%3A21Z&sp=r&sv=2023-11-03&sr=b&rscc=max-age%3D31536000%2C%20immutable&rscd=attachment%3B%20filename%3D53c7983c-c33e-4b28-80ff-ad5cdd387868.webp&sig=MN94zDnTPF%2BB0jZFNe434yjt40e6dIcfT%2BQiedrSLBg%3D)

<!-- more -->

-   [
    
    Lifecycle of a HTTP request
    
    ][1]
-   [
    
    1\. Sender Makes a Request
    
    ][2]
-   [
    
    2\. DNS Lookup:
    
    ][3]
-   [
    
    3\. TCP Handshake:
    
    ][4]
-   [
    
    4\. Transmit HTTP Request
    
    ][5]
-   [
    
    5\. Packets routed across Internet to Server
    
    ][6]
-   [
    
    Step-by-step explanation of how text makes it across the internet
    
    ][7]
-   [
    
    6\. Server Response
    
    ][8]
-   [
    
    7\. Content Rendering:
    
    ][9]
-   [
    
    Little Layer Review
    
    ][10]
-   [
    
    HTTPS = HTTP + Encryption
    
    ][11]
-   [
    
    TLS Handshake
    
    ][12]
-   [
    
    TLS Handshake
    
    ][13]
-   [
    
    Everything you’ve learned here is a lie.
    
    ][14]
-   [
    
    What is different about a handshake in TLS 1.3?
    
    ][15]
-   [
    
    Shameful Plug
    
    ][16]

As a mediocre engineer, I took Internet and HTTPS communication for granted and never dove any deeper. Today we’re improving as engineers and learning a rough overview of how internet communication works, specifically focusing on HTTP and TLS.

The Internet is “just” a network of interconnected computer networks. The term "Internet" literally means "between networks." It operates as a packet-switched [mesh network][17] with best-effort delivery, meaning there are no guarantees on whether a packet will be delivered or how long it will take. The reason why the internet appears to operate so smoothly (at least from a technical perspective) is the layers of abstraction that handle retries, ordering, deduplication, security and so many other things behind the scenes. Letting us developers just focus on the application layer (aka. Writing HTTP requests from San Francisco for $300K/year).

Each layer provides certain functionalities, which can be fulfilled by different [protocols][18]. Such modularization makes it possible to replace the protocol on one layer without affecting the protocols on the other layers.

Here’s a simple table of the layers.

<table class="notion-table col-header"><tbody><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px;background:var(--color-color-default)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Name</strong></span></div></td><td style="min-width:250px;max-width:250px;background:var(--color-color-default)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Description</strong></span></div></td><td style="min-width:143px;max-width:143px;background:var(--color-color-default)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Unit of Communication</strong></span></div></td><td style="min-width:162px;max-width:162px;background:var(--color-color-default)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Unique Identifier</strong></span></div></td><td style="min-width:125px;max-width:125px;background:var(--color-color-default)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Example</strong></span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px"><div class="notion-table__cell"><span class="notion-semantic-string">Application layer</span></div></td><td style="min-width:250px;max-width:250px"><div class="notion-table__cell"><span class="notion-semantic-string">Manages application-specific logic</span></div></td><td style="min-width:143px;max-width:143px"><div class="notion-table__cell"><span class="notion-semantic-string">Messages</span></div></td><td style="min-width:162px;max-width:162px"><div class="notion-table__cell"><span class="notion-semantic-string">Application-specific</span></div></td><td style="min-width:125px;max-width:125px"><div class="notion-table__cell"><span class="notion-semantic-string">HTTP</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px"><div class="notion-table__cell"><span class="notion-semantic-string">Security layer</span></div></td><td style="min-width:250px;max-width:250px"><div class="notion-table__cell"><span class="notion-semantic-string">Provides encryption and authentication</span></div></td><td style="min-width:143px;max-width:143px"><div class="notion-table__cell"><span class="notion-semantic-string">Records</span></div></td><td style="min-width:162px;max-width:162px"><div class="notion-table__cell"><span class="notion-semantic-string">Public Key Certificate</span></div></td><td style="min-width:125px;max-width:125px"><div class="notion-table__cell"><span class="notion-semantic-string">TLS</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px"><div class="notion-table__cell"><span class="notion-semantic-string">Transport layer</span></div></td><td style="min-width:250px;max-width:250px"><div class="notion-table__cell"><span class="notion-semantic-string">Ensures reliable data transfer</span></div></td><td style="min-width:143px;max-width:143px"><div class="notion-table__cell"><span class="notion-semantic-string">Segments (TCP) / Datagrams (UDP)</span></div></td><td style="min-width:162px;max-width:162px"><div class="notion-table__cell"><span class="notion-semantic-string">Port number</span></div></td><td style="min-width:125px;max-width:125px"><div class="notion-table__cell"><span class="notion-semantic-string">TCP</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px"><div class="notion-table__cell"><span class="notion-semantic-string">Network layer</span></div></td><td style="min-width:250px;max-width:250px"><div class="notion-table__cell"><span class="notion-semantic-string">Routes packets across the Internet</span></div></td><td style="min-width:143px;max-width:143px"><div class="notion-table__cell"><span class="notion-semantic-string">Packets</span></div></td><td style="min-width:162px;max-width:162px"><div class="notion-table__cell"><span class="notion-semantic-string">IP address</span></div></td><td style="min-width:125px;max-width:125px"><div class="notion-table__cell"><span class="notion-semantic-string">IP</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px"><div class="notion-table__cell"><span class="notion-semantic-string">Link layer</span></div></td><td style="min-width:250px;max-width:250px"><div class="notion-table__cell"><span class="notion-semantic-string">Manages the physical medium</span></div></td><td style="min-width:143px;max-width:143px"><div class="notion-table__cell"><span class="notion-semantic-string">Frames</span></div></td><td style="min-width:162px;max-width:162px"><div class="notion-table__cell"><span class="notion-semantic-string">MAC address</span></div></td><td style="min-width:125px;max-width:125px"><div class="notion-table__cell"><span class="notion-semantic-string">Wi-Fi</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:138px;max-width:138px"><div class="notion-table__cell"><span class="notion-semantic-string">Physical Layer</span></div></td><td style="min-width:250px;max-width:250px"><div class="notion-table__cell"><span class="notion-semantic-string">Physically transmit raw bits from one device to another</span></div></td><td style="min-width:143px;max-width:143px"><div class="notion-table__cell"><span class="notion-semantic-string">Bits</span></div></td><td style="min-width:162px;max-width:162px"><div class="notion-table__cell"><span class="notion-semantic-string">N/A</span></div></td><td style="min-width:125px;max-width:125px"><div class="notion-table__cell"><span class="notion-semantic-string">Fiber optic, Ethernet cables</span></div></td></tr></tbody></table>

We’ll go over these layers more in-depth layer, but first, let’s see this in action.

# Lifecycle of a HTTP request

Here is the path of an HTTP request through these layers (Skipping physical layer for brevity).

![image](https://images.spr.so/cdn-cgi/imagedelivery/j42No7y-dcokJuNgXeA0ig/ca58d5c9-2241-4a54-b3f3-9ec8faf474d3/HTTP_Request/w=3840,quality=90,fit=scale-down)

## **1\. Sender Makes a Request**

The process begins at the Application layer, where the client (usually a web browser) constructs an HTTP request. HTTP is a text-based protocol, meaning that all this data is sent as plain text over the wire.

The first line typically includes:

-   **HTTP method** (GET, POST, etc)
-   **Requested Resource** (Example: `/index.html` )
-   **Protocol version.**

The remainder of the HTTP message contains headers in a `key: value` format an an optional message body.

**Example: HTTP Request**

Copy

```
GET /index.html HTTP/1.1
Host: www.example.com
Accept: text/html
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36
```

## **2\. DNS Lookup**:

The Domain Name System (DNS) translates the human-readable domain name (`**www.example.com**`) into an IP address (e.g., `**93.184.216.34**`). The client queries DNS servers to resolve the domain name to its corresponding IP address. This process goes through multiple resolvers until it reaches the authoritative server which does the conversion of domain name to IP address. At a very high level, the three components are

-   **Stub resolvers**, which lives on the client machine and routes the request to the appropriate recursive resolver (explained next)
-   **Recursive resolvers**, which receives requests from the stub resolver and queries authoritative servers to resolve the domain name - often caching the result. Your Internet Service Provider (ISP) typically provides a recursive resolver, or you may use a public one like Google DNS (8.8.8.8).
-   **Authoritative servers** which contain the actual DNS records (like A, MX, CNAME, etc.) for a domain and responds to queries with the information in those records. Authoritative servers are the final source of truth for domain name data.

When a client issues a request for a resource using a domain name, the **stub resolver** on your computer sends a query to a recursive resolver to resolve the domain name.

The recursive resolver, queries authoritative DNS servers as needed to resolve the domain name to an IP address.

## **3\. TCP Handshake**:

Now that we have the IP address of the server, the client can begin transmitting the HTTP and we move to the Transport Layer. There are two primary protocols for the transport layer, **TCP (Transmission Control Protocol) and UDP (User Datagram Protocol).**

💡

TCP is a connection-oriented protocol that ensures reliable, ordered, and error-checked data delivery between applications.

UDP is a connectionless protocol that provides fast, low-overhead data transmission without guaranteeing delivery, order, or error checking.

As of 2024, TCP is the main protocol for managing data transport across the internet, while UDP is less commonly used, typically for real-time applications like streaming or video calls, where low latency is crucial and occasional packet loss is acceptable. Now back to the topic at all.

Once the client has obtained a the IP address, it initiates a TCP connection with the server on port 80 (the standard port for HTTP). This involves a three-step handshake:

-   **SYN**: The client sends a SYN (synchronize) packet to the server to request a connection.
-   **SYN-ACK**: The server responds with a SYN-ACK (synchronize-acknowledge) packet to acknowledge the request.
-   **ACK**: The client sends an ACK (acknowledge) packet back to the server, establishing a reliable connection.

## 4\. Transmit HTTP Request

With the TCP connection in place, the client sends the actual HTTP request. As mentioned, HTTP is a text-based protocol, so the request headers and the body (if any) are sent as plain text.

## 5\. Packets routed across Internet to Server

**⚠️⚠️⚠️⚠️⚠️ We’re going deep here ⚠️⚠️⚠️⚠️⚠️**

When a client sends a request, the data packets don't travel directly to the server. Instead, they follow a path through various network devices, primarily routers, which determine the best route for the packets to reach the server network gateway. From there, the link layer comes into play.

### Step-by-step explanation of how text makes it across the internet

1.  **Initial Transmission**:

The client's device encapsulates the HTTP request data into TCP segments and then into IP packets. These packets are further encapsulated into smaller chunks, referred to as frames, suitable for the Link Layer (e.g., Ethernet frames if using a wired connection).

3.  **Local Network**:

The frames are transmitted over the local network to the client's router. The Link Layer handles the communication within this local network, ensuring the frames reach the router.

5.  **Local Router Processing**:

The router receives the frames, strips off the Link Layer headers, and processes the IP packets. The router examines the destination IP address in the packets and determines the next hop on the path to the server.

7.  **Routing Across Networks**:

The router forwards the packets to the next network, often through one or more intermediary routers. Each intermediary router repeats the process: receiving the packets, determining thenext hop, and forwarding them.

9.  **Final Network**

Eventually, the packets reach a router on the same network as the destination server. This router performs the final routing decision and sends the packets to the appropriate local device (the server).

11.  **Server Reception**:

The server's router forwards the packets over the local network segment to the server. The Link Layer ensures the frames are correctly transmitted to the server's network interface. (It has been doing that for every machine → machine communication for this whole time.

13.  **Server Processing**:

The server receives the frames, extracts the IP packets, and processes the encapsulated TCP segments to reconstruct the original HTTP request. The server then generates an HTTP response and the process reverses to send the response back to the client.

⁉️

The process of sending packets across the internet (The Network Layer) is used for essentially all communication over the internet. So it was used for all the steps earlier (like resolving the domain name, the TCP handshake, etc) however there’s only so much that can be explained at once.

### 6\. Server Response

The server receives the HTTP request and processes it. After processing the request, the server sends an HTTP response back to the client. The response includes:

-   **Protocol** (The HTTP version being used)
-   **Status information** (The HTML Status code like 200, 404, etc)
-   **Response headers** (Like Request Header but Response)
-   **Requested content/Body** (The actual content, such as HTML of the request page or JSON data)

Copy

```
HTTP/1.1 200 OK
Date: Sat, 26 May 2023 10:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html
Content-Length: 3456

<!DOCTYPE html>
<html>
<head>
    <title>Example Page</title>
</head>
<body>
    <h1>Hello, world!</h1>
</body>
</html>
```

You may have seen something like this when debugging requests.

![image](https://images.spr.so/cdn-cgi/imagedelivery/j42No7y-dcokJuNgXeA0ig/cfcdb0b7-d20d-4a0c-a50f-720d59a0fe82/Screenshot_2024-05-26_at_12.22.31_AM/w=3840,quality=90,fit=scale-down)

### **7\. Content Rendering**:

The client receives the HTTP response and processes it. The browser interprets the HTML and renders the content on the screen. If the response includes additional resources (e.g., images, CSS, JavaScript), the browser will make further HTTP requests to fetch these resources, following the same process.

So now that we’ve gotten a basic HTTP request out of the way, there’s only one problem. **It’s not secure at all.** Anyone listening on the connection can view 100% of the data being passed back-and-forth. Additionally, someone could pretend to be a server such that the client is tricked into sending valuable information. That’s where the **Security Layer** comes into play

## Little Layer Review

While we’re here, let’s do a brief review of the layers and their purpose, while we introduce the Security Layer.

-   **Application Layer**: Where applications create and communicate user data. This is what you have interacted the most with. Uses transport layer services for reliable or unreliable data transmission. Protocols include HTTP, FTP, SSH, SMTP. Uses ports to address processes/services.
-   **Security Layer**: Ensures secure communication by providing encryption, authentication, and data integrity. Common protocols include TLS (Transport Layer Security) and its predecessor SSL (Secure Sockets Layer). This layer protects data in transit and verifies the identity of the communicating parties.
-   **Transport Layer**: Manages host-to-host communications, providing channels for application data. Includes:

-   **UDP**: Unreliable, connectionless datagram service.
-   **TCP**: Reliable, connection-oriented service with flow control and connection establishment.

-   **Network Layer**: Responsible for exchanging packets across network boundaries via routing the packets through various intermediate routers. Primary protocol: Internet Protocol (IP).
-   **Link Layer**: Manages local network communications without routers. Defines local network topology and interfaces for transmitting datagrams to neighboring hosts.

Specifically pay attention to the **Security Layer**, as that layer is the defining difference between an HTTP request (which we just covered) and an HTTPS request ([~86% of the current internet][19] and growing).

# HTTPS = HTTP + Encryption

**HTTPS is HTTP with encryption and verification**. While there are multiple ways of securing HTTP communication over the internet, the current implementation everyone uses is Transport Layer Security (TLS)**.**

TLS is how the client and server can verify each other identities and ensure all the payloads are encrypted in a way both parties will be able to decrypt them. The **TLS handshake process**, specifically, determines how the client and server will exchange encryption and verification keys. Once the keys have been exchanged, the client and server will communicate using HTTP as normal, and use the keys to encrypt and verify messages.

The flow of an HTTPS is the exact same as the HTTP request we covered previously, with the addition of a Security Layer in between the Application Layer and the Transport Layer (although typically TCP is used for the TLS handshake).

![image](https://images.spr.so/cdn-cgi/imagedelivery/j42No7y-dcokJuNgXeA0ig/6532134b-285a-4eb6-b7cc-f0c40ee67c91/HTTPS_Request/w=3840,quality=90,fit=scale-down)

## TLS Handshake

The TLS handshake is for the client and server to agree on a few different aspects of the communication. Specifically, the collection of algorithms that will be used for verifying, compressing, and encrypting messages.

<table class="notion-table col-header"><tbody><tr style="color:var(--color-text-default)"><td style="min-width:120px;max-width:240px;background:var(--color-undefined)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Component</strong></span></div></td><td style="min-width:120px;max-width:240px;background:var(--color-undefined)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Description/Purpose</strong></span></div></td><td style="min-width:120px;max-width:240px;background:var(--color-undefined)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Common Implementations</strong></span></div></td><td style="min-width:120px;max-width:240px;background:var(--color-undefined)"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Primarily Currently Used</strong></span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Compression Algorithm</strong></span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">How they client and server will compress data over the wire</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">Gzip, Brotli</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">Brotli</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Key Exchange Algorithm</strong></span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">Securely exchange cryptographic keys over a public channel</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">ECDHE-RSA, ECDHE-ECDSA</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">ECDHE (provides perfect forward secrecy)</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Authentication Algorithm</strong></span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">Authenticate the identity of the parties during the handshake</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">RSA, ECDSA</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">RSA (widely used), ECDSA (gaining popularity)</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>Symmetric Encryption Algorithm</strong></span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">Encrypts the data transmitted between the client and server</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">AES-128-GCM, AES-256-GCM</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">AES-GCM (provides strong security and efficiency)</span></div></td></tr><tr style="color:var(--color-text-default)"><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string"><strong>MAC Algorithm</strong></span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">Ensures the integrity and authenticity of the messages</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">HMAC-SHA256, HMAC-SHA384</span></div></td><td style="min-width:120px;max-width:240px"><div class="notion-table__cell"><span class="notion-semantic-string">HMAC-SHA256 (common), GCM modes in modern cipher suites</span></div></td></tr></tbody></table>

🔒

This collection of algorithms are referred to as **cipher suites.** To be specific all of them except the compression algorithm are considered the cipher suite, but for brevity I’ll refer to the full collection of them the cipher suite going forward.

By agreeing on all these algorithms, exchanging random seeds, and the server’s SSL certificate containing the private key; the client and server can generate a symmetric key that will be used to encrypt and verify the messages being passed back and forth. This process of agreeing on cipher suites and distributing the necessary information (seeds and SSL cert) is referred to as the TLS handshake.

![source: ](https://images.spr.so/cdn-cgi/imagedelivery/j42No7y-dcokJuNgXeA0ig/d6a4d9a3-2c45-4d5b-825c-fa99b8e555e7/Untitled/w=3840,quality=90,fit=scale-down)

source: [Cloudflare][20]

**Note:** All communication happens over TCP, the blue steps indicate the TCP handshake and the yellow steps are TLS handshake .

### TLS Handshake

1.  **Client Hello**

1.  The client will send a “Client Hello”, which is an TCP message to the server specifying the **cipher suites** it supports, as well as the supported TLS version and a random number (called the Client Random)

3.  **Server Hello**

1.  The server will respond with a “Server Hello” which is a TCP message containing the chosen TLS version, the chosen cipher suite algorithms, and it’s own random number (the Server Random)

5.  **Certificate Verification**

1.  The client verifies the server’s SSL certificate with the Certificate Authority and retrieves the server’s public key.

7.  **Premaster Secret Generation**

1.  The client generates a premaster secret, encrypts it with the server’s public key, and sends it to the server.

9.  **Decryption**

1.  The server decrypts the premaster secret using its private key.

11.  **Session Key Creation**

1.  Both client and server use the client random, server random, and premaster secret to create session keys.

13.  **Client Ready**

1.  The client sends a "finished" message encrypted with a session key.

15.  **Server Ready**

1.  The server sends a "finished" message encrypted with a session key.

17.  **Secure HTTP Communication**

1.  The session keys are used for secure symmetric encryption, ensuring both parties can now communicate securely.

Boom. That’s the TLS handshake, except for one more thing, and that is….

## **Everything you’ve learned here is a lie.**

The process we just describe is for the original version of TLS, which is outdated compared to the more modern version of TLS 1.3.

# **What is different about a handshake in TLS 1.3?**

The process we just went through is a little outdated, but it’s a great place to start due to it introducing the necessary concepts of what needs to be agreed upon for secure server <> client communication.

Current version of TLS (>1.3) do not support RSA (and various other cipher suites) for security reasons. The newer versions are more opinionated, allow significantly fewer options, which makes them simpler, more secure, and faster. However, the components and concepts are all very much the same. You still have an TLS handshake process that agrees on the compression method, the server-authentication, and key exchange in the pursuit of generating a symmetric encryption key for securing the data of the packets being exchanged via TCP.

TLS 1.3 does not support RSA, nor other cipher suites and parameters that are vulnerable to attack. It also shortens the TLS handshake, making a TLS 1.3 handshake both faster and more secure.

The basic steps of a TLS 1.3 handshake are:

-   **Client hello:** The client sends a client hello message with the protocol version, the client random, and a list of cipher suites. Because support for insecure cipher suites has been removed from TLS 1.3, the number of possible cipher suites is vastly reduced. The client hello also includes the parameters that will be used for calculating the premaster secret. Essentially, the client is assuming that it knows the server’s preferred key exchange method (which, due to the simplified list of cipher suites, it probably does). This cuts down the overall length of the handshake — one of the important differences between TLS 1.3 handshakes and TLS 1.0, 1.1, and 1.2 handshakes.
-   **Server generates master secret:** At this point, the server has received the client random and the client's parameters and cipher suites. It already has the server random, since it can generate that on its own. Therefore, the server can create the master secret.
-   **Server hello and "Finished":** The server hello includes the server’s certificate, digital signature, server random, and chosen cipher suite. Because it already has the master secret, it also sends a "Finished" message.
-   **Final steps and client "Finished":** Client verifies signature and certificate, generates master secret, and sends "Finished" message.
-   **Secure symmetric encryption achieved**

There you go. Go out and ace your technical interviews now.

# Shameful Plug

If you want to read more posts like these, you can subscribe.

<iframe src="https://chilipepper.io/form/xhot-darkbrown-jalapenos-25c28489-cf1f-4231-a35e-4e84d9bf24e5" title="chilipepper.io" sandbox="allow-scripts allow-popups allow-forms allow-same-origin allow-top-navigation-by-user-activation " allowfullscreen="" loading="lazy" frameborder="0"></iframe>

In addition to writing mediocre [technical blog posts][21], I also offer consultancy services and run a [development agency][22]. I have [built a lot of things][23], including

…an RAG AI chatbot and search tool for corporate knowledge bases - acquired by Brex

…distributed Python and Scala services at Twilio and Valon

…award-winning Military Recall App chosen by SAIC for the US Department of Defense

I’ve also helped lead teams at some of these elite startups. If you are looking for software development services or consultation for a project, I might be able to help. Feel free to reach out at [devonperoutky@gmail.com][24].

[1]: #block-48f58523ccfd407ea9e5bfc91b3e3387
[2]: #block-22a8490d95884f838d770d5dca6c2e27
[3]: #block-63b5bd78e0cd4a5c91f14115b45f3c50
[4]: #block-f968956e32b84f5aaaed3368df807c89
[5]: #block-5614ab20bd1f445c980ec96553f97ce6
[6]: #block-145c998254ff4d8c9f1554b68a2849e4
[7]: #block-e908d8f6125247daad89e708ffc8ee50
[8]: #block-0ab8d0947866452b9713493cfb3519f5
[9]: #block-ca9b1955aebd4ed59067e3b73e134d79
[10]: #block-5b6b6b8f5d5641d8882d359d116c590b
[11]: #block-9ae044f73fbd4f999ce8add94e367939
[12]: #block-1fa6c45c12cc415cb427356fbfc72218
[13]: #block-7fc77a5a056141e8aa638adda7e798ce
[14]: #block-c7300fc061d5483daaa88154bb2cea66
[15]: #block-b2d72c0ae8ce49a286ca653d39478c83
[16]: #block-a4074de1cd2343cd893d699766bd6194
[17]: https://en.wikipedia.org/wiki/Mesh_networking
[18]: https://en.wikipedia.org/wiki/Communication_protocol
[19]: /blog-posts/intro-to-llms-for-engineers-1
[20]: https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/
[21]: /
[22]: http://jedsoftware.com
[23]: /
[24]: mailto:devonperoutky@gmail.com