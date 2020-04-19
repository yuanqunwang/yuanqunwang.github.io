---
title: TCP Connection Establishment and Termination
tags: [TCP]
---
# 18 TCP Connection Establishment and Termination

## Connection Establishment
**three-way handshake**

1. The requesting end(normally called the client) sends a SYN segment specifying the port number of the server that the client wants to connect to, and the client's initial sequence number(ISN).

2. The server responds with its own SYN segment containing the server's ISN.The server also acknowledges the client's SYN by ACKing the client's ISN plus one.

3. The client must acknowledge this SYN from the server by ACKing the server's ISN plus one.
***
## Reset Segments

* In general, a reset is sent by TCP whenever a segment arrives that doesn't appear correct for the referenced connection(src&dest IP and port). A common case for generating a reset is when a connection request arrives and no process is listening on the destination port.
* **orderly release:** one side send a FIN.
* **abortive release:** one side send a reset instead of a FIN.
* **half close** One end of a connection to terminate its output by sending an FIN to other end, while still receiving data from the other end.
* **half open** A TCP connection is said to be half-open if one end has closed or aborted the connection without the knowledge of the other end.

## TCP Server Design
* Most TCP servers are concurrent, when a new connection request arrives at a server, the server accepts the connection and invokes a new process to handle the new client.
* `netstat` command to list Local Address, Foreign Address and (state) info.By using `sock` command to specify the Local or Foreign Address to restrict corresponding IP address.
