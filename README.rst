
.. image:: https://travis-ci.org/katzenpost/catshadow.svg?branch=master
  :target: https://travis-ci.org/katzenpost/catshadow

.. image:: https://godoc.org/github.com/katzenpost/catshadow?status.svg
  :target: https://godoc.org/github.com/katzenpost/catshadow


the catshadow client
====================

Catshadow is a mix network messaging system. This repository contains
a client library which can be used with a Katzenpost mix network. It
not only uses strong modern end to end encryption (Noise + Double
Ratchet), but it is also designed to reduce the amount of metadata
leaked onto the network.

This code is actively being developed and is intended
to be used with our Qt user interface, catchat:

* https://github.com/katzenpost/catchat


contact
=======

* IRC: irc.oftc.net #katzenpost <irc://irc.oftc.net/#katzenpost>
* Mailing List <https://lists.mixnetworks.org/listinfo/katzenpost>


disclaimer
==========

Katzenpost is still pre-alpha.  DO NOT DEPEND ON IT FOR STRONG SECURITY OR ANONYMITY.


testing
=======

optional docker tests
---------------------

To run the optional docker tests firstly, see our docker repo
and start your local dockerized mix network:

https://github.com/katzenpost/docker

A couple of minutes after startup run the tests like this:
::

   GORACE=history_size=7 go test -tags=docker_test -race -v -run Docker

This will run our docker based integration tests for the catshadow library.


design
======

It is my understanding that in terms of the analysis presented in this
blog post ( Brian Warner's **"Petmail mailbox-server delivery protocol"**
http://www.lothar.com/blog/53-petmail-delivery/ ),
the catshadow messaging system can be described as:

**S1, M0, R1, Rev0**

Here I rephrase the definitions of the above messaging system
properties: Catshadow clients can compare delivery tokens to determine
if they share contacts. However the message spool server cannot tell
which message came from which sender, not even that two messages came
from the same sender, nor can it determine how many senders might be
configured for each recipient. The recipient cannot use the transport
information to identify the sender. The recipient depends upon
information not visible to the mailbox server to identify the sender,
which means a legitimate (but annoying) sender could flood the server
without revealing which sender they are. Finally, the revocation
behavior is such that the recipient can revoke one or more senders
without involving the remaining senders.

Clients make use of a Sphinx SURB based protocol to retrieve messages
from their remote spool service. The mix network has several providers
which operate spool services which clients can interact with. The
spool service is in fact a seperate process which uses our CBOR/HTTP
over unix domain socket plugin system to communicate with the mix server.

Over time I plan on replacing the spool services with gradually more
sophisticated spool services until I finally have a replicating CRDT
based spool service which can help eliminate single points of failure
in this messaging system.

Clients make use of the PANDA protocol for exchanging spool identities
and the Signal Double Ratchet keys. That is, this messaging system creates
bidirectional metadata leakage resistant communications channels which
are composed with two unidirection channels. Each unidirectional channel
contains the required information to write to a correspondant's
remote message spool.

Katzenpost is a variant of the Loopix design and as such makes use of
the Poisson mix strategy and therefore must be properly tuned. Tuning
of the Poisson mix strategy has not been publicly solved yet but I
suspect the solution has something to do with a discrete network event
simulator and possibly some machine learning algorithms as
well. Perhaps we all should consider the tuning of this mixnet
messaging system as half of its design.

Another unfinished design area is: The Catshadow client periodically
polls the client's remote message spool where the intervals between
polling are the result of a Poisson process. Currently, tuning this
Poisson procress is left unfinished, however, I can state that the
goal in tuning this would be to reduce vulnerability to a long term
statistical disclosure attack where the passive adversary or
compromised Provider tries to link clients with their spool
service.


**"The Loopix Anonymity System"**:

https://www.usenix.org/system/files/conference/usenixsecurity17/sec17-piotrowska.pdf


the longer design overview
--------------------------

The design of this messaging is not yet fully specified but is
partially specified in these specification documents:

* https://github.com/katzenpost/docs/blob/master/drafts/client.rst
* https://github.com/katzenpost/docs/blob/master/drafts/deaddrop.rst
* https://github.com/katzenpost/docs/blob/master/drafts/decoy_traffic.rst
* https://github.com/katzenpost/docs/blob/master/drafts/panda.txt

Whereas all those specifications assume the existence of the core
Katzenpost specifications here which mostly covers the design of
the server infrastructure:

* https://github.com/katzenpost/docs/blob/master/specs/mixnet.rst
* https://github.com/katzenpost/docs/blob/master/specs/wire-protocol.rst
* https://github.com/katzenpost/docs/blob/master/specs/kaetzchen.rst
* https://github.com/katzenpost/docs/blob/master/specs/sphinx_replay_detection.rst
* https://github.com/katzenpost/docs/blob/master/specs/sphinx.rst
* https://github.com/katzenpost/docs/blob/master/specs/end_to_end.rst
* https://github.com/katzenpost/docs/blob/master/specs/pki.rst
* https://github.com/katzenpost/docs/blob/master/specs/certificate.rst

There is an older copy of our core Katzenpost specifications rendered
in Latex if you prefer to read it that way:
https://panoramix-project.eu/wp-content/uploads/2019/03/D7.2.pdf


code organization
=================

This repository contains a small amount of high level client
code. This client depends on lots of code in other Katzenpost
repositories including my fork of agl's PANDA and agl's Signal Double
Ratchet:

* https://github.com/katzenpost/doubleratchet
* https://github.com/katzenpost/panda
* https://github.com/katzenpost/channels
* https://github.com/katzenpost/memspool
* https://github.com/katzenpost/client
* https://github.com/katzenpost/minclient
* https://github.com/katzenpost/core


license
=======

AGPL: see LICENSE file for details.


acknowledgments
===============

* I would like to thank Leif Ryge for feedback during the design of this
  client and many of its protocols.

* I would like to also thank Adam Langely for writing [Pond](https://github.com/agl/pond)
  which has very obviously inspired a few of our design choices and has provided some
  code that we use such as the PANDA cryptographic protocol and the Signal Double Ratchet.


supported by
============

The development of the Catshadow Katzenpost client has been supported by the Samsung Next Stack Zero grant.
See **Announcing the Samsung NEXT Stack Zero Grant recipients**.

https://samsungnext.com/whats-next/category/podcasts/decentralization-samsung-next-stack-zero-grant-recipients/
