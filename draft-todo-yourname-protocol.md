---
title: "Continuous Updating and Ratcheting for Rekeying Encrypted Network Transport - Problem Statement"
abbrev: "CURRENT - Problem Statement"
category: info

docname: draft-todo-current-problem-statement
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: Security
workgroup: TBD
keyword:
 - continuous
 - asynchronous
 - updating
 - ratcheting
 - key update
venue:
  group: TBD
  type: Non-Working Group
  mail: km-fs-pcs@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/km-fs-pcs/
#  github: USER/REPO
#  latest: https://example.com/LATEST

author:
 -
    fullname: Your Name Here
    organization: Your Organization Here
    email: your.email@example.com

normative:

informative:

...

--- abstract

The IETF has published many protocols that require interactive exchanges
between endpoints. The security protocols developed by the IETF are no
exception to this design model. In some network environments, security
protocols that depend on persistent connections are less than ideal
because reestablishing a connection requires a new handshake that
consumes scarce power and bandwidth.

Some security protocols, for example Continuous Key Agreement protocols,
offer an alternative approach that establishes a connection once and then
performs asynchronous key updates thereafter. This approach requires
fewer round trips. Each endpoint advances its key state on its own schedule, so
one endpoint may initiate two key updates for every key update initiated
by the other endpoint. This allows the functionality and risk profile of
each endpoint to be adjusted to its constraints. Key updates also spread
the overhead of post-quantum cryptography across the life of the
connection rather than concentrating it in each handshake, and that
property in particular makes these approaches appealing for post-quantum
use. This document examines how asynchronous key updates can solve
problems in certain network environments. It also provides high-level
requirements for a solution in this space.

--- middle

# Introduction

The IETF has developed many protocols that are widely deployed. Many of
these require interactive exchanges between endpoints. Protocols that
support web browsers are among the best known and they fit this model
well. If you are browsing the Internet, this is the model you want. In
other cases, key updates that do not require an interactive exchange have
advantages.

Those cases arise in network environments with high latency and
unreliable network paths that come and go: networks with the potential
for long delays and many disruptions. They are not the only cases;
classic server-to-server communication applies as well. Protocols that
depend on persistent connections are not ideal there, because
reestablishing a connection requires a new handshake, and a new handshake
is time consuming and therefore consumes bandwidth and other resources.
Resumption is the usual answer to this objection, and it has been claimed
as sufficient. It is not, for two reasons developed in Section 5:
resumption does not preserve forward secrecy and post-compromise security
across an outage of operational length, and it does not spread the
post-quantum cost across the life of the connection.

This document concerns the key management aspects of establishing
connections between endpoints and updating their keys. It examines why a
Continuous Key Agreement protocol can provide new strategies in certain
network environments, and it provides high-level requirements for a
solution.

Three limits of scope are stated at the outset because each has been
misread previously. The problem statement and use cases are not intended
to apply to web browsers. They are restricted to the two-party case;
some Continuous Key Agreement protocols support group keys and those are
not in scope here. And a Continuous Key Agreement protocol need not be
an application layer protocol: the construction discussed here supplies
key schedule and ratcheting machinery to network and transport layer
security, and is not a proposal to carry an application layer protocol
in the network layer.

# Terminology

Endpoint:
: One of the two participants in a connection.

Connection:
: Shared protocol state between two endpoints. A connection can persist
while no network path is available.

Asynchronous network:
: The network does not guarantee that both endpoints are reachable at the
same time. Delivery may be delayed, use store-and-forward operation, or
be limited to contact windows.

Interactive key agreement:
: Key agreement that requires both endpoints to exchange messages before
new keying material can be used.

Asynchronous key update:
: An endpoint can advance the shared key state without a live exchange
with the other endpoint. The key update is valid when the other endpoint
receives it, whenever that is.

Continuous Key Agreement protocol:
: A cryptographic protocol with three properties: asynchronous key
updates, Proposals and Commits, and ratcheting.

Forward secrecy:
: Compromise of current key material does not expose traffic already
sent.

Post-compromise security:
: Traffic sent after recovery remains secure even if an endpoint was
compromised in the past.

Post-quantum cryptography:
: Cryptography secure against an adversary with a cryptanalytically
relevant quantum computer.

Amortization of post-quantum cost:
: Spreading the cost of post-quantum key establishment across the life
of a connection through many small key updates, rather than paying it in
full during each handshake.

Bundle Protocol:
: The store-and-forward protocol used in delay-tolerant networking.

0-RTT data:
: Application data sent in the first flight of a resumed connection. TLS
and QUIC do not provide inherent replay protection for 0-RTT data.


# Problem Statement

Conflating an asynchronous network with an asynchronous key update has
obscured what is being asked for. The recap meeting of 7 August recorded
this as a specific source of confusion at IETF 126. The requirement in
this document is for an asynchronous key update. An asynchronous network
is not the requirement itself.

Five problems follow.

1. Amortization of post-quantum cost:
: Post-quantum key material and signatures are large relative to the
payloads carried on constrained links. Where a handshake recurs, its
cost is not amortized over a long connection; it recurs with it, and
competes directly with the traffic the link exists to carry. Spreading
that overhead is already practiced at scale: Signal spreads
post-quantum overhead, which is otherwise burdensome, across its
key updates rather than paying it in one exchange.

2. Lack of asynchronous key updates:
: Interactive key agreement requires both endpoints to compute a key
update together in a live exchange; neither can advance the key state
alone. On network paths subject to contact windows, propagation delay, or
interruption, the handshake becomes the operation most likely to fail and
the one whose failure is hardest to recover.

3. Lack of uninterrupted resumption after network failure:
: Where a connection must be reestablished, the cost is paid again in full.
Mechanisms that reduce that cost exist, and Section 5 sets out why they
are not sufficient for the environments described here.

4. Forward secrecy and post-compromise security under intermittent connectivity:
: A connection that never performs a key update cannot recover from
compromise. A connection that supports only interactive key updates
cannot update its keys when the other endpoint is unreachable, which in
these environments is much of the time. Both properties therefore depend
on key updates that either endpoint can initiate independently.

5. Per-endpoint key update management:
: Connections today update keys on an all-or-nothing basis. With
asynchronous key updates, a network manager can set the key update
frequency of each endpoint according to its power and capability, and
schedule key updates so they do not disrupt workloads. Not disrupting
workloads matters both to post-quantum adoption and to network management.

# Use Cases

Each case states the environment, why the current handshake does not fit
it, and what is disputed. The disputes are recorded here rather than
omitted, because they are known and a reader who finds them later reads
them as omissions.

Numbers are deliberately absent. Where a link budget, a contact window
duration or an outage length would settle a case, that is said rather
than estimated.

<aside markdown="block">
See about getting numbers.
</aside>

## Space

Everything from banking information to critical infrastructure
management now flows through space communication systems. Public safety,
health and financial transactions are all high value targets, and they
motivate attacks against space communications.

Two characteristics distinguish this environment. Propagation delay
makes each round trip expensive in wall-clock time. The primary source
in Section 11 records round-trip times on the order of twenty to two
hundred and fifty milliseconds in low earth and geostationary orbit,
five to fourteen seconds for lunar communications, and between just
under one minute and twenty-three minutes between Earth and Mars
depending on orbital positions. It also records that delay-tolerant
protocols begin to outperform IP once round-trip times exceed roughly
two hundred milliseconds. At the upper end of that range an interactive
handshake is impractical rather than merely slow.

Contact windows bound the time available. An exchange that cannot
complete inside a window does not complete late; it fails and is retried
at the next window. Compounding this, post-quantum key material and
signatures are larger than their classical equivalents, so the exchange
that must fit inside the window is the one that has grown.

Store-and-forward operation follows from both. The Delay-Tolerant
Networking working group is progressing key agreement for Bundle
Protocol Security, which is independent evidence that this case is real
and that a constituency outside this work holds it.

## Internet of Things and Operational Technology Systems

Constrained radio links carry small payloads, and devices on them are
frequently power-limited and duty-cycled. Post-quantum key material and
signatures are large relative to those payloads, so a handshake is not
a fixed cost paid once at connection establishment. On a connection that
is reestablished often, it is a recurring charge against the mission
traffic.

Post-quantum keys, which are large by classical standards, exacerbate this
problem. For example, in a sensor network with many intermediaries where
sensors wake up and decide they want a new post-quantum key share and to
start sending data immediately, waiting for intermediaries to respond is
non-ideal. Further, any ability to amortize the cost of post-quantum
cryptography over multiple connections is ideal.

<aside markdown="block">
Disputed. It has been observed that implementations in this space could
use symmetric keys instead, particularly if signatures are also removed.
That is fair. The case stands where key distribution at scale, or
recovery from compromise, makes a purely symmetric approach unworkable,
and it should be argued on those grounds rather than on payload size
alone.
</aside>

## Virtual Private Networks

VPN connections are often long-lived. Establishing a post-quantum-secure
connection requires an initial post-quantum exchange, whose cost is paid in full
at connection establishment. After establishment, a Continuous Key Agreement
protocol can introduce fresh post-quantum key contributions through in-band key
updates without repeating a complete authenticated key exchange. The cost of
subsequent post-quantum key updates can therefore be distributed throughout the
life of the connection rather than being concentrated in repeated handshakes.

Asynchronous key updates also give an operator independent control over when and
how often each endpoint advances its key state. Key update schedules can use
independent key updates to account for security policy, workload, power, and
available network capacity.

This control is useful when a VPN client has asymmetric upload and download
availability. For example, a client with limited upload capacity can schedule
its outbound key updates for periods when that capacity is available while
continuing to receive key updates over a less constrained downstream path. The
endpoints need not initiate key updates at the same rate or on the same
schedule.

## Unidirectional Communications

Radio transmissions inherently reveal the location of the sender;
returning delivery acknowledgements would be enough to divulge the
sender's location. For use cases where privacy is highly sensitive,
unidirectional communications, in receive-only mode, is a requirement.
A Continuous Key Agreement protocol provides the best defense.

<aside markdown="block">
Disputed. In the case of unidirectional communications, the sender does
not know whether packets were delivered so there is a question about the
amount of survivability needed for lost key updates. Continuous Key
Agreement protocols are probably the best you can get in these
unidirectional cases.
</aside>

# Limitations of Current Solutions

This document does not argue against the extended key update work. The
most common objection heard at IETF 126 was that effort should go there
rather than to something new, and that objection deserves a direct
answer rather than silence.

For both QUIC and TLS the central issue is the absence of native support
for post-compromise security. An extended key update mechanism is being
developed to add it to both. Neither specification is complete, and the
associated proofs are not complete either. What follows is a statement
of what remains uncovered once that work lands, not a claim that it
should not proceed.

QUIC supports resumption, and it permits 0-RTT data on a new connection.
0-RTT reuses information from a previous connection and sacrifices replay
protection for efficiency. The QUIC designers have advised disabling
these features because of privacy concerns and replay vulnerabilities,
which makes them ill-advised as a means of addressing latency and
handshake overhead in the environments described here.
Despite those warnings, early testing of QUIC in space systems has
specifically examined 0-RTT, given the latency cost of secure connection
establishment. QUIC also does not offer post-compromise security once
keys are lost, including when resumption is used.

Neither QUIC nor TLS provides a means of amortizing the computational
and bandwidth cost of post-quantum algorithms across a long-lived
connection. Each establishment pays that cost in full.

Two points of maturity are worth recording, because the proposal is
sometimes read as untested. A variant replacing the QUIC handshake with
a Continuous Key Agreement protocol has been designed, implemented as a
prototype, benchmarked, and analyzed in a formal cryptographic model
with a security proof. Separately, an integration with Bundle Protocol
Security has been implemented against the Interplanetary Overlay
Network and Bundle Protocol Security reference implementations, with
the source published, and its costs measured against group size. Both
are cited in Section 11.

Those implementations also qualify a claim that is often made for this
approach. The theoretical scaling advantage over pairwise handshakes is
logarithmic in group size, and the measured behavior is generally
linear, with logarithmic scaling appearing only at ideal group sizes.
The advantage over repeated handshakes remains, and it is smaller than
the asymptotic figure suggests. This document states the measured
position rather than the theoretical one.

# Basic Requirements

These requirements are adapted from the presentations at IETF 125 and
IETF 126.

* The key management MUST: Support Layer 3 and Layer 4. Asynchronous
key updates. Post-quantum cryptography. Forward secrecy.
Post-compromise security. Protocol formal analysis.

* The key management SHOULD: Operate over asynchronous networks. Support
groups and peer-to-peer protocols.

A note on the groups item, since this document is restricted to the
two-party case: the item is carried from the primary source, which lists
scalability to groups of devices for Bundle Protocol use specifically.
The two positions are not contradictory, and Section 7 gives the reason
the two-party case is pursued here.

Expressed in terms of the environments in Section 4, a Continuous Key
Agreement protocol meeting these requirements needs to:

* support high-latency networks with the minimum practicable number of
round trips

* allow either endpoint to initiate a key update independently, at
regular or predetermined intervals

* allow the endpoints to update their key state at different rates, so
the functionality and risk profile of each endpoint can be adjusted
independently

* allow the key update frequency of each endpoint to be set according to
its power and capability, without disrupting workloads

* support keying at either or both the network and the transport layer

* support post-quantum cryptography with amortization options

* ensure traffic sent before current keying material is compromised
remains protected, which is forward secrecy

* such a protocol is not required to be stateless, because unlike
Internet use cases there is no risk of exhaustion from very large
numbers of connection establishments, and stateful protocols may
therefore offer better security or functionality. The environments
described in Section 4 involve relatively few paired identities with
connections that may persist for years, which is what makes stateful
operation available at all.

# Security Considerations

This document is entirely about security. It is a problem statement,
and the security considerations for specific solutions will be discussed
in solutions documents.

As stated in Section 1, this document concerns two-party use cases
rather than groups. That scope is deliberate. A mechanism that admits a
third party to a two-party connection raises concerns about
surveillance capability which are separate from, and likely to attract
more opposition than, the engineering questions raised here.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
