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
 - rekey
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

The IETF has published many protocols that are synchronous, meaning
protocols that need continuous, real-time interactive sessions between
client and server. The security protocols developed by the IETF are no
exception to this design model. In some network environments,
session-based security protocols are less than ideal because of the need
to reestablish sessions via new handshakes that consume scarce power and
bandwidth.

Some security protocols, for example Continuous Key Agreement protocols,
offer an alternative approach that starts a session once and then
performs asynchronous key updates thereafter. The gain is not only fewer
round trips. Each party advances the key state on its own schedule, so
one device may update twice for every one update of its peer, which lets
the functionality and risk profile of each device be adjusted to its
constraints. Updates also spread the overhead of post- quantum
cryptography across the life of the association rather than concentrating
it in each handshake, and that property in particular makes these
approaches appealing for post-quantum use. This document examines the
reasons why asynchronicity can provide new strategies for solving
problems in certain network environments. It also provides high level
requirements for a solution in this space.

--- middle

# Introduction

The IETF has developed many protocols that are widely deployed. Many of
these follow a synchronous design pattern, one where clients and servers
use continuous, real-time interactive sessions. Protocols that support
web browsers are among the best known and they fit this paradigm well.
If you are browsing the Internet, this is the model you want. There are
other cases where an asynchronous design pattern has advantages.

Those cases arise in network environments with high latency, and with
unreliable links that come and go: networks with the potential for long
delays and many disruptions. They are not the only cases; classic
server-to-server communication applies as well. Session-based protocols
are not ideal there, because re-establishing a session requires a new
handshake, and a new handshake is time consuming and therefore bandwidth
and resource consuming as well. Session resumption is the usual answer
to this objection, and it has been claimed as sufficient. It is not, for
two reasons developed at section 5: resumption does not preserve forward
secrecy and post-compromise security across an outage of operational
length, and it does not spread the post-quantum cost of the association
across its life.

This document concerns the key management aspects of establishing and
updating links between client and server. It examines why a Continuous
Key Agreement protocol can provide new strategies in certain network
environments, and it provides high level requirements for a solution.

Three limits of scope are stated at the outset because each has been
misread previously. The problem statement and use cases are not intended
to apply to web browsers. They are restricted to the two-party case;
some Continuous Key Agreement protocols support group keys and those are
not in scope here. And a Continuous Key Agreement protocol need not be
an application layer protocol: the construction discussed here supplies
key schedule and ratcheting machinery to network and transport layer
security, and is not a proposal to carry an application layer protocol
in the network layer.

# Problem Statement

Two distinct properties are both commonly described as asynchronicity,
and conflating them has obscured what is being asked for. The recap
meeting of 7 August recorded this as a specific source of confusion at
IETF 126.

*1.* Asynchronous Network:

What it means:
: The network does not guarantee that both parties are
reachable at the same time. Delivery may be delayed,
store and forward, or subject to contact windows.

What it does not mean:
: It says nothing about how keys are managed. A
protocol can operate over such a network using
entirely interactive key agreement, and fail for that
reason.

*2.* Asynchronous Key Update:

What it means:
: A party can advance the shared key state without a live
exchange with its peer. The update is valid when the peer
receives it, whenever that is.

What it does not mean:
: It does not require an asynchronous network. A
protocol on a well connected link may still need this in
order to rekey without paying for a fresh handshake.

The requirement in this document is for the second. An asynchronous
network is not the requirement itself.

Five problems follow.

1. Amortization of post-quantum cost:
: Post-quantum key material and signatures are large relative to the
payloads carried on constrained links. Where a handshake recurs, its
cost is not amortized over a long session; it recurs with it, and
competes directly with the traffic the link exists to carry. Spreading
that overhead is already practiced at scale: Signal spreads
post-quantum overhead, which is otherwise burdensome, across its
updates rather than paying it in one exchange.

2. Lack of asynchronicity:
: Interactive key agreement requires both parties to compute the update
together in a live exchange; neither can advance the key state alone. On
links subject to contact windows, propagation delay, or interruption,
the handshake becomes the operation most likely to fail and the one whose
failure is hardest to recover.

3. Lack of uninterrupted resumption after network failure:
: Where a session must be re-established, the cost is paid again in full.
Mechanisms that reduce that cost exist, and section 5 sets out why they
are not sufficient for the environments described here.

4. Forward secrecy and post-compromise security under intermittent connectivity:
: An association that never rekeys cannot recover from compromise. An
association that can only rekey interactively cannot rekey when the peer
is unreachable, which in these environments is much of the time. Both
properties therefore depend on updates that either peer can initiate
independently.

5. Per device update management:
: Network links today rekey on an all-or-nothing basis. With asynchronous
updates, a network manager can set the update frequency of each device
according to its power and capability, and schedule updates so they do
not disrupt workloads. Not disrupting workloads matters both to
post-quantum adoption and to network management.

# Terminology

Continuous Key Agreement protocol:
: A cryptographic protocol with three properties: asynchronous design,
propose and commit actions, and ratcheting.

Forward secrecy:
: Compromise of current key material does not expose traffic already
sent.

Post-compromise security:
: An association can recover after a compromise, so that an adversary
who obtains key material does not retain access indefinitely.

Post-quantum cryptography:
: Cryptography secure against an adversary with a cryptanalytically
relevant quantum computer.

Amortization of post-quantum cost:
: Spreading the cost of post-quantum key establishment across the life
of an association through many small updates, rather than paying it in
full at each handshake.

Bundle Protocol:
: The store and forward protocol used in delay tolerant networking.

Zero round trip time resumption:
: A mechanism that allows application data to be sent with the first
flight of a resumed connection, at a cost in replay protection.

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
management now flows through space connections. Public safety, health
and financial transactions are all high value targets, and they motivate
attacks against space communications.

Two characteristics distinguish this environment. Propagation delay
makes each round trip expensive in wall clock time. The primary source
at section 11 records round trip times on the order of twenty to two
hundred and fifty milliseconds in low earth and geostationary orbit,
five to fourteen seconds for lunar communications, and between just
under one minute and twenty three minutes between Earth and Mars
depending on orbital positions. It also records that delay tolerant
protocols begin tooutperform IP once round trip times exceed roughly
two hundred milliseconds. At the upper end of that range an interactive
handshake is impractical rather than merely slow.

Contact windows bound the time available. An exchange that cannot
complete inside a window does not complete late; it fails and is retried
at the next window. Compounding this, post-quantum key material and
signatures are larger than their classical equivalents, so the exchange
that must fit inside the window is the one that has grown.

Store and forward operation follows from both. The Delay Tolerant
Networking working group is progressing key agreement for Bundle
Protocol Security, which is independent evidence that this case is real
and that a constituency outside this work holds it.

## Internet of Thing and Operational Technology Systems

Constrained radio links carry small payloads, and devices on them are
frequently power limited and duty cycled. Post-quantum key material and
signatures are large relative to those payloads, so a handshake is not
a fixed cost paid once at session establishment. On a link that
re-establishes often it is a recurring charge against the mission
traffic.

PQC keys, which are large by classical standards, exacerbate this
problem. For example, in a sensor network with many intermediaries where
sensors wake up and decide they want a new PQC keyshare and to start
sending data immediately, waiting for intermediaries to respond is
non-ideal. Further, any ability to amortize the cost of PQC over
multiple connections is ideal.

<aside markdown="block">
Disputed. It has been observed that implementations in this space could
use symmetric keys instead, particularly if signatures are also removed.
That is fair. The case stands where key distribution at scale, or
recovery from compromise, makes a purely symmetric approach unworkable,
and it should be argued on those grounds rather than on payload size
alone.
</aside>

## Virtual Private Networks

Long lived associations between fixed endpoints must rekey without
re-establishment, both to bound the value of any single compromise and
because re-establishment costs more than some links can bear.

The cost of returning after an absence has been measured. In a two
party group, a member that has missed commits pays roughly one
millisecond and one kilobyte per missed commit to reach the current
epoch, measured at five, ten, twenty, fifty and one hundred missed
commits. The cost is therefore linear in the length of the absence
rather than fixed, which is a real constraint, and it is still
materially cheaper than re-establishing a session and paying the
post-quantum handshake again.

<aside markdown="block">
Disputed. Session resumption with connection identifiers addresses part
of this, and the objection is expected. The case stands only where
resumption does not preserve the security properties across an outage
of operational length, and section 5 addresses why that is so.
</aside>

## Unidirectional Communications

Radio transmissions inherently reveal the location of the sender;
returning delivery acknowledgements would be enough to divulge the
sender's location. For use cases where privacy is highly sensitive,
unidirectional communications, in receive only mode, is a requirement.
A CKA protocol provides the best defense.

<aside markdown="block">
Disputed. In the case of unidirectional communications, the sender does
not know whether packets were delivered so there is a question about the
amount of survivability needed for lost key updates. CKA (Continuous Key
Agreement) protocols are probably the best you can get in these
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

QUIC supports stateful connections using session resumption and zero
round trip time resumption, and these sacrifice security for efficiency
by reusing previous connection state. The QUIC designers have advised
disabling these features because of privacy concerns and replay
vulnerabilities, which makes them ill-advised as a means of addressing
latency and handshake overhead in the environments described here.
Despite those warnings, early testing of QUIC in space systems has
specifically examined the zero round trip time option, given the latency
cost of secure session establishment. QUIC also does not offer
post-compromise security once keys are lost, including in its session
resumption option.

Neither QUIC nor TLS provides a means of amortizing the computational
and bandwidth cost of post-quantum algorithms across a long-lived
association. Each establishment pays that cost in full.

Two points of maturity are worth recording, because the proposal is
sometimes read as untested. A variant replacing the QUIC handshake with
a continuous key agreement protocol has been designed, implemented as a
prototype, benchmarked, and analyzed in a formal cryptographic model
with a security proof. Separately, an integration with Bundle Protocol
Security has been implemented against the Interplanetary Overlay
Network and Bundle Protocol Security reference implementations, with
the source published, and its costs measured against group size. Both
are cited at section 11.

Those implementations also qualify a claim that is often made for this
approach. The theoretical scaling advantage over pairwise handshakes is
logarithmic in group size, and the measured behavior is generally
linear, with logarithmic scaling appearing only at ideal group sizes.
The advantage over repeated handshakes remains, and it is smaller than
the asymptotic figure suggests. This document states the measured
position rather than the theoretical one.

# Basic Requirements

The wording is preserved from the presentations at IETF 125 and IETF
126.

* The key management MUST: Support Layer 3 and Layer 4. Asynchronous
key updates. Post-quantum cryptography. Forward secrecy.
Post-compromise security. Protocol formal analysis.

* The key management SHOULD: Asynchronous communication. Support groups
and peer to peer protocols.

A note on the groups item, since this document is restricted to the two
party case: the item is carried from the primary source, which lists
scalability to groups of devices for Bundle Protocol use specifically.
The two positions are not contradictory, and section 7 gives the reason
the two party case is pursued here.

Expressed in terms of the environments at section 4, a Continuous Key
Agreement protocol meeting these requirements needs to:

* support high latency networks with the minimum practicable number of
round trips

* allow either peer to initiate a key update independently, at regular
or predetermined intervals

* allow the parties to update at different rates, so the functionality
and risk profile of each device can be adjusted independently

* allow the update frequency of each device to be set according to its
power and capability, without disrupting workloads

* support keying at either or both the network and the transport layer

* support post-quantum cryptography with amortization options

* ensure current session keys are protected if previous session keys are
compromised, which is forward secrecy

* such a protocol is not required to be stateless, because unlike
Internet use cases there is no risk of exhaustion from very large
numbers of connection establishments, and stateful protocols may
therefore offer better security or functionality. The environments
described at section 4 involve relatively few paired identities with
associations that may persist for years, which is what makes stateful
operation available at all.

# Security Considerations

This document is entirely about security. It is a problem statement,
and the security considerations for specific solutions will be discussed
in solutions documents.

As stated at section 1, this document concerns two party use cases
rather than groups. That scope is deliberate. A mechanism that admits a
third party to a two party association raises concerns about
surveillance capability which are separate from, and likely to attract
more opposition than, the engineering questions raised here.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
