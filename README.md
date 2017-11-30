# sdtest
DNS Service Discovery subscription test tool.

## Overview

* sdtest is a client simulation tool for testing DNS subscriptions. It supports both the older Long Lived
    Queries (LLQ) and the new replacement DNS Push Notifications.

* sdtest will provide a command line interface (CLI) for interactive use or take commands from an input
    file. DNS Internet Class 'IN' is used in all cases. Optional arguments are in brackets.

```
  discover-soa [--verbose] [--force_ipv4|--force_ipv6] [--nameserver=<IPv4 or IPv6 address>] 
               [--protocol=llq|push|update] [--transport=udp|tcp|tls] _http._tcp.foo.bar.com
```

* find the DNS server responsible for subscriptions for the service using type SOA then, if defined,
    issue an SRV query for _dns-llq.`<proto>`.`<zone>` or _dns-push-tls._tcp.`<zone>`.
* default resolver will be used unless a nameserver is specified.
* default ports used: UDP 53, TCP 53, TLS 853
* can force IPv4 or IPv6. Otherwise, first IPv4 address is used
* defaults to Push Notifications over TLS for all commands. Push over TCP is allowed for testing only,
    but should not be deployed in production. Push over UDP is not permitted.

```
  test-dso-keepalive [--verbose] [--nameserver=<IPv4 or IPv6 address>]
                     [--transport=tcp|tls] [--keepalive-interval=n]
                     [--idle-timeout=n] [--pad]
```

 * Send a Stateful Operation Keepalive request and get back the servers       
     adjusted keepalive interval and idle timeout value.
 * If the nameserver doesn' support the stateful operations opcode, NOTIMP will be returned.
 * Other errors could be returned for other reasons. Consult the spec.
 
```
  test-dso-errors [--verbose] [--nameserver=<IPv4 or IPv6 address>]
                  [--transport=tcp|tls] [--keepalive-interval=n]
                  [--idle-timeout=n]
```

 * Test a server for different Stateful Operations error conditions.
 
```
  subscribe [--verbose] [--protocol=llq|push] [--target=<IPv4 or IPv6 address>]
            [--transport=udp|tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* After subscription confirmation, any matching instances will be displayed.
    Transport defaults to UDP for LLQ and TLS for Push. type defaults to PTR. An succesor ordinal number
    subscription id is assigned to each subscription starting with '1'.
* Discovery is performed for each subscription.
* When an explicit subscription is issued, the connection will remain open until there is an explicit 'unsubscribe',
    an explicit 'close', or the test program exits and automatically closes all sockets.
* By default, an SOA discovery followed by SRV discovery will locate the target to send the subscription to. The --target option can be used to bypass the discovery process.

```
  show nameservers
```

* Display current nameserver state.

```
  show subscriptions
```

* Display current subscription state.

```
  unsubscribe [--verbose] [--protocol=llq|push] [--target=<IPv4 or IPv6 address>]
              [--transport=udp|tcp|tls] [--force] [--type=PTR] id=n | _http._tcp.foo.bar.com
```

* must match the subscription exactly. Unsubscribe commands that do not match a current subscription
    will not be sent by default. Use --force to send them for testing Unsubscribe with no subscription.
    Use 'show subscriptions' to see  current subscription state if specifying by id.
* By default, the unsubscribe will be sent to the same address as the subscribe. This can be bypassed with the --target option.

```
  close id=n
```

* close the socket of an existing subscription without sending the unsubscribe. With TCP/TLS,
    the server will get signaled that the client has closed the socket. For LLQ over UDP,
    no indication will be sent. The id is discovered using 'show subscriptions'.

```
  query-soa [--verbose] [--target=<IPv4 or IPv6 address>] [--transport=udp|tcp|tls]
            [--id=n] [--type=PTR] _http._tcp.foo.bar.com
```

* Send query to SOA for record with type. Transport defaults to match subscription. While the subscription channel is
    intended for asynchronous responses from the server, direct unicast queries are supported and are included
    for testing. Subscription IDs are sequential numbers based on order of subscription and can be found with
    'show subscriptions'. Without the subscription ID, the first subscription will be the default.
* By default, the query will be sent the same address as the subscribe. This can be bypassed with the --target option.

```
  test-register-receive-single [--verbose] [--protocol=llq|push] [--transport=udp|tcp|tls]
                               [--target=foo.bar.com] [--txt="PATH=/"] [--port=8080] [--timeout=2]
                               instance=server._http._tcp.foo.bar.com
```
  
* register a service over mDNS and then look for an LLQ subscription notification from the server. Includes PTR, SRV, TXT.
    This will register the service instance, receive the service notification and compare.
    It will wait _timeout_ seconds (default 2) before declaring an error if it is not received.
    It assumes you have already subscribed to the service with the __subscribe__ command.

```
  test-register-receive-poisson [--verbose] [--rate=50] [--interval=2] [--protocol=llq|push]
                                [--transport=udp|tcp|tls] [--service=_poisson-llq._tcp]
                                [--timeout=4] domain=foo.bar.com
```

* this is an automated test which subscribes to a service (defaults to _poisson-llq._tcp), then generates 'rate'
    events per second over the interval in seconds using a poisson distribution and correlates the responses.
    After the timeout, it unsubscribes from the service.

## Building from git

Dependencies: libedit, libevent, openssl

To build in general, use:
```
autoreconf -i
./configure
make
```

On MacOSX, use:

```
brew install pkgconfig autoconf automake
brew tap pusateri/homebrew-libevhtp
brew install libevhtp
brew install openssl libevent libedit

autoreconf -i
./configure --with-pkg-config-path=/usr/local/opt/openssl/lib/pkgconfig:/usr/local/opt/libedit/lib/pkgconfig
make
```

On FreeBSD,

```
pkg install libedit openssl
echo WITH_OPENSSL_PORT=yes >> /etc/make.config
cd /usr/ports/devel/libevent2; make install     # build libevent2 with openssl port, not system version of openssl

autoreconf -i
./configure --prefix=/usr/local
make

```

On Ubuntu 16.04 LTS,

```
sudo apt-get install libedit-dev
```

