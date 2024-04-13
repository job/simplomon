# What simplomon does

> This file is not yet complete! However, everything you read here should be
> correct.

Simplomon is a self-contained program that runs *checkers* on your infrastructure.
Checkers can signal one or more alert conditions. Such an alert might be configured to
need a few repeats before it leads to a *notification*. 

Simplomon supports several notifiers, which can also be configured to only send
out alert conditions that persist for x minutes or more. Suitable for
management people. 

# Configuration
Configuration is read from a file that is actually interpreted as Lua code. 

A useful but minimal configuration file is:

```Lua
addEmailNotifier{from="bert@hubertnet.nl", to="bert@hubertnet.nl", server="10.0.0.2"}

-- at 10 AM UTC receive confirmation that things are working
dailyChime{utcHour=10}

-- check certificates, content, warn on certificates close expirey
https{url="https://berthub.eu/", regex="Europe"}

-- check if all nameservers have the same SOA record
dnssoa{domain="berthub.eu", 
	servers={"100.25.31.6", "86.82.68.237", "217.100.190.174"}
}
```

Simplomon will read its configuration file from "./simplomon.conf", or from
the first command line argument, or from the URL specified in the SIMPLOMON_CONFIG_URL
environment variable (for use in containers).

Because the configuration file is Lua, it is possible to store variables for
reuse, or to have loops that populate many checks from a loop etc.

# What simplomon does
The configuration file is read once, and it defines:

 * Checkers: what do we check
 * Notifiers: how to we notify alerts
 * Settings: how often do we check, using how many parallel checks etc
 * Logger: if alert, notifications, statistics get logged
 * Webserver: if we should launch the webserver, if there should be a
 dashboard

Simplomon by default tries to perform all checks once every minute. To do
so, it will by default perform at most 4 checks in parallel. After all
checks are done, simplomon determines how long this took. If it took longer
than the configured check interval, the number of allowed parallel checks is
raised by 1 for the next round. By default at most 16 checks will happen in
parallel.

# When does a notification go out?
This is a multi-step process, and it might currently be a bit too confusing.

A *checker* delivers a list of alerts. Simplomon then counts if there have
been more than `minFailures` alerts within `failureWindow` seconds. And if
so, the alert is passed on to the nofitier(s). The default values for most
checkers are 1 alert within 120 seconds. 

The upshot of this is that a single alert will persist for 2 minutes.
However, the Ping checker defaults to `minFailures=2`. This means that only
two subsequent failed pings will be reported to the notifiers.

By default the notifiers will just forward whatever they get. However, as an
additional feature, a notifier can introduce an additional delay. If a
notifier gets passed a `minMinutes` parameter, an actual notification will
only go out if a notification persists for that many minutes.

# All checkers
## dns

## dnssoa

## httpredir

## https
The https checker supports many things, but with sensible defaults you can
often just use `https{url="https://berthub.eu/"}` and be done. Of specific
note, if you have an AAAA IPv6 address for your domain name, and if your
Simplomon has working IPv6, this check will check both IPv4 and IPv6
automatically. 

Here are the parameters, of which only `url` is mandatory:

 * url: needs to include https. Simplomon will follow any redirects.
 * maxAgeMinutes: alert if the webserver says content is older than this
 * minCertDays: alert if a certificate in the chain expires within this many
   days (defaults to 14)
 * serverIP: perform the check for `url` on this IPv4/IPv6 address. Useful
   for if you know you have multiple backends, and want to force the test to
   happen on all of them.
 * localIPv4/localIPv6: bind to these addresses when connecting to IPv4 or
   IPv6. Can be useful to perform tests from systems with multiple internet
   connections.
 * minBytes: if the web server returns fewer bytes than this, it is an alert
 * regex: search for this regex in the returned content, and if it isn't
   found, this is an alert
 * method: GET or HEAD. Be aware that some sites effectively do not support
   HEAD, possibly because of "web firewalls"
 * dns: get the IP address from these nameservers. Useful when testing
   against DNS-based CDNs (like Akamai). 

## imap

## ping

## prometheusExp

## rrsig

## smtp

## tcpportclosed





