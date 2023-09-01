---
layout: base
---

# Blockchain Naming Scheme:
Ethereum Name Service

**Name-Service Concept:** The [Ethereum Naming Service
(ENS)](https://ens.domains/) is a service that operates on top of the
Ethereum blockchain platform.  Although a naming service, it is
intended primarily not to map names to computers but instead names to
data references such as cryptocurrency wallet addresses or IPFS
(Inter-Planetary File System) identifiers.  Conceptually there are
primary domains, like `.eth`, that represent new registrations but
there is also support for existing domain holders to register their
existing names using a DNSSEC verification method for some top level
domains like `.xyz`.  Two recently added top level domains (`.kred`
and `.luxe`) have custom integrations with ENS.

**Name Resolution Process:** Systems that wish to resolve ENS names can
take one of two approaches: they can either directly query a local
copy of the Ethereum blockchain (requiring a full node which requires
over 1 TB of storage and a 25 Mbps connection to maintain
synchronized) or rely on a third party API service that conducts such
a query.  In either case the objective is to lookup records associated
with a name, most commonly cryptocurrency addresses.

For looking up a record internally to Ethereum the domain in question
is hashed (using the Namehash strategy discussed later) and the
storage for the master contract for the domain is read from the
Ethereum blockchain.  That contains a pointer to a registrar contract
which is responsible for authenticated storage of actual records.  The
data storage associated with the registrar contract is then examined
to obtain the desired records of the supported types.

Almost all users however just use one of several RESTful API providers
that when queried for a name provide either some or all of the
associated records in a JSON reply.  Their program simply connects to
the provider, authenticates with an authentication token, and then
conducts the query with the response record returned to the querying
program.

**Ethereum’s Computational Model:** All ENS records are stored and updated
using the Ethereum virtual machine (EVM).  Thus this imposes limits on
how ENS can work in practice as all ENS updates involve EVM
transactions, so it is critical to discuss how Ethereum works in
theory and practice.

Ethereum, unlike previous cryptocurrencies, implements a deterministic
virtual machine that executes programs that are stored in Ethereum’s
public ledger when transacting in the cryptocurrency.  Reading the
ledger outside the EVM has no cost, but executing a transaction that
will update the ledger is charged ‘gas’ for each instruction executed
by the virtual machine needed to complete the transaction.

The model of computation requires that someone who wishes to make a
transaction needs to provide a gas fee calculated as the number of
units times the cost-per-unit.  If the computation completes the user
is charged for the amount of gas used but if the computation fails to
terminate before the gas is exhausted all the gas fees are forfeited.

The gas fee is split into two parts, a fixed amount that is “burned”
(removed from circulation) and a variable amount that goes to the
validator which wins the cryptocurrency lottery and successfully
creates the block, creating an auction dynamic where a user can bid
higher in attempt to get their transaction included sooner under
circumstances of high demand.

The fixed portion of the gas fee itself dynamically responds to
demand.  Each Ethereum block (created roughly every 10 seconds) can
contain transactions consuming at most 30 million units of gas.  But
if the amount of gas used is over 15 million, the base portion of the
per-gas transaction fee grows exponentially at 12.5% per block until
the Ethereum blocks are reduced to using only 15 million gas.

For purposes of this document we will generally assume that 100,000
gas costs $5, which is a reasonable approximation of the steady-state
cost, but any actual usage needs to consider that the cost can
increase exponentially seemingly without warning: with the 12.5%
increase per filled block the gas price can jump 10 fold in the space
of just over 3 minutes when facing a surge of demand.  These price
spikes can easily exceed 10x in practice and the rapidity of increase
can occur almost without warning.

As a computational fabric Ethereum is not very significant.  A single
256b add operation consumes 3 gas, so the 1.5 million gas/second can,
at most, compute 500,000 256b adds per second yet costs $75 a second
to use.  In comparison a <$75 Raspberry Pi 4, with its 1.5 GHz quad
core superscalar ARM processor, can implement a 256b addition using 4,
64b adds and thus has a theoretical maximum performance of 3 billion
256b adds/second: over 4 orders of magnitude more powerful.

Likewise storage is similarly expensive and is implemented as a
key/value store using 256b keys and 256b data blocks.  Storing a 256b
block of data at a 256b key into the Ethereum ledger uses a complex
formula, but in the end writing a block of data to a new key requires
slightly more than 20,000 gas while changing an existing key requires
5000 gas.  Thus storing a new KB of data into the Ethereum ledger
requires storing 640,000 gas, or roughly $30, while updating this data
requires slightly less than $10.  Again, these represent normal costs
but high demand can cause an exponential increase in the transaction
fees.

In building Ethereum applications it is thus critical to limit the
amount of gas used whenever possible, as any update to Ethereum’s
global state is significantly expensive which means although the
Ethereum “smart contracts” represent general-purpose programs, it can
be a substantial cost to perform any actual computations in this
fabric.

In contrast, reading the global state of Ethereum is free from the
outside since it is a single global ledger of data.  So although
reading the data is also a program, the read operations can be invoked
without executing data into the global ledger and its associated
costs.  API vendors however may charge per query to provide easier
access to such data.

This also means that Ethereum applications should, whenever possible,
read data outside the actual execution of a smart contract as reading
the data from the blockchain in the smart contract itself incurs gas
fees (at 2100 gas per 256b value, or roughly $0.10) that do not occur
if the data is read using an external program and the resulting
information used in invoking the target smart contract.

But there is a complication with such cost-saving behavior: Ethereum
is an innately hostile computational fabric.

Within a single block the different programs are considered to be
implemented sequentially, but from the view of the block creator it is
the block, not the individual programs, that are the atomic unit of
operation.  This has led to the rise of MEV, or Miner/Maximal
Extracted Value.  In MEV extraction a miner constructs the block,
using additional private transactions, in order to gain additional
revenue.  An example of MEV exploitation might be the miner
automatically front-running a user trading on a decentralized exchange
by inserting trades around the included transaction.

A related problem is that there are numerous actors who will both
manually and automatically exploit flaws in Ethereum programs in an
attempt to gain revenue.  As a consequence the Ethereum blockchain is
a highly malicious environment and code that runs on Ethereum needs to
assume it is under conditions where any vulnerability that could be
profitably exploited will be.  Thus an executing Ethereum smart
contract may need to rely on reading the data from the blockchain
internally to prevent time-of-check to time-of-use vulnerabilities.

Finally, although the Ethereum blockchain itself is “decentralized”,
almost all Ethereum applications (called Dapps or Web3 apps) rely
heavily on centralized and trusted intermediaries.  Simply reading the
Ethereum blockchain to extract relevant information requires
maintaining a 650 GB pool of data and needs to be updated continuously
to avoid going out of sync.  As such almost no Ethereum application
relies on directly reading this data.

Instead, Ethereum applications use third party services provided by
centralized providers like Infura.  This includes browser extensions
like MetaMask that are used to mediate interactions between a user’s
cryptocurrency wallet and Ethereum applications.  Although in theory
Ethereum users can use alternate data providers, in practice Ethereum
users tend to use the default providers when deploying and interacting
with Ethereum applications.

**Development History and Organizational Model:** The Ethereum Name
System is not primarily intended as a DNS replacement but instead as a
mechanism to replace Ethereum public keys as the primary identifier of
a user.  An Ethereum address is the hex representation of the last
120b of the SHA3-256 hash of the ECDSA public key over curve secp256k1
written as 40 hexadecimal digits, so
`0xb794f5ea0ba39494ce839613fffba74279579268` is an example of a raw key.
However, since such keys are easy to introduce errors, Ethereum added
an additional (optional but highly recommended) checksumming procedure
encoded in the key.

This checksum is encoded in the camel case for the hex characters for
a-f.  The public key is hashed again with SHA3-256.  Each letter in
the hexadecimal representation of the public key is capitalized if the
corresponding nibble in the hash is > 7.  Thus with an expected 15
bits of encoding data, a typo should be accepted with probability of
only 1 in 215.  Thus the resulting key presented to the user would be
`0xb794F5eA0ba39494cE839613fffBA74279579268` after applying the checksum
algorithm.  Such addresses are clearly not human-meaningful nor easy
to communicate outside data channels.

ENS offers a service primarily intended to map human-readable names,
such as `vitalik.eth` to cryptocurrency addresses like
`0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`. ENS uses a DNS like
hierarchical structure, with the primary ENS registrar offering names
under the `.eth` top-level namespace, and as they are the same this will
generally refer to ENS as both the generic system and the .eth
namespace operation.

The types natively supported by ENS are blockchain addresses for both
Ethereum and arbitrary cryptocurrencies, content hashes for IPFS or
Swarm (distributed storage systems with the data address itself stored
in multicodec format), a ABI definition for an associated smart
contract, raw SECP256k1 public keys, and key/value text records.  Text
record keys are defined for email addresses, URLs, avatar image URLs,
human-relevant descriptions and notices, keywords, and github and
twitter handles.

Control of the ENS system and the `.eth` namespace is in the hands of 7
individuals who operate a 4 out of 7 multisignature authorization
which is used to update the controlling programs that control the ENS
treasury that receives registration fees for names.  These individuals
are not formally incorporated, thus the responsibility for running ENS
is best described as a general partnership.

Additionally, ENS subsequently issued its own token on top of
Ethereum, also called ENS, that represents a voting and profit share
in the “Decentralized, Autonomous Organization”.  However, as control
of the treasury is maintained by the 7 multisignature holders, it is
questionable how much control the larger DAO can assert over the
system’s operation.

**ENS namehash:** Overall, Ethereum has poor support for strings.  The
storage model for persistent data is just 256b key/256b value pairs.
Thus the initial core transformation necessary is a SHA3-256-based
hashing for names.  After normalizing to lower case and punycode as
performed by DNS, ENS’s namehash is a recursive structure consisting
of the hash of the current set as the hash of the upper level
concatenated with the current label and then hashed, with the hash of
the empty string defined as 256 bits of 0s.

So, for example, the hash of `an.example.eth` is calculated as
H(H(H(`0x0..` || `‘eth’`) || `‘example’` || `‘an’`))).  This scheme results in
a deterministic, hierarchical hash so a name will have not only a
consistent hash for the label itself but a series of higher level
hashes that can enforce a hierarchy.

**ENS registration procedure:** To register a name under the .eth ENS
domain one first sends a commitment request to the registration
contract, which is simply a 256b value constructed by hashing the name
with a random 256b secret.  This transaction is relatively
inexpensive, costing roughly 40,000 gas/$2.  After one minute the user
should then send the registration request consisting of the name, the
contract identifying the name’s owner, the 256b secret, and a validity
time combined with a payment transfer.  This two-part registration
request is to keep a hostile observer from registering the name as the
commitment is meaningless without the 256b random secret.

The `.eth` registrar charges $5 per year to register most names (a value
priced in dollars and provided by an ‘oracle’ service which provides
external data to the Ethereum smart contract) in addition to the gas
fee necessary for the transaction.  Since this costs roughly 300,000
gas the current cost to register a `.eth` name for a year is roughly
$20, although the cost/year can be reduced by registering for a longer
term.  Names less than 7 characters cost more per year and are only
available by auction if they aren’t already registered.

Names need to be renewed before the expiration period by paying
$5/year (a transaction that also costs roughly 100,000 gas/$5).

This only acts to reserve the name in the `.eth` namespace.  To
actually provide data the user must first set a ‘resolver’ contract
associated with the name (40,000 gas/$2), and in that resolver
contract the user can then provide mappings between the user’s domain
and various types of records.  The resolver contract does not charge
to introduce such data but the gas fees can be significant, as
updating a short TXT record is 100,000 gas/$5.

Currently the ens default public resolver contract supports only a few
data types: Ethereum addresses, addresses on other blockchains,
content hashes for IPFS or similar systems, and text records that are
formatted key/value pairs.  There is explicitly no support for
DNS-type records (NS, A, CNAME, etc…) in the standard ENS public
resolver.

The `.eth` registrar treats transferring names like any other
non-fungible token, thus an individual can sell control of their `.eth`
domain to another.  There exists a large number of such domains
available for sale on OpenSea or other NFT marketplaces.

For existing DNS domains with added ENS support such as `.com` and
`.xyz` there are different registrar contracts.  These contracts can
query a DNSSEC-secured record for a domain to specify or update the
ownership of an address by placing a TXT record with an Ethereum
address as `_ens.{domain}`.  Then the registrar contract can be
triggered, creating an ENS record controlled by the ethereum address.
The owner address can then assign other records to the ENS name.
These contracts only incur gas costs to use but, as previously seen,
gas costs can be significant.

**ENS name resolution:** Name resolution, unlike name registration,
only requires reading data.  For name resolution one first queries the
data for the registrar contract of the top level domain for the
address of the resolver contract associated with the namehash of the
target domain.  Then the registrar contract's data is then queried based on
the namehash of the actual name for the particular fields desired.
There exist multiple APIs for accessing this data, including
JavaScript and Go, that rely on third party services to interpret the
data on the Ethereum blockchain.

**Dispute Resolution:** There exists no dispute mechanism within the `.eth`
domain.  This is argued as a feature by the developers (names are
“uncensorable”), but this means that there can be no method to remove
problematic domains, resolve trademark disputes, nor even recover from
cryptographic errors.

The ownership of domains themselves is also pseudonymous, only
referred to by cryptocurrency address.  This means it can be difficult
or impossible to discover the actual identity of an owner of a
problematic domain.

**Security of Name to Data Mappings:** As long as the domain owner
maintains control of their private key the name system security is
effectively equivalent to modern DNSSEC.  But unlike DNSSEC there are
cryptographic failures that can render an .eth domain permanently
unrecoverable.  A domain owner can recover their ENS names when they
are within existing top level domains by recovering control of the DNS
and then updating the DNSSEC entry for the associated ENS name data
but no such mechanism exists for a native ENS domain.

ENS users are vulnerable to catastrophic system compromise like almost
every other cryptocurrency.  An attacker who gains a target’s private
key can both remove all the Ethereum and can transfer any .eth names
to the attacker, permanently stealing from the target the ability to
control or update the name.

Similarly, if a domain owner loses their private key there is no way
to update the information stored in the `.eth` ENS registrar and,
after the renewal period expires, the registration will automatically
lapse.  With no dispute resolution it is impossible for the target to
reclaim control of a domain.

**Data Privacy:** Due to Ethereum acting as a global public ledger and the
API used to register domains, all `.eth` domains are public.  All data
associated with the domain itself is also public: for data associated
with the domain itself (e.g. all records for `example.eth`) they are
directly mappable to the registered domain.  For subdomains
(e.g. `an.example.eth`) it is recorded by namehash so if the name is
predictable both the name and associated data can be known but if the
name is unpredictable only the data fields are public.

Reading this data outside of a smart contract is private except to
whatever API service provider is interpreting the Ethereum blockchain
on behalf of the user, while reading the data within a smart contract
is public as the entire stream of execution of a smart contract is a
public activity.

**Data Update Mechanism:** A transaction to update data in the public
registrar costs roughly 40000 to 100000 gas, depending on the type of
update (cryptocurrency keys or text records).  If the only use of
Ethereum was ENS this would support a global update rate of roughly 15
to 30 names/second before the cost per transaction grows
exponentially.

Unfortunately for those seeking to use ENS, ENS is just one use of the
Ethereum blockchain.  The Ethereum blockchain regularly experiences
fee spirals where the cost per transaction can jump by 10x in the
space of a few minutes.  So although a normal data update may be
expected to cost roughly $5, if there is an urgent need to do an
update at the wrong time it could cost $50 or more due to the
exponentially increasing transaction fees.

**ENS to DNS gateway:** An Ethereum developer, Virgil Griffiths, created a
ENS to DNS gateway called `eth.link`.  Its primary use is not to resolve
names directly, but instead to forward name requests to an IPFS to
Internet gateway, such as the one run by Cloudflare as a centralized
service.  This Cloudflare service itself looks up the IPFS file
locator in the Ethereum blockchain and then acts as a proxy to IPFS to
display the root web page corresponding to the `.eth` domain.
Cloudflare’s IPFS proxy service provides 50 GB of transfer a month
free before it charges the domain owner and the domain owner must
create a Cloudflare account to enable access to their domain.

The `eth.link` DNS gateway itself is currently involved in a legal
dispute.  Virgil Griffiths is currently incarcerated for violations of
US government sanctions against North Korea and, during that time, the
registration for eth.link lapsed and was sold to a third party.
Although currently the domain is back under control of the original
provider, the court case is ongoing.

There are other alternatives, for example the service `eth.limo`
provides an equivalent gateway, looking up IPFS links and acting as an
IPFS to HTTP gateway.  Neither service is formally affiliated with the
group developing ENS.

ENS & DNS combined TLDs: Two top level domains, `.kred` and `.luxe`, offer
integration with ENS as part of their DNS service.

For the `.kred` TLD, a user also obtains an ENS name and an associated
Ethereum token.  This token used to track ownership of the `.kred`
name, and updates to the ENS data associated with this token are
reflected in the DNS data: ENS data can contain a pointer to an IPFS
link to store the DNS records and updates to the IPFS data update the
DNS records.  The `.kred` registrar retains actual control of the DNS
records and can, in the case of a terms of service violation,
forcefully update and change the ENS records as well.  Thus it is
`.kred`, not the user, who in the end has control over the associated
name in both ENS and DNS.

The `.luxe` TLD integration is simpler.  The primary mechanism is
standard DNS registration, but a handful of DNS registrars for `.luxe`
also support a simple Ethereum registration, where the user can
specify an Ethereum address in effectively the same manner they can
specify an A record, a NS record, or other data.  This Ethereum
address is then published to an ENS resolver for the `.luxe` domain.
There is no mechanism for the ENS data to update the DNS data.

**Security against Rollback Attacks:** Ethereum is regarded as a secure
cryptocurrency, and applications on top of Ethereum do benefit from
this security.  In particular a “double spending” attack is
practically impossible without the collusion of a large number of
those holding Ethereum cryptocurrency who act as block producers.

A double-spend attack is effectively a rollback attack, allowing an
attacker to rollback history and then replay it in a different manner.
In ENS a rollback attack is effectively impossible unless the attacker
has already corrupted over 50% of the Ethereum validators.

**Censorship resistance:** ENS’s attempt at censorship resistance may
result in a lack of dispute resolution but it is not absolute.  There
are both the Ethereum validators themselves, which are controlled by a
small cartel, and the centralized services that can and have executed
censoring requests on behalf of the US government, primarily around
Tornado Cash, a money laundering service favored by bad actors
including North Korea.

When the US government sanctions went into effect, Infura and other
central data providers blocked access to the Tornado Cash contracts,
making any external program (such as the Metamask browser extension)
incapable of accessing this data.  And now a plurality of Ethereum
blocks will not include transactions interacting with Tornado Cash.

This includes the ENS records associated with Tornado Cash.  The ENS
domains, `tornadocash.eth` and `tornadocashcommunity.eth` are not
resolved by the `.eth.link` service, but `eth.limo` service will
return HTTP 451 (blocked for legal reasons) for `tornadocash.eth` but
fully supports `tornadocashcommunity.eth` despite the OFAC sanctions.
Some API providers, such as Infura, block the lookup of records
associated with the Tornado Cash ENS domains.

**Record Enumeration:** The names directly registered under `.eth` are
visible as the smart contract itself requires the clear text name in
registration.  Further subdomains are identified only by the namehash,
so only low entropy labels can be enumerated by using a brute-force
search comparable to NSEC-3 record enumeration.

**Overall Suitability:** ENS has several limitations that may prove
problematic if it seeks to act as a significant TLD.  ENS does not
support DNS resource records.  There exists no dispute resolution
service to moderate disputes or recover from errors such as lost or
stolen cryptographic keys.  The ENS developers appear to be
uninterested in integrating into the normal DNS infrastructure, and
the only interfaces are third party services.  Ethereum can only
process approximately 15 to 30 updates per second before the cost to
update would spiral out of control due to the global limit on gas
creation.  The cost to store data in Ethereum is currently roughly $30
per kB under normal conditions.  Finally, Ethereum regularly
experiences price shocks where a $5 operation suddenly costs $50 due
to other usage demands
