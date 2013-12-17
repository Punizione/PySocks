PySocks
=======

Updated version of SocksiPy. Many old bugs fixed, including a bug in `__recvall()` and a bug in `__negotiatehttp()`.

**Version:** 1.4.2

----------------

Features
========

* Fully supports Python 2.6 - 3.3

* SocksiPyHandler, courtesy e000, was also added as an example of how this module can be used with urllib2. See example code in sockshandler.py.

* urllib3, which powers the requests module, is working on integrating SOCKS proxy support based on this branch

* `SOCKS5`, `SOCKS4`, and `HTTP` are now aliases for `PROXY_TYPE_SOCKS5`, `PROXY_TYPE_SOCKS4`, and `PROXY_TYPE_HTTP`

* Tests added

* Various style and performance improvements; codebase simplified

* Actively maintained

Installation
============

    python setup.py install

--------------------------------------------

*Warning:* PySocks/SocksiPy only supports HTTP proxies that use CONNECT tunneling. Certain HTTP proxies may not work with this library. It is recommended that you rely on your HTTP client's native proxy support instead when connecting through non-SOCKS proxies.

--------------------------------------------

Usage
=====

Original SocksiPy README attached below, amended to reflect API changes.

--------------------------------------------

SocksiPy - version 1.4.2

A Python SOCKS module.

(C) 2006 Dan-Haim. All rights reserved.

See LICENSE file for details.


*WHAT IS A SOCKS PROXY?*

A SOCKS proxy is a proxy server at the TCP level. In other words, it acts as
a tunnel, relaying all traffic going through it without modifying it.
SOCKS proxies can be used to relay traffic using any network protocol that
uses TCP.

*WHAT IS SOCKSIPY?*

This Python module allows you to create TCP connections through a SOCKS
proxy without any special effort.

*PROXY COMPATIBILITY*

SocksiPy is compatible with three different types of proxies:
1. SOCKS Version 4 (SOCKS4), including the SOCKS4a extension.
2. SOCKS Version 5 (SOCKS5).
3. HTTP Proxies which support tunneling using the CONNECT method.

*SYSTEM REQUIREMENTS*

Being written in Python, SocksiPy can run on any platform that has a Python
interpreter and TCP/IP support.
This module has been tested with Python 2.3 and should work with greater versions
just as well.


INSTALLATION
-------------

Simply copy the file "socks.py" to your Python's `lib/site-packages` directory,
and you're ready to go. [Editor's note: it is better to use `python setup.py install` for PySocks]


USAGE
------

First load the socks module with the command:

    >>> import socks
    >>>

The socks module provides a class called `socksocket`, which is the base to
all of the module's functionality.
The `socksocket` object has the same initialization parameters as the normal socket
object to ensure maximal compatibility, however it should be noted that `socksocket`
will only function with family being `AF_INET` and type being `SOCK_STREAM`.
Generally, it is best to initialize the `socksocket` object with no parameters

    >>> s = socks.socksocket()
    >>>

The `socksocket` object has an interface which is very similiar to socket's (in fact
the `socksocket` class is derived from socket) with a few extra methods.
To select the proxy server you would like to use, use the `set_proxy` method, whose
syntax is:

    set_proxy(proxy_type, addr[, port[, rdns[, username[, password]]]])

Explanation of the parameters:

`proxy_type` - The type of the proxy server. This can be one of three possible
choices: `PROXY_TYPE_SOCKS4`, `PROXY_TYPE_SOCKS5` and `PROXY_TYPE_HTTP` for SOCKS4,
SOCKS5 and HTTP servers respectively. `SOCKS4`, `SOCKS5`, and `HTTP` are all aliases, respectively.

`addr` - The IP address or DNS name of the proxy server.

`port` - The port of the proxy server. Defaults to 1080 for socks and 8080 for http.

`rdns` - This is a boolean flag than modifies the behavior regarding DNS resolving.
If it is set to True, DNS resolving will be preformed remotely, on the server.
If it is set to False, DNS resolving will be preformed locally. Please note that
setting this to True with SOCKS4 servers actually use an extension to the protocol,
called SOCKS4a, which may not be supported on all servers (SOCKS5 and http servers
always support DNS). The default is True.

`username` - For SOCKS5 servers, this allows simple username / password authentication
with the server. For SOCKS4 servers, this parameter will be sent as the userid.
This parameter is ignored if an HTTP server is being used. If it is not provided,
authentication will not be used (servers may accept unauthentication requests).

`password` - This parameter is valid only for SOCKS5 servers and specifies the
respective password for the username provided.

Example of usage:

    >>> s.set_proxy(socks.SOCKS5, "socks.example.com") # uses default port 1080
    >>> s.set_proxy(socks.SOCKS4, "socks.test.com", 1081)

After the set_proxy method has been called, simply call the connect method with the
traditional parameters to establish a connection through the proxy:

    >>> s.connect(("www.sourceforge.net", 80))
    >>>

Connection will take a bit longer to allow negotiation with the proxy server.
Please note that calling connect without calling `set_proxy` earlier will connect
without a proxy (just like a regular socket).

Errors: Any errors in the connection process will trigger exceptions. The exception
may either be generated by the underlying socket layer or may be custom module
exceptions, whose details follow:

class `ProxyError` - This is a base exception class. It is not raised directly but
rather all other exception classes raised by this module are derived from it.
This allows an easy way to catch all proxy-related errors. It descends from `IOError`.

class `GeneralProxyError` - When thrown, it indicates a problem which does not fall
into another category. The parameter is a tuple containing an error code and a
description of the error, from the following list:
1 - invalid data - This error means that unexpected data has been received from
the server. The most common reason is that the server specified as the proxy is
not really a SOCKS4/SOCKS5/HTTP proxy, or maybe the proxy type specified is wrong.
4 - bad proxy type - This will be raised if the type of the proxy supplied to the
set_proxy function was not PROXY_TYPE_SOCKS4/PROXY_TYPE_SOCKS5/PROXY_TYPE_HTTP.
5 - bad input - This will be raised if the connect method is called with bad input
parameters.

class `SOCKS5AuthError` - This indicates that the connection through a SOCKS5 server
failed due to an authentication problem. The parameter is a tuple containing a
code and a description message according to the following list:

1 - authentication is required - This will happen if you use a SOCKS5 server which
requires authentication without providing a username / password at all.
2 - all offered authentication methods were rejected - This will happen if the proxy
requires a special authentication method which is not supported by this module.
3 - unknown username or invalid password - Self descriptive.

class `SOCKS5Error` - This will be raised for SOCKS5 errors which are not related to
authentication. The parameter is a tuple containing a code and a description of the
error, as given by the server. The possible errors, according to the RFC are:

1 - General SOCKS server failure - If for any reason the proxy server is unable to
fulfill your request (internal server error).
2 - connection not allowed by ruleset - If the address you're trying to connect to
is blacklisted on the server or requires authentication.
3 - Network unreachable - The target could not be contacted. A router on the network
had replied with a destination net unreachable error.
4 - Host unreachable - The target could not be contacted. A router on the network
had replied with a destination host unreachable error.
5 - Connection refused - The target server has actively refused the connection
(the requested port is closed).
6 - TTL expired - The TTL value of the SYN packet from the proxy to the target server
has expired. This usually means that there are network problems causing the packet
to be caught in a router-to-router "ping-pong".
7 - Command not supported - The client has issued an invalid command. When using this
module, this error should not occur.
8 - Address type not supported - The client has provided an invalid address type.
When using this module, this error should not occur.

class `SOCKS4Error` - This will be raised for SOCKS4 errors. The parameter is a tuple
containing a code and a description of the error, as given by the server. The
possible error, according to the specification are:

1 - Request rejected or failed - Will be raised in the event of an failure for any
reason other then the two mentioned next.
2 - request rejected because SOCKS server cannot connect to identd on the client -
The Socks server had tried an ident lookup on your computer and has failed. In this
case you should run an identd server and/or configure your firewall to allow incoming
connections to local port 113 from the remote server.
3 - request rejected because the client program and identd report different user-ids - 
The Socks server had performed an ident lookup on your computer and has received a
different userid than the one you have provided. Change your userid (through the
username parameter of the set_proxy method) to match and try again.

class `HTTPError` - This will be raised for HTTP errors. The parameter is a tuple
containing the HTTP status code and the description of the server.


After establishing the connection, the object behaves like a standard socket.
Call the close method to close the connection.

In addition to the `socksocke`t class, an additional function worth mentioning is the
`set_default_proxy` function. The parameters are the same as the `set_proxy` method.
This function will set default proxy settings for newly created `socksocket` objects,
in which the proxy settings haven't been changed via the `set_proxy` method.
This is quite useful if you wish to force 3rd party modules to use a socks proxy,
by overriding the socket object.
For example:

    >>> socks.set_default_proxy(socks.PROXY_TYPE_SOCKS5, "socks.example.com")
    >>> socket.socket = socks.socksocket
    >>> urllib.urlopen("http://www.sourceforge.net/")


PROBLEMS
---------

Please open a GitHub issue at https://github.com/Anorov/PySocks