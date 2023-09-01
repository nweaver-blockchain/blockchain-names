---
layout: base
---

# Handshake

**Name-Service Concept:** Handshake is a cryptocurrency (HNS)[1],
derived from Bitcoin, designed to purportedly replace the DNS root and
TLD (Top Level Domain) by providing a mechanism to map names to DNS
infrastructure.  This mapping is a flat namespace that represents an
alternative infrastructure for top level domains.

**Development History and Organizational Structure:** Handshake purports
to have no formal organizational structure, but the reality is that
the system was founded by two individuals, Joseph Poon and Andrew Lee,
and there is a small core of developers who actually work on the
system.  Control of the system is largely dictated by access to the
master Github repositories which contain the official version of all
code for the cryptocurrency.

When created, the Handshake group created a “pre-mine” of 1.36 billion
HNS tokens.  7.5% (103 million HNS) were given to the creators and
7.5% were sold for $10.3 million to a group of investors.  Most of the
rest of the premine will never be claimed, which means that the
initial cryptocurrency allocation was almost entirely to the insiders
who created the cryptocurrency and the investor group who funded the
launch.

**Name Registration Method:** Handshake does not support a hierarchical
namespace, instead it purports only to register entirely new top level
domains, such as `.web3` or `.xn--ls8h` (aka `.💩`), with full punycode
support for names.  When someone wishes to register an unclaimed name
they trigger an auction which is open for 5 days.

Participants bid by locking up an amount of the Handshake
cryptocurrency in a transaction which combines both the actual bid and
an additional blind value.  For example, a bid could consist of 10 HNS
with a blinding amount of 25 HNS.  The bid would then be made public
as a “bid + blind” of 35 HNS.  After the bidding process is complete
every participant reveals their bid and blind over the next 10 days.

The reveal process returns all bidders’ blind values and all but the
winning bidder’s bid (less the cryptocurrency transaction fees
inherent in all transactions).  The winning bidder’s bid is then split
into two pieces, the amount of the second highest bid which is
“burned” (removed from circulation) and the remaining amount
returned to the bidder.  The intent is to create a sealed bid Vickery
auction, where the bids are confidential but to mitigate the “winner’s
curse” phenomenon somewhat the actual winning bidder only pays the
amount of the second highest bid.  This attempt is incomplete as
although the bid is not visible the bid + blind values are revealed to
all others during the course of the auction.

The existing TLDs (`.com`, `.net`, etc) were reserved in the Handshake
namespace and do not resolve unless the TLD operator wishes to claim
the name by publishing a DNSSEC proof and performing a fee-only
cryptocurrency transaction.  Similarly, top level names were reserved
for the Alexa top 100k domains (e.g. `.google`) which can also be
claimed by publishing a DNSSEC proof.

**Name Resolution Method:** The owner of the domain name can introduce
DNS-related records into the Handshake blockchain through a HNS
cryptocurrency transaction.  The supported resource records are DS
(Delegated Signer), NS (Nameserver name), GLUE4 (an A record only for
a nameserver), GLUE6 (an AAAA record only for a nameserver), SYNTH4 (a
combined NS/A record which identifies a name server by IPv4 address),
SYNTH6 (a combined NS/AAAA record), and a TXT record.  The total
amount of data associated with a name is limited to 512B.

As a consequence the Handshake network does not replace the need for a
set of authoritative DNS servers for the new domain, instead it only
serves to replace the information that the root might provide about
TLD DNS servers.

The Handshake client consists of a specialized SPV (Simplified Payment
Verification) node coupled to `libunbound`.  `libunbound` is set to use
the Handshake resolver as the DNS root server (`.`), and the local
resolver’s key is added to the allowed DNSKEY set for the root to
enable cryptographic authentication of data provided by the SPV node.

An SPV node is a cryptocurrency node that only stores block headers
and, only on demand, queries the peer to peer network to obtain data
for particular blocks.  This SPV node will look up the targeted top
level domain and, if valid, query the larger P2P network to get the
appropriate information, validate the resulting records, and return
the NS, DS (if any), and appropriate A/AAAA records to libunbound.
The SPV node can synthesize DNSSEC signatures for the responses as the
SPV’s key is trusted by libunbound.

If the name is not in the P2P network it will instead query the root
for the appropriate NS, glue, and DS records and forward it to
libunbound, allowing libunbound to otherwise work unchanged and making
the Handshake namespace a strict superset of the conventional DNS
namespace at the time when Handshake was released.  The rest of the
DNS resolution process is conducted by libunbound in the normal
fashion as an iterative DNS lookup.

Although Handshake is not using centralized recursive resolvers
provided by the ISP or others (which results in more load on authority
servers by reducing the effectiveness of caching), any widespread
adoption of Handshake could mitigate this problem by using a
Handshake-compatible recursive resolver instead of performing the
lookup on the client.

**Dispute Resolution:** There exists no dispute mechanism.  This is argued
as a feature by the developers (names are “uncensorable”), but this
means that there can be no method to remove problematic domains,
resolve trademark disputes, nor even recover from cryptographic
errors.

The ownership of domains themselves is also pseudonymous, only
referred to by cryptocurrency address and the IP addresses which host
the actual DNS resolution for a domain.  This means it can be
difficult or impossible to discover the actual identity of an owner of
a problematic domain.

**Lookup Privacy:** The use of local data and the P2P network for
resolving the Handshake top level domains limits visibility by third
parties.  The SPV node will not query the P2P network if the
appropriate records are already in its local cache and, if not, will
query an arbitrary node in the full P2P network to obtain the
information, limiting the number of third parties able to see the
domain the user is querying.  The subsequent query to the
authoritative DNS server is privacy equivalent to the current DNS
system.

**Data Update Mechanism:** Data in the Handshake blockchain is updated
by performing a cryptocurrency transaction that includes a new set of
resource records.  The cost of this update is simply the transaction
fee which is currently a fraction of a penny.  However, this low fee
only is in place as long as the network is not used on a regular
basis.

With a data update of roughly 500B when including overhead, a block
size of 1 MB, and a block update rate of 10 minutes, the current
Handshake network can support only approximately 3 updates per second.
If the network demand ever exceeded 3 updates per second the result
would be a fee spiral as there is an inelastic supply of space in each
block which is resolved by a fee-auction.

Such fee spirals have regularly made cryptocurrencies too expensive to
use; as this is written the Bitcoin network is experiencing such a fee
spiral due to the addition of “Ordinals” and “BRC20” tokens, a method
of bringing Ethereum like functionality to the Bitcoin blockchain.

**Name Registration Renewal:** After two years, the owner of the name
needs to renew the transaction although the cost is near trivial, a
single 0 HNS + mining fee transaction is sufficient to maintain the
registration.  Failing to renew in 2 years time will result in the
name being removed from use and becoming available for auction.

**Security of Name to Data Mappings:** As long as the domain owner
maintains control of their private key the name system security is
effectively equivalent to modern DNSSEC, just that the DS record for a
TLD is distributed by the Handshake network rather than from the root.
But unlike DNSSEC there are cryptographic failures that can render a
domain permanently unrecoverable.

Handshake users are vulnerable to catastrophic system compromise like
almost every other cryptocurrency.  An attacker who gains a target’s
private key can both remove all the HNS tokens and can transfer the
name to the attacker, permanently stealing from the target the ability
to control or update the name.

Similarly, if a domain owner loses their private key there is no way
to update the information stored in the Handshake blockchain and,
after two years, the registration will automatically lapse.

With no dispute resolution it is impossible for the target to reclaim
control of a domain.

**Underlying Cryptocurrency Security:** The Handshake cryptocurrency,
although derived from Bitcoin, uses its own separate proof of work
function.  Proof of work attempts to secure the network by generating
partial hash collisions.  Such a partial collision is easy to verify
but the process required generating a large number of candidates
before discovering a partial collision.  Any attempt to create an
alternate version of history needs to conduct effectively the same
amount of work.  This hash function, based on Blake2b and SHA3, is
unique among cryptocurrencies.

The Handshake cryptocurrency is a very low value cryptocurrency: a
thinly traded, low value cryptocurrency.  Indeed, only a handful of
offshore cryptocurrency exchanges support buying or selling HNS.  Such
cryptocurrencies, when they use their own proof of work mechanism, are
inherently vulnerable to “double spending” attacks.

In a double spending attack, the attacker first transfers the
cryptocurrency to a recipient and can then undo the transaction for a
modest amount of money by recreating the transaction history while
substituting a different spending transaction.  Such attacks are
economically viable when the amount of effort used to protect the
network is less than the amount an attacker can gain from the attack.

The Handshake block reward is now 1000 HNS, the block creation rate is
once every 10 minutes.  At the current price for Handshake ($.03) the
block reward is $30.  In equilibrium the cost of mining a block tends
to approach the block reward.  As a consequence this network is
protected by effectively $4000 per day of effort spent.  Thus an
attacker willing to spend more than $4000 can rewrite the past day of
history to undo a transaction as part of a double-spending attack.

Additionally, 2/3rds of the mining is performed by F2Pool.
Cryptocurrencies where a single entity controls half of the available
hash power must be considered insecure against double spending as the
single mining pool can execute complete control over what transactions
are allowed or excluded from the cryptocurrency and can rewrite
history effectively at will.

**Security against Rollback Attacks:** A double-spend attack is
effectively a rollback attack, allowing an attacker to rollback
history and then replay it in a different manner.

**Adoption Metrics:** Handshake’s usage is purely speculative.  Although
numerous names are registered, most were registered for only a few HNS
which is a value measured in pennies.

**Record Enumeration:**  All records are publicly visible.

**Overall Suitability:** Handshake has several limitations that may
prove problematic if it seeks to replace the DNS root and TLD system.
There exists no dispute resolution service to moderate disputes or
recover from errors such as lost or stolen cryptographic keys.  The
system is a flat namespace resulting in a global contention for names.
The underlying cryptocurrency itself can’t be considered secure by
cryptocurrency standards due to the low effort for the proof of work
employed.  Finally the system can only process approximately 3 updates
per second before the cost to update would spiral out of control due
to the global limit of block creation, limiting the number of updates
possible.

