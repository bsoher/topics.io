---
sort: 1
---

# Building a Working Client
This document describes how to build an XMLRPC client that talks to a 
Python server on another machine using [XMLRPC-C](http://xmlrpc-c.sourceforge.net/)
as the XMLRPC library and [libcurl](http://curl.haxx.se/libcurl/) as the 
transport mechanism.

The scanner runs 64-bit Linux. I did this build under 64-bit Ubuntu 8.04. 
I deliberately chose an older Linux under the theory that the resulting 
library will depend on the glibc runtime, and that an older glibc is more
likely to be on the scanner's Linux than a newer one.


## Building libcurl
Since the client runs in a Linux environment that we don't know much about
and don't control, we can't assume too much about what libraries are present
at runtime. In its default build config, libcurl is capable of a ton of stuff
(IPv6, HTTPS, telnet, FTP, POP, LDAP, etc.). In many cases it relies on other 
runtime libraries to provide these services.

To limit runtime dependencies, I built libcurl with every feature I could find
documented disabled. Specifically --

1. Download the latest libcurl & unpack. Mine was in `/usr/local/src/curl-7.26.0`.
1. Run `./configure` and `make`, but use the configure command below. 
1. After `make` is complete, you need `libcurl.a`. You can fish it out of
 the build directory. On my build it was in 
 /usr/local/src/curl-7.26.0/lib/.libs/libcurl.a. You can find it with this command --
    ```
    find /usr/local/src/curl-7.26.0 -name libcurl.a
    ```
1. Optionally, run `sudo make install`. Bear in mind that this will become the
 default `libcurl` on your system.

Here's the configure command to use --

```
./configure --disable-shared --disable-thread \
            --disable-ftp    --disable-tftp \
            --disable-file   --disable-rtsp \
            --disable-ldap  --disable-proxy \
            --disable-dict --disable-telnet \
            --disable-threaded-resolver \
            --disable-pop3 --disable-imap \
            --disable-smtp --disable-gopher \
            --disable-manual --disable-ipv6 \
            --disable-sspi --disable-crypto-auth \
            --disable-ntlm-wb --disable-tls-srp \
            --disable-cookies --without-ssl \
            --without-gnutls  --without-polarssl \
            --without-cyassl --without-nss \
            --without-axtls \
            --without-ca-bundle --without-ca-path  \
            --without-libssh2 --without-librtmp \
            --without-libidn \
            --without-zlib  
```


## Building XMLRPC-C
I built libxmlrpc-c 1.16.42 a.k.a. super stable with the typical commands -- 

```
./configure
make
sudo make install
```

Nothing fancy required, and this should be true for other XMLRPC-C versions, too.

## Putting It All Together: Building the Client
Attached is a .zip file that contains a very simple demo client. It reads two
data files (included) and sends them to the server via XMLRPC-C.

On my Ubuntu 8.04 x64 system, I built the client using this command -- 

```
gcc -I/usr/local/include/xmlrpc-c -Wall \
    -static \
    -L/usr/lib/x86_64-linux-gnu/ \
    -L/usr/local/lib/ \
    test_xmlrpc-c.c \
    -lxmlrpc_client \
    -lxmlrpc \
    -lxmlrpc_util \
    -lxmlrpc_xmlparse \
    -lxmlrpc_xmltok \
    -lcurl \
    -lrt \
    -lpthread \
    -o client 
```

You can also use something very inflexible like this if you want to be 110% 
sure that gcc uses specific static libraries --

```
gcc -I/usr/local/include/xmlrpc-c -Wall \
    -static \
    test_xmlrpc-c.c \
    /usr/local/lib/libxmlrpc_client.a \
    /usr/local/lib/libxmlrpc.a \
    /usr/local/lib/libxmlrpc_util.a \
    /usr/local/lib/libxmlrpc_xmlparse.a \
    /usr/local/lib/libxmlrpc_xmltok.a \
    /usr/local/lib/libcurl.a \
    /usr/lib/x86_64-linux-gnu/librt.a \
    /usr/lib/x86_64-linux-gnu/libpthread.a \
    -o client 
```


Remember, [the ldd command](http://linux.die.net/man/1/ldd) is your friend!

## Caveat
There is one caveat to this process. When I build, I get this warning:
```
/usr/local/lib/libcurl.a(libcurl_la-netrc.o): In function `Curl_parsenetrc':
netrc.c:(.text+0x30f): warning: Using 'getpwuid' in statically linked applications requires at runtime the shared libraries from the glibc version used for linking
/usr/local/lib/libcurl.a(libcurl_la-curl_addrinfo.o): In function `Curl_getaddrinfo_ex':
curl_addrinfo.c:(.text+0x34f): warning: Using 'getaddrinfo' in statically linked applications requires at runtime the shared libraries from the glibc version used for linking
```

I don't entirely understand this, but what I think it says is that my statically linked executable will work on any machine that has the same C runtime as where I built it, but elsewhere, all bets are off.

So far in our limited testing this hasn't been a problem. If it becomes a problem, I see that there a number of conversations on the Net about it, so we can tap the accumulated wisdom of those folks for a solution.
