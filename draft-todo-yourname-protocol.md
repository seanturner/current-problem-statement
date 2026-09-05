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

informative:
  I-D.ietf-quic-extended-key-update:
  I-D.ietf-tls-extended-key-update:
  RFC7296:
  RFC8446:
  RFC9001:
  RFC9171:
  RFC9420:

...

--- abstract

TLS and QUIC allow either endpoint to initiate a traffic key update, but
their standardized key update mechanisms derive new keys from existing
key state. They do not introduce fresh keying material and therefore cannot
restore security after that state has been compromised. Mechanisms that
are currently being developed for TLS and QUIC to introduce fresh
post-quantum keying material use an interactive exchange.

This document considers mechanisms through which either endpoint can
independently initiate a key update that incorporates fresh randomness and can
provide post-compromise security. The initiator does not wait for a live
response before sending the update, and the peer can process it later. This
property is useful even when both endpoints are normally reachable because it
allows each endpoint to schedule updates according to its constraints.
Intermittent connectivity and long propagation delays make the property more
valuable. Such updates can also distribute post-quantum update costs across a
long-lived connection.

--- middle

# Introduction

TLS 1.3 and QUIC permit either endpoint to initiate a traffic key update
{{RFC8446}} {{RFC9001}}. These updates are useful for limiting the amount of
traffic protected by one traffic secret. However, they apply a key
derivation function to existing key state and do not introduce fresh keying
material. An attacker that obtains the current state can compute the
same subsequent traffic secrets.

Post-compromise security requires a key update that incorporates fresh
randomness unknown to the attacker after the attacker loses access to the
endpoint. The resulting fresh keying material allows the connection to recover
confidentiality. The Extended Key Update proposals for TLS and QUIC introduce
fresh keying material through an interactive exchange
{{I-D.ietf-tls-extended-key-update}}
{{I-D.ietf-quic-extended-key-update}}.

The property considered in this document is that either endpoint can
independently initiate such an update without waiting for a live response from
its peer. The peer can process the update later from compatible predecessor
state. This property is useful even on a reliable network because it allows the
endpoints to use different update schedules. Long propagation delays,
intermittent connectivity, asymmetric bandwidth, power limits, and workload
constraints make independent initiation more valuable.

Connection resumption addresses a different problem. TLS 1.3 resumption
with fresh ephemeral Diffie-Hellman can provide forward secrecy for new
1-RTT application data, while PSK-only resumption and 0-RTT data have weaker
security properties {{RFC8446}}. Resumption does not by itself recover from
compromise if the attacker also obtained the resumption secret. Repeating
connection establishment can also repeat the computation and bandwidth
cost of post-quantum key exchange.

This document concerns two-party key management. It examines continuous
key agreement as a means of introducing fresh keying material through
asynchronous key updates, and it describes the desired properties of a
solution. The resulting keying material can be supplied to security
protocols at the network, transport, or application layer. Group key
agreement is out of scope.

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
: Key agreement that requires a request and a corresponding response before
the initiator can complete the agreement and use the resulting keying
material.

Asynchronous key update:
: A key update that either endpoint can generate without a live exchange with
the peer. The peer can process the update later if it retains compatible
predecessor state. A protocol defines how it handles lost, duplicated,
reordered, stale, and concurrent updates. In this document, an asynchronous
key update incorporates fresh randomness and introduces fresh keying material
that can provide post-compromise security under the stated threat model.

Continuous key agreement (CKA) protocol:
: A key agreement protocol in which the endpoints periodically incorporate
fresh keying material and derive new shared keys over the life of a
connection. A particular CKA protocol can support asynchronous key updates,
but that property is not part of the general definition.

Fresh keying material:
: Secret keying material derived using fresh randomness rather than solely from
the current connection state. To provide post-compromise security, an update
must incorporate randomness that remains unknown to the attacker after the
compromise ends.

Forward secrecy:
: Protection of traffic from compromise of keying material at a later time.
This property depends on deletion of traffic keys and the secret state from
which they can be derived.

Post-compromise security:
: The property that a connection can restore confidentiality after an
attacker has compromised its current key state, the attacker no longer has
access to an endpoint, and a successful key update introduces fresh keying
material that the attacker does not know.

Post-quantum cryptography:
: Cryptography secure against an adversary with a cryptanalytically
relevant quantum computer.

Amortization of initial post-quantum cost:
: Accounting for the fixed cost of initial post-quantum key establishment
over the application traffic carried during a connection. The initial cost
is paid in full, but its relative cost decreases as the connection carries
more traffic.

Distribution of post-quantum update cost:
: Scheduling complete post-quantum key updates at different points during
the life of a connection. This changes when the cost is incurred but does
not necessarily reduce its total computation or bandwidth cost. A protocol
can separately choose to fragment a single update across several messages.

Bundle Protocol:
: The store-and-forward protocol used in delay-tolerant networking
{{RFC9171}}.

0-RTT data:
: Application data sent in the first flight of a resumed connection. TLS
and QUIC do not provide inherent replay protection for 0-RTT data
{{RFC8446}} {{RFC9001}}.


# Problem Statement

An asynchronous network and an asynchronous key update are distinct. The
network property concerns reachability and delivery. The key management
property considered here has two parts: either endpoint can independently
initiate an update without waiting for a live response, and the update
incorporates fresh randomness in a way that can provide post-compromise
security. Some use cases also involve an asynchronous network, but that is not
a prerequisite for using the key update mechanism.

Five problems follow.

1. Post-quantum connection establishment and update cost:
: An initial post-quantum exchange is required and its cost is paid in full.
For a long-lived connection, that fixed cost can be accounted for over more
application traffic. Later post-quantum updates still incur computation and
bandwidth costs, but they can be scheduled throughout the connection rather
than concentrated in repeated authenticated handshakes. Fragmenting a
single update is a separate design choice.

2. Lack of asynchronous updates with fresh keying material:
: The standardized TLS and QUIC key update mechanisms allow unilateral
initiation, but do not introduce fresh keying material. Extended Key
Update mechanisms under development for TLS and QUIC introduce such material
through an interactive exchange
{{I-D.ietf-tls-extended-key-update}}
{{I-D.ietf-quic-extended-key-update}}. On paths subject to contact windows,
long propagation delay, or interruption, completing that exchange can be
difficult.

3. Limits of resumption after network failure:
: Resumption can avoid a full authenticated handshake. Its security depends
on the selected mode and on which secrets an attacker has obtained. Fresh
ephemeral Diffie-Hellman can protect new 1-RTT traffic, but resumption does
not recover security if the attacker retains the resumption secret. A new
connection can also require another post-quantum exchange.

4. Post-compromise security under intermittent connectivity:
: Forward secrecy can be provided by the initial ephemeral key exchange and
secure deletion of old key state. Post-compromise security additionally
requires that after the attacker loses access, the connection must incorporate
fresh keying material unknown to the attacker {{RFC9420}}. If updates that
introduce fresh keying material require an interactive exchange, recovery
cannot begin while the peer is unreachable.

5. Per-endpoint key update management:
: Standardized TLS and QUIC traffic key updates can be initiated by either
endpoint, but do not add fresh keying material. Interactive exchanges that
add fresh keying material couple an initiator's progress to a peer response.
An asynchronous mechanism could let each endpoint schedule such key updates
according to its power, workload, risk, and available network capacity. The
protocol must still define when an update takes effect and how the
endpoints resolve concurrent or missing updates.

# Use Cases

The following use cases motivate different parts of the problem. A solution
does not need to apply to every use case, but it needs to identify the network
and endpoint assumptions under which it provides the relevant properties.

## Space

Everything from banking information to critical infrastructure
management now flows through space communication systems. Public safety,
health and financial transactions are all high value targets, and they
motivate attacks against space communications.

Two characteristics distinguish this environment. Propagation delay
makes each round trip expensive in wall-clock time. Published measurements
report round-trip times on the order of twenty to two
hundred and fifty milliseconds in low earth and geostationary orbit,
five to fourteen seconds for lunar communications, and between just
under one minute and twenty-three minutes between Earth and Mars
depending on orbital positions. They also indicate that delay-tolerant
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
frequently power-limited and duty-cycled. Post-quantum key establishment
can require messages that are large relative to the application payloads
on these links. Repeating that exchange after each loss of connection
consumes airtime and power that would otherwise carry application traffic.

A device that wakes for a short transmission window might be able to send an
asynchronous key update but not remain reachable for the peer's response. The
peer could process the update later. Until that happens, the sender can continue
using an existing key state if its security policy permits. A long-lived
connection also accounts for its initial post-quantum establishment cost over
more application traffic.

Devices provisioned with pairwise symmetric keys might not need public-key
key establishment. This use case applies where key distribution at scale or
recovery from compromise makes that approach unsuitable.

## Virtual Private Networks

VPN connections are often long-lived. Establishing a post-quantum-secure
connection requires an initial post-quantum exchange, whose cost is paid in
full. The relative cost of that exchange decreases as the connection carries
more traffic. After establishment, a CKA protocol can introduce fresh
post-quantum keying material without repeating the complete
authenticated connection-establishment exchange. The updates still consume
bandwidth and computation, but an operator can distribute them throughout the
life of the connection.

IKEv2 already supports Security Association lifetimes, rekeying, concurrent
rekey collision handling, and optional fresh Diffie-Hellman input. Each IKEv2
exchange consists of a request and a response {{RFC7296}}. The additional
property sought here is the ability to generate and transmit an update that
introduces fresh keying material without completing such a live exchange at
update time.

This property can give an operator more control over when and how often each
endpoint initiates an update. For example, a client with limited upload
capacity could receive peer-initiated updates over a less constrained
downstream path and defer its own update until upload capacity is available. The
protocol's recovery claim must state which endpoint was compromised because
peer-initiated and locally initiated updates do not have identical security
effects.

## Unidirectional Communications

An observer can sometimes use radio transmissions to estimate the location
of a transmitter. In such a deployment, a receiver might avoid return traffic
because an acknowledgment would disclose the receiver's activity or location.
This produces a one-way channel rather than an asynchronous two-way channel.

An asynchronous key update can be delivered over that one-way channel without
an acknowledgment, but this does not provide general post-compromise security.
A receive-only endpoint whose state was compromised needs fresh keying material
that the attacker does not know. A send-only endpoint cannot receive a
peer-initiated update. A solution for this use case therefore needs to state the
direction of compromise, the source of fresh entropy, and how it handles lost
updates. Some compromise cases cannot be repaired without eventual return
communication or an out-of-band update.

In unidirectional communication, the sender does not know whether an update
was delivered. A protocol that supports this case needs a bound on retained
sender and receiver state, a rule for skipping missed updates, and a recovery
procedure for state that can no longer be reconciled.

# Limitations of Current Solutions

Extended Key Update mechanisms are being developed to add fresh keying
material to TLS and QUIC
{{I-D.ietf-tls-extended-key-update}}
{{I-D.ietf-quic-extended-key-update}}. They address the lack of
post-compromise recovery in the standardized traffic key update mechanisms.
They require response messages before the endpoints complete an update. They
therefore do not provide the asynchronous update property described in this
document.

QUIC supports resumption, and it permits 0-RTT data on a new connection.
0-RTT data does not have the same replay protection or forward secrecy as
1-RTT data {{RFC8446}} {{RFC9001}}. Resumption with fresh ephemeral
Diffie-Hellman can provide forward secrecy for new 1-RTT traffic. It cannot
provide post-compromise recovery if the attacker obtained and retains the
resumption secret. A deployment also needs to consider the linkability of
resumed connections and the application consequences of replayed 0-RTT data.

The standardized QUIC and TLS key update mechanisms do not introduce
fresh keying material {{RFC8446}} {{RFC9001}}. They can limit the amount
of traffic protected under one traffic secret, but an attacker that knows
the current traffic secret can derive later traffic secrets. Obtaining
post-compromise recovery requires fresh keying material and secure
deletion of the compromised state from which future keys could be derived.

Prototype work demonstrates that this approach is feasible. One
implementation replaces the TLS-based handshake used by QUIC with a CKA
protocol. The implementation has been benchmarked, and its design has been
analyzed in a formal cryptographic model.

# Properties of a Solution

The following properties describe a solution to the problem. They do not
prescribe a particular key agreement protocol.

A solution needs to:

* derive keying material for a two-endpoint security protocol at the network,
transport, or application layer.

* allow an endpoint to generate an update that introduces fresh keying
material without a live response from its peer, and allow the peer to process
that update later from compatible predecessor state.

* support post-quantum key establishment and state whether authentication is
classical, post-quantum, or hybrid.

* authenticate each endpoint and bind every update to the connection, the
initiating endpoint, and the applicable key state.

* provide forward secrecy under an explicit secure-deletion assumption.

* provide post-compromise security after the attacker loses access and a
successful key update introduces fresh keying material that the attacker does
not know.

* prevent downgrade from the post-quantum or hybrid mode selected by policy.

* be accompanied by a formal security analysis.

The use cases in Section 4 add the following properties:

* allow an endpoint to generate and transmit an asynchronous key update
without waiting for a live round trip.

* allow either endpoint to initiate an asynchronous key update independently.

* allow each endpoint to use a separate key update frequency and
schedule, including regular or predetermined intervals, based on its
role, risk profile, power, capabilities, and workload.

* allow a deployment to distribute complete post-quantum key updates
throughout the life of a connection. A solution can also fragment one update
across several messages, but fragmentation is a separate mechanism.

* ensure that compromise of current key state does not reveal traffic
protected using earlier key states, provided the secrets needed to derive
those earlier states have been deleted.

* define how endpoints handle lost, duplicated, reordered, stale, and
concurrent updates.

* detect or recover from state rollback and state divergence, or fail without
reusing keys or accepting unauthenticated state.

* define key confirmation and the point at which each endpoint can use or
delete a key state.

* separate keying material used for different directions, epochs, protocols,
and purposes.

* limit the computation, storage, and bandwidth that unauthenticated or stale
updates can cause.

A solution can retain per-connection state. It needs to specify storage bounds,
state expiration, crash recovery, and the security consequences of restoring
an older state snapshot.

# Security Considerations

A solution needs to state whether compromise covers traffic keys, key schedule
state, update private keys, long-term authentication keys, resumption secrets,
or all endpoint memory. It also needs to state when the attacker is assumed to
lose access. Post-compromise security is impossible while an attacker can read
new endpoint state or learn all fresh keying material. Recovery of traffic
confidentiality does not restore peer authentication if the attacker retains a
long-term authentication key.

Forward secrecy depends on deletion of old traffic keys and any secret state
from which they can be derived. Post-compromise security additionally depends
on incorporating fresh keying material that the attacker does not know. A key
update that only applies a key derivation function to compromised state cannot
provide post-compromise security. Restoring an old snapshot can also restore
compromised key state or cause key reuse.

An asynchronous protocol must authenticate updates and bind them to the correct
connection, endpoint, direction, and key state.

Key confirmation requires care when the sender does not receive an immediate
response. Until confirmation arrives, an endpoint might need to retain multiple
states or continue sending under an older state. A protocol must specify the
security and availability consequences of these choices. It should also limit
the work and storage caused by bogus updates so that key updates do not
create a denial-of-service vector.

Post-quantum key establishment and post-quantum authentication are separate
properties. A solution must state whether it addresses a passive adversary that
records traffic for later decryption, an active quantum-capable adversary, or
both. Hybrid negotiation must bind the selected algorithms and key updates
to authenticated protocol state to prevent downgrade or component stripping.

A resumed connection cannot claim post-compromise recovery if it is based on a
resumption secret that the attacker still knows. A solution needs to specify
how a compromised resumption state is invalidated.

The scope is limited to connections between two endpoints. A relay or delivery
service that only forwards encrypted protocol messages need not learn the
connection keys, as illustrated by the MLS Delivery Service model {{RFC9420}}.
A service that contributes secret keying material, authenticates endpoint
identities, or receives connection keys changes the trust model and requires
separate security and privacy analysis.

Update timing, size, retransmission, and acknowledgment behavior can reveal
endpoint activity and connectivity patterns even when the update contents are
encrypted. Deployments with traffic-analysis concerns need to account for this
metadata.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
