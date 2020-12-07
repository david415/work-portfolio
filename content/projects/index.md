+++
categories = ["projects"]
comments = false
date = ""
draft = false
slug = ""
tags = ["projects"]
title = "Open Source Projects"

showpagemeta = false
+++

David Stainton's open source and free software projects:

The following specified projects are a small sample of my open source
contributions and is not by any means an exhaustive list of the
software projects I've worked on. I have for instance, contributed to
various proprietary software projects which of course will not be
mentioned here.

Below you will find the repeated theme of networking and cryptography. Of course
I know how to load information from databases, transform data, etc. However most
of my independent computer security research involves minimal databases usage and
mostly involves networking and cryptography; my apologies if this is not the skillset you
are looking for. These projects represent how I spend my free time and do not represent
the totality of my skills as a software developer.

See [David Stainton's github profile](https://github.com/david415) for additional open source projects.


**Rust**

* [Noise based cryptographic network protocol for mixnets](https://github.com/sphinx-cryptography/mix_link)
* [Rust ffi bindings to AEZ written in C](https://github.com/sphinx-cryptography/aez)
* [Sphinx cryptographic packet format create](https://github.com/sphinx-cryptography/rust-sphinxcrypto)
* [Lioness wide block cipher crate](https://github.com/sphinx-cryptography/rust-lioness)
* [ECDH wrapper crate](https://github.com/sphinx-cryptography/ecdh_wrapper)
* [My solutions to some of the cryptopals challenges](https://github.com/david415/cryptopals)

**Golang**

* [Katzenpost](https://github.com/katzenpost)

From 2017 to around 2020 I've worked on an anonymous communication
network, specifically a decryption mix network known as
Katzenpost. This open source software project started off as part of
the Horizon 2020 Panoramix Project funded by the European Commision
and included collaborations with some of the top European computer
science researchers. The design of Katzenpost was largely based off of
[The Loopix Anonymity System](https://www.freehaven.net/anonbib/cache/loopix2017.pdf) which was published at 2017 Usenix
Security and Privacy conference.

This project included 3 months of collaborations with European
academics to collaboratively write our design specification documents
followed by several months of software development time. Katzenpost
has also received grant funding from Samsung and NLNet.

[See NLnet accouncement](https://nlnet.nl/project/katzenpost/index.html)

[See Announcing the Samsung NEXT Stack Zero Grant recipients.](https://samsungnext.com/whats-next/category/podcasts/decentralization-samsung-next-stack-zero-grant-recipients/)


* [Passive TCP injection attack detector ``Honeybadger''](https://github.com/david415/honeybadger) and [docs](https://honeybadger.readthedocs.org/)

In 2014, shortly after the Edward Snowden document leaks I wrote
Honeybadger, the most advanced detector of TCP injection
attacks. Honeybadger uses my own classification of injection attacks:

1. handshake hijack
2. segment veto
3. sloppy injection
4. out-of-order coalesce injection
5. censorship injection

To read about my TCP injection classification see: [TCP injection attack categories](https://github.com/david415/HoneyBadger_docs/blob/master/source/how-to-detect-TCP-injection-attacks.rst#tcp-injection-attack-categories)

During my work on Honeybadger I have also acted as an advisor for Google in the improvement of their Gopacket library.
In particular I was coresponding with Graeme Connell ( gconnell@google.com ) and I've also made contributions to a Google
design document in collaboration with Sentryo (sentryo.net), specifically Laurent Hausermann ( laurent.hausermann@sentryo.net )
in April of 2015. NOTE: I have never worked for Google; I am an independent security researcher!


* [BPF Sniffer API for Google's gopacket library](https://github.com/google/gopacket/blob/master/bsdbpf/bsd_bpf_sniffer.go)

I added the BSD BPF sniffer to Google's gopacket library as part of
the previously mentioned Honeybadger software project. I did this so
that Honeybadger could be efficiently used on BSD operating systems
such as FreeBSD without having to use libpcap. Might as well have Google
maintain the code for me, am I right? :)

* [Roflcoptor, Tor control port filter daemon for the Subgraph OS project](https://github.com/subgraph/roflcoptor)

I wrote the most sophisticated Tor control port filter daemon. The Tor
control port allows applications to control the Tor daemon and perform
various operations such as create an onion service or create a Tor
circuit. With Roflcoptor acting as a proxy to the Tor control port,
each application has a white list of allow Tor control port
commands. This greatly reduces the risk of deanonymization when giving
an application access to the Tor control port. (see Tor control port
specification
https://gitweb.torproject.org/torspec.git/tree/control-spec.txt )

* [Fixed a bug in the syscall module of the Go standard library](https://github.com/golang/go/issues/16681)

* [Wrote the IPFS Tor Onion transport](https://github.com/OpenBazaar/go-onion-transport)

* [Parasitic TCP traceroute tool](https://github.com/david415/ParasiticTraceroute)

My Parasitic TCP traceroute tool makes use of the Linux kernel's
nfqueue subsystem to mangle TCP packets of an existing connection in
order to perform a traceroute. Such a traceroute might in some
circumstances be useful for tracing a connection beyond an upstream NAT
(network address translation) device.

* [Contributed some code to Bulb, for the onion listener](https://github.com/Yawning/bulb)

Bulb is useful for Tor integration and this feature I added allows for Go applications to easily create an Tor onion service.

* [Added SOCKS5 proxy to Subgraph OS Firewall daemon](https://github.com/subgraph/fw-daemon)

* [Reported a subtle bug in Subgraph's Linux netfilter queue library](https://github.com/subgraph/go-nfnetlink/issues/1)

**Python**

Most of my Python experience involves async IO networking with Twisted, however, I've also used Tornado and blocking IO.
Many of my Python project involve networking and cryptography; this is after all, the main theme repeated throughout my work portfolio.
I don't have much experience with Python 3.

* [Contributed various feature patches to the Tahoe-LAFS project](https://github.com/tahoe-lafs/tahoe-lafs)

* [Forked Ian Goldberg's reference implementation of the Sphinx cryptographic packet format](https://github.com/applied-mixnetworks/sphinxmixcrypto)

I made the Sphinx cryptographic packet library compatible with Python 2 and Python 3. I've also upgraded some of the cryptographic primitives.

* [Wrote a Lioness wide block cipher library](https://github.com/david415/pylioness)

* [Wrote a active network scanner for a Linux kernel vulnerability that affected the Tor network](https://github.com/david415/scan_tor_rfc5961) and [sent a report to the tor-dev mailing list](https://lists.torproject.org/pipermail/tor-reports/2016-December/001105.html)

* [Wrote a Tor network partition detection scanner](https://github.com/david415/tor_partition_scanner)

* [Made various contributions to txtorcon](https://github.com/meejah/txtorcon)

* [Assisted with pycryptopp maintenance releases](https://github.com/tahoe-lafs/pycryptopp)

* [Assisted with Tor integration for Foolscap](https://github.com/warner/foolscap)

* [Wrote a socat-like tool using Twisted](https://github.com/david415/twistedcat)

* [Wrote a Tor onion transport VPN](https://github.com/david415/onionvpn)

* [Wrote Twisted http to tor proxy](https://github.com/david415/txtorhttpproxy)
