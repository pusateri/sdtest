# sdtest
DNS Service Discovery subscription test tool

## Overview

* sdtest is a client simulation tool for testing DNS subscriptions. It supports both the older Long Lived Queries (LLQ) and the new replacement DNS Push Notifications.

* sdtest will provide a command line interface (CLI) for interactive use or take commands from an input file. DNS Internet Class 'IN' is used in all cases. Optional arguments are in brackets.

```
  discover-soa [--protocol=llq|push|update] [--transport=udp|tcp|tls] _http._tcp.foo.bar.com
```

* find the DNS server responsible for subscriptions for the service using type SOA then, if defined, issue an SRV query for _dns-llq.`<proto>`.`<zone>` or _dns-push-tls._tcp.`<zone>`.

* defaults to Push Notifications over TLS for all commands. Push over TCP is allowed for testing only, but should not be deployed in production. Push over UDP is not permitted.
     
```
  subscribe [--protocol=llq|push] [--transport=udp|tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* After subscription confirmation, any matching instances will be displayed. Transport defaults to UDP, type defaults to PTR.   Discovery is performed for each subscription.

```
  unsubscribe [--protocol=llq|push] [--transport=udp|tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* must match the subscription exactly.

```
  test-single [--protocol=llq|push] [--transport=udp|tcp|tls] --instance=server._http._tcp.foo.bar.com
              [--target=foo.bar.com] [-txt="PATH=/"] [--port=8080] [--timeout=2]
```
  
* register a service over mDNS and then look for an LLQ subscription notification from the server. Includes PTR, SRV, TXT.
    This will register the service instance, receive the service notification and compare.
    It will wait _timeout_ seconds (default 2) before declaring an error if it is not received.
    It assumes you have already subscribed to the service with the __subscribe__ command.

```
  test-poisson [--rate=50] [--interval=2] [--protocol=llq|push] [--transport=udp|tcp|tls]
               [--service=_poisson-llq._tcp] [--timeout=4] --domain=foo.bar.com
```

* this is an automated test which subscribes to a service (defaults to _poisson-llq._tcp), then generates 'rate' events per second over the interval in seconds using a poisson distribution and correlates the responses. After the timeout, it unsubscribes from the service.

## Building from git
```
autoreconf -i
./configure
make
```

Dependencies: ldns

On MacOSX, use:

```
brew install ldns
brew install openssl
brew install homebrew/dupes/libedit

autoreconf -i
./configure --with-pkg-config-path=/usr/local/Cellar/ldns/1.6.17_1/lib/pkgconfig:/usr/local/opt/openssl/lib/pkgconfig
make
```
