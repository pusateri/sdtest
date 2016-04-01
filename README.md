# sdsub
DNS Service Discovery subscription test tool

## Overview

* sdsub is a client simulation tool for testing DNS subscriptions. It supports both the older Long Lived Queries (LLQ) and the new replacement DNS Push Notifications.

* sdsub will provide a command line interface (CLI) for interactive use or take commands from an input file. Comands include (IN class is assumed):

```
  llq-discover _http._tcp.foo.bar.com | _http._tcp.bar.com
```

* find the DNS server responsible for LLQ subscriptions for the service using type SOA then issue an SRV query for _dns-llq._udp.`<zone>`.

* A domain or subdomain can be specified. If a domain is specified, and multiple subdomains are browseable, multiple DNS queries and responses will be sent and received.
     
```
  llq-subscribe [--transport=udp|tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* transport defaults to UDP, type defaults to PTR. Discovery is performed for each subscription.

```
  llq-unsubscribe [--transport=udp|tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* must match the subscription exactly.

```
  push-discover _http._tcp.foo.bar.com | _http._tcp.bar.com
```

* find the DNS server responsible for push notifications for the service using type SOA, then issue an SRV query for _dns-push-tls._tcp.<zone>.
     A domain or subdomain can be specified. If a domain is specified, and multiple subdomains are browseable, multiple DNS servers will be returned.

```
  push-subscribe [--transport=tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* The default transport is TLS. TCP without TLS may be used for testing but is not expected to be used in a production server.

```
  push-unsubscribe [--transport=tcp|tls] [--type=PTR] _http._tcp.foo.bar.com
```

* Must match the subscription exactly.

```
  register-mdns --instance=server._http._tcp.foo.bar.com --target=foo.bar.com [--port=1080] [--timeout=2]
```
  
* register a service over mDNS and then look for a subscription notification from the server. Includes PTR, SRV, TXT.
    This will subscribe, register the service instance, receive the service notification, and unsubscribe from the service.
    It will wait _timeout_ seconds (default 2) before declaring an error. 

```
  llq-poisson [--rate=50] [--interval=2] [--transport=udp|tcp|tls] [--service=_poisson-llq._tcp] 
              [--timeout=4] --domain=foo.bar.com
```

* this is an automated test which subscribes to a service (defaults to _poisson-llq._tcp), then generates 'rate' events per second over the interval in seconds using a poisson distribution, correlates the responses, then unsubscribes from the service.

```
  push-poisson [--rate=50] [--interval=2] [--transport=tcp|tls] [--service=_poisson-push._tcp]
               [--timeout=4] --domain=foo.bar.com
```

* this is an automated test which subscribes to a service (defaults to _poisson-push._tcp), then generates 'rate' events per second over the interval in seconds using a poisson distribution, correlates the responses, then unsubscribes from the service.

## Building from git
```
autoreconf -i
./configure
make
```
Dependencies
ldns

On MacOSX, use
```
brew install ldns
autoreconf -i
./configure --with-pkg-config-path=/usr/local/Cellar/ldns/1.6.17_1/lib/pkgconfig
make

libedit
    LDFLAGS:  -L/usr/local/opt/libedit/lib
    CPPFLAGS: -I/usr/local/opt/libedit/include

```
