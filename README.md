![exitmap logo](https://nullhypothesis.github.com/exitmap_logo.png)
===================================================================

[![Build Status](https://travis-ci.org/NullHypothesis/exitmap.svg?branch=master)](https://travis-ci.org/NullHypothesis/exitmap)

Overview
--------

Exitmap is a fast and modular Python-based scanner for
[Tor](https://www.torproject.org) exit relays.  Exitmap modules implement tasks
that are run over (a subset of) all exit relays.  If you have a background in
functional programming, think of `exitmap` as a `map()` interface for Tor exit
relays.  Modules can perform any TCP-based networking task; fetching a web page,
uploading a file, connecting to an SSH server, or joining an IRC channel.

In practice, exitmap is useful to monitor the reliability and trustworthiness of
exit relays.  Mainly, we use exitmap to check for false negatives on the Tor
Project's [check](https://check.torproject.org) service and to find [malicious
exit relays](http://www.cs.kau.se/philwint/spoiled_onions).  It is easy to
develop new modules for exitmap; just have a look at the file HACKING in the
doc/ directory, or check out one of the existing modules.

Exitmap uses [Stem](https://stem.torproject.org) to create circuits to all given
exit relays, and as soon as tor notifies exitmap of an established circuit, a
module is invoked for the newly established circuit.  Modules can be pure Python
scripts or executables.  For executables,
[torsocks](https://github.com/dgoulet/torsocks/) is necessary.

Finally, note that exitmap is a network measurement tool and not useful to
ordinary Tor users.  The Tor Project is running the tool on a regular basis, and
more exitmap scans just cause additional network load.  The only reason exitmap
is publicly available is because its source code and design might be of interest
to some.

Installation
------------

Exitmap uses the library Stem to communicate with Tor.  There are
[plenty of ways](https://stem.torproject.org/download.html) to install Stem.
The easiest might be to use pip in combination with the provided
requirements.txt file:

    $ pip install -r requirements.txt

Running exitmap
---------------

The only argument exitmap requires is the name of a module.  For example, you
can run exitmap with the checktest module by running:

    $ ./bin/exitmap checktest

The command line output will then show you how Tor bootstraps, the output of the
checktest module, and a scan summary.  If you don't need three hops and prefer
to use two hops with a static first hop, run:

    $ ./bin/exitmap --first-hop CCEF02AA454C0AB0FE1AC68304F6D8C4220C1912 checktest

To run the same test over German exit relays only, execute:

    $ ./bin/exitmap --country DE --first-hop CCEF02AA454C0AB0FE1AC68304F6D8C4220C1912 checktest

If you want to pause for five seconds in between circuit creations to reduce the
load on the Tor network and the scanning destination, run:

    $ ./bin/exitmap --build-delay 5 checktest

Note that
[`CCEF02AA454C0AB0FE1AC68304F6D8C4220C1912`](https://atlas.torproject.org/#details/CCEF02AA454C0AB0FE1AC68304F6D8C4220C1912)
is a relay run by Karlstad University.  Feel free to use it, but try to use your
own relay to distribute the scanning load.

To get an overview of exitmap's other options, run:

    $ ./bin/exitmap --help

Exitmap comes with batteries included, providing the following modules:

* testfds: Tests if an exit relay is able to fetch the content of a simple
  web page.  If an exit relay is unable to do that, it might not have enough
  file descriptors available.
* checktest: Attempts to find false negatives in the Tor Project's
  [check](https://check.torproject.org) service.
* dnspoison: Attempts to resolve several domains and compares the received DNS A
  records to the expected records.
* dnssec: Detects exit relays whose resolver does not validate DNSSEC.
* patchingCheck: Checks for file tampering.
* cloudflared: Checks if a web site returns a CloudFlare CAPTCHA.

Configuration
-------------

By default, exitmap tries to read the file .exitmaprc in your home directory.
The file accepts all command line options, but you have to replace minuses with
underscores.  Here is an example:

    [Defaults]
    first_hop = CCEF02AA454C0AB0FE1AC68304F6D8C4220C1912
    verbosity = debug
    build_delay = 1
    analysis_dir = /path/to/exitmap_scans

Alternatives
------------

Don't like exitmap?  Then have a look at
[tortunnel](http://www.thoughtcrime.org/software/tortunnel/),
[SoaT](https://gitweb.torproject.org/torflow.git/tree/NetworkScanners/ExitAuthority/README.ExitScanning),
[torscanner](https://code.google.com/p/torscanner/),
[DetecTor](http://detector.io/DetecTor.html), or
[SelekTOR](http://www.dazzleships.net/selektor-for-linux/).

Tests
-----

Before submitting pull requests, please make sure that all unit tests pass by
running:

    $ pip install -r requirements-dev.txt
    $ py.test --cov-report term-missing --cov-config .coveragerc --cov=src test

Feedback
--------

Contact: Philipp Winter <phw@nymity.ch>  
OpenPGP fingerprint: `B369 E7A2 18FE CEAD EB96  8C73 CF70 89E3 D7FD C0D0`
