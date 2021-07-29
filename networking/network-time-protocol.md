# Network Time Protocol

> What else may hap, to time I will commit. 
Only shape thou thy silence to my wit.

Oh time, how tricky you are! I came across the "NTP" while learning about [Lamport and vector clocks](https://people.cs.rutgers.edu/~pxk/417/notes/logical-clocks.html), which in turn came up while I was reading about Amazon's design choices in [Dynamo](https://sites.cs.ucsb.edu/~agrawal/fall2009/dynamo.pdf).

Warranting a TIL in their own right, these "logical clocks" are helpful for determining chronology and causality in the context of distributed systems, which may not have a globally synchronized *physical* clock to tell us what "happened before" via simple timestamps. Each node in a system may indeed have a mechanism for generating such timestamps; the trouble comes when we try to reconcile differing timestamps that may have drifted from each other or may even match for concurrent actions. See [clock drift and skew in these neat slides](https://cse.buffalo.edu/~stevko/courses/cse486/spring14/lectures/06-time.pdf).

 I started to pull on this underlying thread: how *do* we know what time it is? How are my "physical" clocks synchronized?? My laptop's time always seems to match my phone. How do they agree? Well, they are likely checking in with official time servers, via the *Network Time Protocol*.

 > NTP stands for Network Time Protocol, and it is an Internet protocol used to synchronize the clocks of computers to some time reference.

 Very concisely per [this amazingly old school FAQ](http://www.ntp.org/ntpfaq/NTP-s-def.htm)

The need for such a protocol makes sense. Different machines in different places, built with different technology may very well disagree about what time it is. However, once these private machines and even whole private networks link into the Internet, there needs to be some standard agreement on what time it is; otherwise emails can arrive minutes before the send! Indeed, this is a problem humanity has encountered [before](https://www.jstor.org/stable/3105430).

## Overview

Ok, so. The NTP aims to synchronize time across machines to with a few milliseconds of the official time in UTC.

"UTC" or "Universal Time, Coordinated" is the same no matter where you are on Earth:
* Universal: not local time. To get a local timestamp from a UTC timestamp, you must add or subtract the local time's zone offset.
* Coordinated: UTC is built on estimates from several official time-keeping institutions.

To do this, the NTP is the interface for a networked system with both public and private components, arranged hiearchically:

* __Stratum 0__: highly precise mechanisms for official time-keeping. Atomic clocks, radio clocks, etc. all over the world and in space and the ocean floor. No NTP server can publicly identify as Stratum 0.
* __Stratum 1__: computers in this layer are tightly synchronized to the reference clocks in Stratum 0 and they do "sanity checks" with other nodes in this layer. These are the "primary" time servers.
* __Stratum 2__: computers / servers in this layer sync with other S2 nodes and verify time with with S1 nodes.
* ...
* __Stratum 15__: the max layer for sampling time. Like all layers "above", this layer, it syncs to the previous layer.

Given the above tree of servers, NTP algorithms on each computer use a shortest-path spanning tree to reach S1 servers to get the official UTC time, as well as the time it takes to get responses.

An NTP client polls NTP servers and then computes its *time offset* and its *round-trip delay*. The values are subject to STATISTICS: filters, outliers identified and discarded, confident estimates are derived from at lest 3 candidates. Then, while polling, the client process gradually updates its clock frequency OR manages the machine's "systemic bias" (i.e. time-warp it forward or backward in time!). 🤯

Because I'm running a Mac, I see my NTP client is a daemon called `timed`:

```console
> man timed

TIMED(8)                  BSD System Manager's Manual                 TIMED(8)

NAME
     timed -- time synchronization daemon

SYNOPSIS
     timed takes no arguments, and users should not launch it manually.

DESCRIPTION
     timed maintains system clock accuracy by synchronizing the clock with reference clocks via technologies like NTP.  Inputs are merged inside of timed, where it calculates uncertainty to facilitate scheduling proactive time jobs. timed is also aware of power/battery conditions.

```

And, there it is polling away:

```console

➜  til git:(main) ✗ ps aux | grep timed
_timed             117   0.0  0.0  4360976   3396   ??  Ss   11:13PM   0:00.36 /usr/libexec/timed
```

### Errata

Ok, some fun NTP Facts:
* We are currently mostly using v4, per [RFC 5905](https://datatracker.ietf.org/doc/html/rfc5905) from 2010
* A new, revised RFC may drop soon??
* The original specification of NTP dates to [1985](https://datatracker.ietf.org/doc/html/rfc958)
* Can be thought of as a P2P *or* Client-Server paradigm
* Built on [UDP and uses Port 123](https://docstore.mik.ua/orelly/networking/firewall/ch08_13.htm)
* Uhhh there are some security issues (see [here](https://www.cloudflare.com/learning/ddos/ntp-amplification-ddos-attack/) and [here](https://en.wikipedia.org/wiki/NTP_server_misuse_and_abuse))