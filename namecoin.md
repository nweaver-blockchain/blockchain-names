---
layout: base
---

# Namecoin

**Name-Service Concept:** Namecoin is a cryptocurrency (NMC), derived
from Bitcoin, designed as a generic key/value store associating global
flat key names with 520B data records.  Its primary purported usage is
for an alternate DNS top level domain (.bit), designed to resist
censorship but it could be used to store other data as well.  It is a
single, global, flat namespace with a first-registered/first-owned
model.

**Development History and Organizational Model:** Namecoin was
initially founded in 2011 as a fork of the Bitcoin codebase.  The
insight was that the Bitcoin could be used as a more generic key/value
store but that to implement this on the Bitcoin blockchain would cause
resource contention: Bitcoin as a system has a limited capacity to add
records, and there were natural concerns that Bitcoin’s monetary
transactions would compete with the use as a public global datastore.

**Name Registration Method:** Namecoin does not support a hierarchical
namespace.  Instead it is a single, global key-value datastore.  To
distinguish between the ‘key’ in key-value datastore and the
cryptographic key used to control the cryptocurrency itself the
Namecoin software uses “name” to identify the datastore key.

If a name doesn’t exist anyone can register that name by generating a
sequence of Namecoin transactions.  The first transaction publishes a
salted hash of the name, so H(name || salt), as a fee-only Namecoin
transaction.  After the Namecoin system generates at least 12 blocks
(roughly 2 hours in practice), the individual wishing to register the
name now does a transaction that costs 0.01 namecoin in addition to
the fee revealing both the name they wish to register and the
associated salt they used in the hash function.
 
This 0.01 Namecoin is now converted to a special cryptographic token
that indicates ownership of the name.  The reason for this two-phase
structure is to prevent another party from front-running possible name
registration: someone else wishing to gain control of the newly
registered name would need to rewrite at least 12 blocks of
transaction history, which in practice is very difficult due to the
merged-mining model that Namecoin employs.

One a user owns a name they can update the associated value, a block
of opaque data up to 520B long, by performing a fee-only Namecoin
transaction.  The owner of the name can also perform an atomic swap
with another user of Namecoin, where the owner transfers control of
the name in return for some amount of Namecoin, enabling users to sell
names to others.

Names need to be maintained or else they expire.  To maintain a name a
user must perform a fee-only Namecoin transaction.  If a name is not
updated for 31,968 blocks (roughly 222 days) it is deemed
semi-expired.  The owner of the name can reclaim it by doing a
fee-only update but otherwise the name does not resolve for the
purpose of the name-value datastore.  After a further 4,032 blocks
(roughly 28 days) the name expires and anyone can register it again.
This ensures that any lookup of a name only needs to examine the past
year worth of Namecoin updates.

The primary intended use for Namecoin is to provide DNS-style mappings
under a new top level domain (.bit), but it can be used for other
purposes.  A Namecoin name representing a domain entry is formatted as
“d/NAME”, with NAME matching the ASCII rules for domain names,
including support for punycode (names starting with ‘xn–’ to represent
Unicode names).

The value format for DNS-style names is JSON encoded ASCII text of at
most 520B long.  This JSON encoding format supports direct name to IP
mappings (equivalent to A and AAAA), aliases (equivalent to CNAME), NS
(nameserver) and DS (Delegated Signer DNSSEC) entries to delegate to a
conventional nameserver for resolution, TXT records, and “dehydrated”
TLS certificates that effectively consist of a bare TLS public key and
a small amount of associated data.

Due to the significant limit of 520B per entry there is also an
include operation that can include by reference up to 3 other names,
allowing the total JSON data to occupy roughly 2 kB after excluding
the overhead of the multiple include entries.

Not only is this storage format significantly limited in capacity but
it is also less efficient than a binary format.  An IPv4 address like
`192.168.3.2` requires just 4 bytes in a binary format but 13 bytes as
JSON encoded ASCII (as the encoding also requires the quotation
marks).  Similarly, DS records use base64 encoding, representing a 33%
overhead.

**Name Resolution Process:** To resolve a Namecoin name a resolver must
have access to the history of all Namecoin transactions over the past
year, either with a “full node” that maintains a complete history of
transactions or a “SPV” node (Simple Payment Verification) that
maintains just a copy of the headers and then queries the central P2P
network for actual data.

Actual resolvers for Namecoin are limited.  The OpenNIC alternative
DNS root used to implement a resolver for `.bit`, but they ceased
operating it in 2019 due to conflicts with the Namecoin maintainers,
and for philosophical reasons there was no attempt by the Namecoin
developers to encourage ISPs to add support to their recursive
resolvers.

Instead users either need to install alternate DNS resolution software
on their host computer that implements a full resolver or a browser
extension.  The host computer software, `ncdns`, uses an existing DNS
stub/recursive resolver (`dnssec-trigger`) and an RPC client to access
information about the Namecoin network from a copy of the Namecoin
peer to peer client.  There exists a Windows installer package but for
other OSs the user must install and configure the components manually.

`ncdns` has a particular peculiarities in its resolution: it will not
just resolve example.bit as a Namecoin name, but will also resolve
example.bit.fubar.com as example.bit.  It is unclear why this design
decision was made but it prevents ncdns’s resolutions from acting as a
strict superset of the DNS system.

For those not wishing to replace their DNS resolver there exists the
Peername browser extension.  This extension supports `.bit` (for
Namecoin) as well as `.emc`, `.coin`, `.lib`, and `.bazar` from EmerCoin
(another cryptocurrency based naming system).  Due to the browser
limitations users must be sure to either input a trailing slash
(example.bit/) or an explicit http (http://example.bit) to prevent the
browser from interpreting a non-recognized TLD as a search request.

There is also no indication of cache lifespan in Namecoin.  It is
assumed that any resolver must have a connection to the Namecoin peer
to peer network and will always use the latest mapping of name to
value.  Using the parameters of a 1 MB block-size and 10 minutes per
block inherited from Bitcoin, such a resolver would need to receive up
to 13 kbps continuously to ensure it isn’t operating on outdated data
although the total data storage required would be at most 50 GB since
Namecoin records need to be updated every year or else they expire.

**Dispute Resolution:** There exists no dispute mechanism.  This is argued
as a feature by the developers (names are “uncensorable”), but this
means that there can be no method to remove problematic domains,
resolve trademark disputes, nor even recover from cryptographic
errors.

The ownership of domains themselves is also pseudonymous, only
referred to by cryptocurrency address and the IP addresses which
records point to.  This means it can be difficult or impossible to
discover the actual identity of an owner of a problematic domain.

**Lookup Privacy:** The use of local data and the P2P network for
resolving Namecoin names limits visibility by third parties.  However
this privacy requires that the local Namecoin resolver maintain a copy
of a year’s worth of all data.  An SPV-based Namecoin resolver will
need to query the P2P network for records when the actual data is not
available but this will still be only visible to a single full node in
the network.

Note however that DNS privacy in this manner is somewhat limited, as
the use of a DNS record to contract a host is visible to the user’s
ISP.

**Data Update Mechanism:** Data in the Namecoin blockchain is updated
by performing a cryptocurrency transaction that includes a new JSON
blob.  The cost of this update is simply the transaction fee which is
currently a fraction of a penny.  However, this low fee only is in
place as long as the network is not used on a regular basis.

With a data update of roughly 600B including overhead, a block size of
1 MB, and a block update rate of 10 minutes, the current Namecoin
network can support only approximately 3 updates per second.  If the
network demand ever exceeded 3 updates per second the result would be
a fee spiral as there is an inelastic supply of space in each block
which is resolved by a fee-auction.

Such fee spirals have regularly made cryptocurrencies too expensive to
use; as this is written the Bitcoin network is experiencing such a fee
spiral due to the addition of “Ordinals” and “BRC20” tokens, a method
of bringing Ethereum like functionality to the Bitcoin blockchain.

**Name Registration Renewal:** After a year the owner of the name needs to
update the associated value or the name expires due to inactivity,
although the cost is currently near trivial, a single 0 NMC + mining
fee transaction is sufficient to maintain the registration.  Failing
to renew in a year's time will result in the name being removed from
use and becoming available for registration.

**Security of Name to Data Mappings:** As long as the domain owner
maintains control of their private key the name system security is
effectively equivalent to modern DNSSEC, just that the DS record for a
TLD is distributed by the Namecoin network rather than from the root
if data is delegated.  But unlike DNSSEC there are cryptographic
failures that can render a domain permanently unrecoverable.

Namecoin users are vulnerable to catastrophic system compromise like
almost every other cryptocurrency.  An attacker who gains a target’s
private key can both remove all the NMC tokens and can transfer the
name to the attacker, permanently stealing from the target the ability
to control or update the name.

Similarly, if a domain owner loses their private key there is no way
to update the information stored in the Namecoin blockchain and, after
a year with no updates, the registration will automatically lapse.

With no dispute resolution it is impossible for the target to reclaim
control of a domain.

**Underlying Cryptocurrency Security:** The Namecoin cryptocurrency,
although a low-value proof of work cryptocurrency, uses “merged
mining” with the Bitcoin blockchain.  In merged mining a Bitcoin miner
can simultaneously mine Namecoin.  So although the profit for mining
Namecoin is near trivia (roughly $1200/day for all Namecoin miners,
compared with about $18M/day for Bitcoin), the actual hash-rate
protecting Namecoin from cryptocurrency-related double-spending
attacks is substantially more.  Since the cost for the miner to
support merged mining is effectively $0, many mining pools will mine
Namecoin alongside Bitcoin just to obtain an extra 0.007% profit.

However, this does mean that Namecoin is dependent on Bitcoin’s near
criminal levels of energy consumption for securing the system.
Bitcoin currency consumes as much power as a medium-sized country.

**Security against Rollback Attacks:** A double-spend attack is effectively a rollback attack, allowing an attacker to rollback history and then replay it in a different manner.  Because Namecoin uses merged mining, a rollback attack is effectively impossible unless the attacker is willing to spend many millions of dollars.

**Record Enumeration:** The records themselves are in cleartext, allowing
easy enumeration of all names.

**Adoption Metrics:** Namecoin’s usage is purely speculative.  Although
numerous names were registered, effectively none are used for any
purpose.  The entire Namecoin network is effectively idle even for the
purpose of cryptocurrency speculation, with most Namecoin blocks
containing no transactions.

**Overall Suitability:** Namecoin has several limitations that may
prove problematic if it seeks to act as a significant TLD.  There
exists no dispute resolution service to moderate disputes or recover
from errors such as lost or stolen cryptographic keys.  The system is
a flat namespace resulting in a global contention for names.  The
Namecoin developers appear to be uninterested in integrating into the
normal DNS infrastructure.  Finally the system can only process
approximately 3 updates per second before the cost to update would
spiral out of control due to the global limit of block creation,
limiting the number of updates possible.

