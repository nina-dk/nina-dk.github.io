---
layout: post
title: The Perilous Journey of an HTTP Request
date: 2021-09-17 21:34:42
categories:
- Networking
tags: 
- HTTP
- TCP
- Ethernet
- TLS
---

This is the story of a lonely HTTP request sailing across the vast Internet seas.

{% img https://images.unsplash.com/photo-1534447677768-be436bb09401?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1194&q=80 '"Boat sailing in the ocean" "Photo by Johannes Plenio on Unsplash"' %}


It all started when a tired pair of human eyes searched for some peace in cute cat images.
<!-- more -->

{% asset_img search-url.jpg "'Google Search for “cute cat images”''Google Search for “cute cat images”'" %}

This was the birth of an *HTTP request*: a text message comprised of some necessary information structured in the way that the **Hypertext Transfer Protocol (HTTP)** commands. Place of birth: the *Application Layer*.

It looked something like this:

{% asset_img curl-request-cut.png "'HTTP Request in curl''The HTTP Request'" %}

Line 1 is the *request line*.

```
GET /search?q=cute%20cat%20images HTTP/2
```

It includes the *HTTP Method* that will tell the destination host, when it gets there, how it should handle it. This is a `GET` request: it asks the server to get a resource (the search results page for “cute cat images”) and send them back to the *client*: our browser.

It also includes the *request-URI*, aka the *path*, that will let the server know which resource it needs to fetch. Our request’s path includes a query string, `?q=cute%20cat%20images`, which passes some additional information to the server regarding the requested resource.

Short pause here to discuss the query string — what is this `q` and why are there some weird characters separating the words we typed in the search box?

`q` is the name of the input field we filled. We can see that here:

{% asset_img q-input-field.jpg "'Google search field name attribute is q''Notice the `name` attribute of the input element'" %}

The “weird” characters, specifically `%20`, are there because we cannot have spaces in URLs. Therefore we use the [ASCII hex code](https://www.rapidtables.com/code/text/ascii-table.html) for a space (the number `20`) preceded by `%`, which is simply a reserved character for entering ASCII codes in the URL to encode characters that are not allowed or are not safe to use.

Close parenthesis.

Finally, the HTTP version in use is specified: `HTTP/2`.

Below the request line is a necessary header that identifies the receiving host, `www.google.com`. We will not get into the following two headers.

Are we all set to travel? By no means. The HTTP request has to undergo a transition before it is ready to embark on its dangerous journey.

## Becoming a TCP segment

It now enters the *Transport Layer*. The commander of this layer is usually the **Transmission Control Protocol (TCP)**. TCP takes its job very seriously and wants to make sure that every segment sent is received, and that in the right order.

Let’s take a break here to briefly discuss how TCP implements these features to ensure a reliable connection between client and server is established.

Firstly, it follows a process known as the *TCP three-way handshake*. During the “handshake”, the client and server exchange several messages.
1. The client-host sends a segment with the `SYN` flag set.
2. The server responds with a segment with `SYN` — `ACK` flags.
3. The client sends the final segment with an `ACK` flag.

Once the handshake is complete, the HTTP request can be safely transmitted.

Additionally, in order to make sure that the request actually got to its destination, an *acknowledgement* message is sent back from the server to our computer’s network interface. If no acknowledgement is received, the segment is retransmitted.

Finally, if a message is too large to be sent in one go, it is broken down into several segments that each have their own *sequence number* (situated in a header field, as well). This way, when they arrive at their destination, they will be processed in the correct order.

The HTTP request is encapsulated in a TCP segment. That segment has a header and a data payload (where the HTTP request now lies).

{% asset_img tcp-segment2.png "'A TCP segment encapsulating an HTTP request''The TCP Segment (outlined in black) encapsulates the HTTP Request'" %}

So what’s in the header?

The Transport Layer is responsible for sending the segment from the requesting application (the web browser, in this case) to the correct process in the Google server (where HTTP requests are handled). It does that through the use of *ports*: integers in the range of 0–65535 that act as identifiers for a specific application or process in a host.

The header of our TCP segment will contain (among other information) the *source port* assigned to that application by our OS and the *destination port*. The default port for HTTP is 80.

That way, when the segment arrives at Google’s server it will be directed to port 80, where the server listens for HTTP requests.

But how will it get to Google’s server in the first place?

## Becoming an IP packet

Onwards, downwards, to the *Internet Layer*.

This is where TCP segments are turned into *IP packets* by the **Internet Protocol (IP)**.

The Internet Protocol is the one that makes it possible for a packet to travel from your home to a server on the other side of the world.

{% asset_img ip-packet.png "'An IP packet encapsulating a TCP segment''The IP Packet (outlined in blue) encapsulates the TCP Segment'" %}

The TCP segment willingly jumps into the packet. IP seals it and writes the return and mailing addresses on it. The sender’s address or *source IP address* is the IP address of my computer (I’m the one in need of cute cat pictures). The recipient or *destination IP address* is the IP address of Google’s server: the unique sequence of numbers that identifies that computer on the internet.

An IP address might look something like this: `207.126.144.100`, for IPv4, or this: `2a02:687:1211:7500:c8cb:bcf9:7acb:a966`, for IPv6.

(*Note:* IPv4 and IPv6 are versions of the IP protocol, with IPv6 being the latest version that can accommodate approximately 10²⁷ times more IP addresses than IPv4 does).

All set; on to the next, final layer.

## Becoming an Ethernet frame

Our request has officially reached its final preparation stage: the *Data Link Layer*. The **Ethernet** protocol is in charge of this layer.

Ethernet wraps the IP packet, containing the TCP segment, containing the HTTP Request, into a *frame*. On the header of the frame are the *source* and *destination MAC addresses*. Except, the destination, in this case, is not yet Google’s server, located miles and miles away, but the next *node* in line. And by node, I’m referring to the closest networked device that the message will be transferred to first — the next “hop” on the way to Google. Right now, it will probably be my router.

{% asset_img ethernet-frame.png "'An Ethernet frame encapsulating an IP packet''The Ethernet Frame (outlined in green) encapsulates the IP Packet'" %}

This is what Ethernet does: it’s responsible for getting the frame to the next “hop” in the network. Once it gets there, the destination MAC address will change to point to the next node. It’s much like how real packages travel across the world: each courier (node) processes the package (frame) and dispatches it to the next courier until it reaches its final destination.

{% img https://images.unsplash.com/photo-1508416163602-e4eb39645e86?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=2074&q=80 "'Light signals''Photo by Daniel Falcao on Unsplash'" %}

What now? Chaos, cables and wires. The frame takes its final form: it becomes a stream of bits travelling as electrical signals (or light signals or radio waves) towards the destination MAC address — the router that will act as a gateway to the outside world.

In the same manner, an *HTTP Response* will be prepared at Google’s servers and travel all the way back to my home, my computer, and ultimately, from my browser to my tired eyes.

{% asset_img cute-cat-images.jpg "'Search results for cute cat images''The HTTP Response message interpreted as a web page'" %}

#### Notes

1. For the purposes of this article, the [TCP/IP model](https://en.wikipedia.org/wiki/Internet_protocol_suite) is used.
2. There is more than one protocol that can be used at each layer. For example, at the Transport Layer, **UDP** might be used by some applications in the place of TCP.
3. In the case of HTTPS, which is the one actually used by google.com, the HTTP Request is first encrypted by the **TLS** protocol, so that it can be safely transmitted.
4. Unfortunately, there’s a *lot* more going on than what is mentioned in this article.