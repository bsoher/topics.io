---
sort: 2
---

# XML-RPC General Topics

## Why We Chose XMLRPC

We considered several options for getting the C++ process on the MRIR machine and the Python process (a.k.a. the listener) on the console machine to talk to one another. The text below is the core of an email in which I recommended XMLRPC over the alternatives. Philip Semanchuk

We like Python more than C++, and deploying C++ code on the MRIR machine is (I assume) a non-trivial process. Both of these facts suggest that we make the C++ code as simple as possible and do the bulk of the work on the Python side.

The C++ process will send both ASCII and binary data (headers & FIDs, and possibly not all in complete chunks) to the Python process. We have several choices of transport mechanism, presented here from highest to lowest level of abstraction. I strongly recommend the first option (XMLRPC) because it allows us to ignore details that the other two don't. 

1) XMLRPC. In this scenario, the Python process runs a simple XMLRPC server (there's one in the standard library) and the C++ process calls functions in it. (There's a BSD-licensed C++ XMLRPC library on sourceforge.) Once the initial network handshaking is resolved, XMLRPC would allow us to (mostly) pretend that the network isn't there. We define an API that the Python server offers, and the C++ process can call those functions at will. Both sender and recipient can "think" in terms of native data types (ints, strings, floats, etc.). This is the only scenario that allows us to completely ignore machine and compiler-specific details about value representation such as endianness and how many bytes a given type occupies. 

2) HTTP(S). In this scenario, the Python process runs a simple HTTP server (there's one in the standard library) and the C++ process uses HTTP POST to send data. This is easier to deal with than sockets since POSTed data arrives in discrete blocks and can include arbitrary metadata. The payload would arrive as a raw data blob (i.e. just bytes) and it would still be incumbent on the recipient to parse the metadata in order to make sense of the data payload. We could use XDR to ensure consistent value representation.

3) Sockets. There are libraries for socket communication in both C++ and Python. Sockets are very fast when managed properly. But sockets send and receive bytes -- they don't know about ints, strings, arrays, etc. Python's pickle library is often used to simplify socket-to-socket communication, but since one of our processes is non-Python, we can't use pickle. If we use sockets, we have to define and implement a communication protocol, so that the receiver knows what data is being streamed over the socket and when one chunk begins and ends.


## Hurdles

The various hurdles we encountered on our way to successfully communicating from the scanner software to the Python listener.

### The Siemens Simulator

Fortunately, Siemens provides a simulator for the scanner environment so we can develop our software without fussing around with the scanner itself. Unfortunately, it's a pretty bad simulator because it runs Windows while the scanner runs Linux (probably SUSE).

Our main gripe with this scenario is that on Windows we have to compile with MSVC 6 which is 14 years old as of this writing. This compiler doesn't understand some of the C++ constructs in the most recent versions of our XMLRPC library of choice ([XMLRPC-C](http://xmlrpc-c.sourceforge.net/)), so we had to use a rather old version of XMLRPC-C and eschew the C++ interface in favor of the somewhat clumsier C interface. Using an old version of XMLRPC-C raised another hurdle for us.

Another consequence of running under Windows is that XMLRPC-C uses WinInet as its transport mechanism whereas under Linux it uses libcurl. This obscured the buggy behavior of Python's XMLRPC server (see below) until we tested it on the scanner. 

### Python's Buggy XMLRPC Server

Python's XMLRPC server doesn't correctly handle the "Expect" header. HTTP clients can send the optional Expect header in anticipation that the server might reject the request. The server is supposed to respond immediately with a "100 Continue" status message if the request is acceptable or an error code if it's not OK. In a perfect world, this is a brief conversation and is more efficient than having the client send a request with a large body only to have it rejected by the server.

The problem that we ran into is that Python's XMLRPC server doesn't respond to the Expect header at all. libcurl politely sends the Python server an HTTP "Expect" header that says, "I'm going to send you X kilobytes of data, is that OK?" After waiting 3 seconds for Python to respond, curl gets tired of waiting and sends the data anyway. 

The practical upshot of this was that when using libcurl, all XMLRPC calls that sent anything more than a trivial amount of data took 3+ seconds to complete when they should have taken milliseconds. (Given that we expect to send data from the scanner every 1-2 seconds, using 3 seconds for each XMLRPC call is a disaster.)

[XMLRPC-C is aware of this problem](http://xmlrpc-c.sourceforge.net/doc/libxmlrpc_client.html#expectcontinue) and versions >= 1.19.0 contain a workaround. (XMLRPC-C tells libcurl not to send the Expect header at all.)

Unfortunately, since Siemens' simulator's use of MSVC 6 sent us back to using an older version of XMLRPC-C (1.16.42, one of the project's three recommended versions). We were forced to debug the problem the hard way. 

Switching to XMLRPC-C 1.19.0 was the solution. This client is robust when communicating with Python's buggy server, and is not so advanced that MSVC 6 can't compile it.

### Libcurl Does Too Much

XMLRPC-C uses libcurl as its transport mechanism on *nix. We built XMLRPC-C against libcurl.a on Linux (Ubuntu, I think) only to find out that it wants a bunch of runtime libraries that aren't present on the scanner Linux.

The solution was to build a custom libcurl with nearly every feature other than simple HTTP communication disabled.


## Efficiency and Speed of XMLRPC Communications

If you look online, there are two main suggestions for speeding up XMLRPC conversations: enabling `TCP_NODELAY` and enabling compression.

### `TCP_NODELAY`

Most (all?) TCP implementations have a builtin method for reducing network congestion. If they're asked to send a small packet, they wait for a moment to see if more data comes in because it is more efficient to send one large packet than two small ones. 

This is fine for most applications. For some, it isn't. Those apps need to enable the `TCP_NODELAY` setting. Telnet is a good example of an app that needs `TCP_NODELAY`. Telnet sends every keystroke to the server and doesn't display feedback to the user until it gets a response. It sends lots of small packets and needs to send them quickly. 

Does it make sense for XMLRPC conversations? Well, if the conversation consists of many small calls that need to happen with as little lag as possible, then `TCP_NODELAY` would make sense. In our case, we're doing the opposite. We're making a few large requests instead of many small ones. *We should leave the `TCP_NODELAY` flag at its default setting.*

### Compression

Most (all?) modern Web servers have the ability to send compressed content to clients. The compression is almost always zlib (the open source algorithm for which gzip is a common front end). If the client supports it (and most do), the server compresses the content (usually a text Web page) before sending it resulting in a fewer number of TCP packets that need to make their way to the client.

It's not so easy to take advantage of compression in our use of XMLRPC. In our case, we are sending lots of data from the client to the server which is the opposite of the common Web server case. There's been far less work put into making that easy to do, and as far as I can tell we can't make it happen without modifying XMLRPC-C.


