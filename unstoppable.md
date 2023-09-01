---
layout: base
---

# Unstoppable domains


**Name-Service Concept:** The [Unstoppable
Domains](https://unstoppabledomains.com/) system is a service that
operates on top of the Polygon blockchain platform.  Although a naming
service, it is intended primarily not to map names to computers but
instead names to data references such as cryptocurrency wallet
addresses, although it does support DNS type records or IPFS
(Inter-Planetary File System) identifiers.  Conceptually there are a
set of primary domains (currently `.crypto`, `.nft`, `.x`, `.wallet`,
`.polygon`, `.dao`, `.888`, `.zil`, `.blockchain`, and `.bitcoin`)
that Unstoppable Domains supports, and although name transfers and
management occur through the Polygon smart contracts, almost all
interactions are through Unstoppable Domain’s centrally managed and
controlled API.


**Polygon’s Computational Model:** Polygon supports Ethereum’s virtual
machine, albeit with different gas parameters and costs.  Ethereum,
unlike previous cryptocurrencies, implements a deterministic virtual
machine that executes programs that are stored in Ethereum’s public
ledger when transacting in the cryptocurrency.  Reading the ledger has
no cost, but executing a transaction and the associated program is
charged ‘gas’ for each instruction executed by the virtual machine.
Polygon inherits the computational model from Ethereum.

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
demand.  Each Polygon block (created roughly every 2 seconds) can
contain transactions consuming at most 20 million units of gas.  But
if the amount of gas used is over 10 million, the base portion of the
per-gas transaction fee grows exponentially until the Polygon blocks
are reduced to using only 10 million gas.

Fortunately for those using the Polygon blockchain the gas cost itself
is moderately reasonable as Polygon itself is not very valuable:
100,000 gas costs a bit over $.01 (it is actually $.03 at the time of
writing, but we will assume $.01 just to keep the back of the envelope
math easier).  However, the exponentially increasing cost can
regularly spike prices by 2x or more in the space of a minute.

As a computational fabric Polygon is not very significant.  A single
256b add operation consumes 3 gas, so the 5 million gas/second can, at
most, compute 1,700,000 256b adds per second yet costs over $.50 a
second to use.  In comparison a <$75 Raspberry Pi 4, with its 1.5 GHz
quad core superscalar ARM processor, can implement a 256b addition
using 4, 64b adds and thus has a theoretical maximum performance of 3
billion 256b adds/second: 4 orders of magnitude more powerful for just
the cost of less than 3 minutes of computation on the Polygon
blockchain.

Likewise storage is similarly expensive and is implemented as a
key/value store using 256b keys and 256b data blocks.  Storing a 256b
block of data at a 256b key into the Polygon ledger uses a complex
formula, but in the end writing a block of data to a new key requires
slightly more than 20,000 gas while changing an existing key requires
5000 gas.  Thus storing a new KB of data into the Polygon ledger
requires storing 640,000 gas, or roughly $0.06 while updating this KB
of data will cost a little more than a penny.  Again, these represent
normal costs but high demand can cause an exponential increase in the
transaction fees.

But although small per KB, these costs quickly grow well beyond
conventional solutions.  Storing 1 GB of data in Polygon costs at
least $60,000.  In comparison, an Amazon S3 bucket charges $.023 per
gigabyte-month, with data transfer costing $.09 per GB transferred out
of Amazon.  So assuming the GB of data is also downloaded 1000 times a
month, the cost of storing 1 GB of data in Polygon is equivalent to 50
years of Amazon service.  If the data is not used outside of Amazon
(thus not incurring download charges) this 1 GB of data on Polygon is
equivalent to over 10 millenia of Amazon storage.

In building Polygon applications it is thus critical to limit the
amount of gas used whenever possible, as any update to Polygon’s
global state is significantly expensive which means although the
Polygon “smart contracts” represent general-purpose programs, it can
be a substantial cost to perform any actual computations in this
fabric.

In contrast, reading the global state is free from the outside since
it is a single global ledger of data.  So although reading the data is
also a program, the read operations can be invoked without executing
data into the global ledger and its associated costs.

Finally, although the Polygon blockchain claims to be “decentralized”,
almost all Polygon applications (called Dapps or Web3 apps) rely
heavily on centralized and trusted intermediaries.  Simply reading the
Polygon blockchain to extract relevant information requires
maintaining a 1.2 TB pool of data and needs to be updated continuously
to avoid going out of sync.  As such almost no Polygon application
relies on directly reading this data.

Instead, Polygon applications use third party services provided by
centralized providers like Infura.  This includes browser extensions
like MetaMask that are used to mediate interactions between a user’s
cryptocurrency wallet and Polygon applications.  Although in theory
users can use alternate data providers, in practice users tend to use
the default providers when deploying and interacting with blockchain
applications.

**Development History and Organizational Model:** Unstoppable Domains
is a commercial entity with backing from multiple venture capital
firms, and maintains a tight control over the development process.
Although clearly inspired by the Ethereum Name System (ENS) in terms
of structure the actual implementation is independent, and Unstoppable
Domains as a company is largely focused on streamlining the process:
Names are registered using credit cards or other conventional payments
and they offer hosting services that eliminate the need for a user to
deal directly with the underlying cryptocurrencies.  Major venture
firms have invested roughly $65M with a notional valuation of $1B.

**Namehash:** Overall, Ethereum and Ethereum-derived systems have poor
support for strings.  The storage model for persistent data is just
256b key/256b value pairs.  Thus the initial core transformation
necessary is a Keccak-256-based hashing for names.  After normalizing to
lower case and punycode as performed by DNS, ENS’s namehash, which
Unstoppable Domains adopts is a recursive structure consisting of the
hash of the current set as the hash of the upper level concatenated
with the current label and then hashed, with the hash of the empty
string defined as 256 bits of 0s.

So, for example, the hash of `an.unstoppable.crypto` is calculated as
H(H(H(`0x0..` || `crypto`) || `unstoppable` || `an`))).  This scheme
results in a deterministic, hierarchical hash so a name will have not
only a consistent hash for the label itself but a series of higher
level hashes that can enforce a hierarchy.

**Name registration:** Initial name registration is handled entirely
through Unstoppable Domains and their web interface.  The underlying
contracts used for data storage (primarily on Polygon, but also
supporting names stored on Ethereum and Zilliqa) can only be
initialized by Unstoppable Domains, forcing all users to register
through Unstoppable Domains’s web site and pay to register by directly
paying Unstoppable Domains.

The cost that Unstoppable Domains charges is a one-time fee that
depends on the cost of the domain, so for example they currently
request $1000 to register `primates.crypto`, but only $40 to register
`aonecufasonefu.crypto`.  At that point a user can have the new domain’s
control transferred to their cryptocurrency address, or have
Unstoppable Domains manage operation for a fee.

Once a user has control over the domain they can then add key/value
pairs to the domain, or if managed by Unstoppable Domains, use the web
interface to have Unstoppable Domains register it.  For individual
users Unstoppable Domains charges $99/year for managing unlimited
domains, and in the process will pay the gas fees on data update, but
if the user self-hosts they will need to pay the gas fees necessary,
currently a couple of pennies for any data update.

Keys are primarily intended to represent cryptocurrency addresses, but
it can also support DNS data and IPFS (Inter-Planetary-File-System)
record locators, allowing web sites to be “hosted” using Unstoppable
Domains names.

The gas limit in Polygon also imposes a global limit on the update
rate.  If Unstoppable Domains executed approximately 90 updates per
second it would exhaust the capacity of the polygon blockchain.

**Name transfer:** Like ENS, Unstoppable Domains implements control over a
domain as an ERC-721 token (aka a ‘non-fungible token’).  The control
of the token controls the rights to update the associated data with a
name and can be transferred or sold to others either directly or on
third party markets like OpenSea.  The smart contract itself does not
enforce an additional transaction fee beyond the underlying “gas fee”
necessary to transfer a name.

Name resolution: Although in theory a name resolution system could
directly access the data on the various blockchains (Unstoppable
Domains will register across three different blockchains: Ethereum,
Polygon (MATIC), and Zilliqa (ZIL)), resolution is performed almost
entirely through a RESTful API operated by Unstoppable Domains,
allowing a single query to obtain the record set associated with the
name.

The Opera and Brave web browsers use this API to process requests in
the `.crypto` domain, and browser extensions are available for Firefox
and Chrome to support other domains.

**The `.coin` domain:** Unstoppable Domains initially included `.coin`
as a top level domain that customers could use.  However, another
blockchain naming scheme, Emercoin, had previously started using
`.coin` as a top level domain.  After becoming aware of the conflict,
Unstoppable Domains decided to withdraw `.coin` from the names they
register and offer a refund in store-credit to those who had
registered names in that top level domain.  This was primarily as a
way of establishing a ‘first come, first served’ precedent in the
blockchain namespace domains.

Unstoppable Domain’s withdrawal of .coin wasn’t limited to just a
refund, however.  Although customers who did not seek a refund would
still “own” `.coin` domains on the Polygon blockchain, Unstoppable
Domains announced that their resolving APIs would no longer support
.coin, meaning that as a central authority Unstoppable Domains was
able to stop all use of `.coin` “Unstoppable" Domain”.

**Lawsuits:** Unstoppable Domains strongly believes in attempting to
apply trademark protection to their top level domains.  The Handshake
naming system supports arbitrary top level domains, and a user
registered `.wallet` as a top level domain in Handshake and began
selling names underneath it.  Unstoppable Domains sued, arguing
trademark infringement and other claims.  Currently the various cases
are working through the court system.

**Dispute Resolution:** There exists no dispute mechanism within
Unstoppable Domains, but there is potential to implement a dispute
resolution service in practice as discussed in the security section.
The lack of dispute resolution is argued as a feature by the
developers (names are “unstoppable”), but this means that there can be
no method to remove problematic domains, resolve trademark disputes,
nor even recover from cryptographic errors.

The ownership of domains themselves can also be pseudonymous, only
referred to by cryptocurrency address.  For names controlled by
Unstoppable Domain’s they would maintain ownership records but such
records disappear when a name is transferred to a third party.  This
means it can be difficult or impossible to discover the actual
identity of an owner of a problematic domain.

**Security:** As long as the domain owner maintains control of their
private key the internal name system data security is effectively
equivalent to modern DNSSEC.  But unlike DNSSEC there are
cryptographic failures that can render an “Unstoppable Domain”
permanently unrecoverable.

Unstoppable Domain users are vulnerable to catastrophic system
compromise like almost every other cryptocurrency.  An attacker who
gains a target’s private key can both remove all cryptocurrency assets
and can transfer any “Unstoppable Domain” names to the attacker,
permanently stealing from the target the ability to control or update
the name.  Similarly, if a domain owner loses their private key there
is no way to update the information stored in the registrar.

Unstoppable Domains understands the problems with this model.  As such
they also explicitly offer custodial services for a fee where
Unstoppable Domains, not the user, actually controls the cryptographic
keys.  This can allow recovery in the case of lost access but there is
no indication that Unstoppable Domains will limit the ability to
transfer out in the event of a compromise.

The resolution process, however, is completely dependent on
Unstoppable Domains and there is no cryptographic integrity in the
JSON API, so even if a name is “owned” by a wallet entirely outside of
Unstoppable Domain’s control, Unstoppable Domains in practice has
complete control over what data is actually returned to a client in
response to a request, allowing it to both censor and modify values at
will.

This control was clearly executed in the case of the `.coin` domain.
Although the names remained in control of the “owners”, the ability to
resolve the names was removed by Unstoppable Domains with a single
change to their API.

Unstoppable Domains does not currently offer any sort of dispute
resolution service, which leaves both compromises and errors
unrecoverable (except in the case where a lost password for accessing
a name in Unstoppable Domain’s custody), but their position in the
resolution process would enable them to implement a dispute service.
However, resolving disputes would make it plain that “Unstoppable”
Domains are anything but, and the “uncensorable” blockchain data is
not the final source of truth in the system.

**Data Privacy:** Due to Polygon and other chains acting as a global
public ledger, all the domains are public in namehash form, allowing
any human-recognizable domain to be effectively brute-forced to be
identified.  All data associated with the domain itself is public and
with no obfuscation.  For subdomains (e.g. `an.example.crypto`) it is
recorded by namehash so if the name is predictable both the name and
associated data can be known but if the name is unpredictable only the
data fields are public.

Reading this data outside of a smart contract is private except to
Unstoppable Domains’s API service provider, while reading the data
within a smart contract is public as the entire stream of execution of
a smart contract is a public activity.

**Record Enumeration:** As Unstoppable Domains themselves are the only
one issuing calls to the smart contract that registers names, this
registration can be performed by namehash.  Thus only human-readable
names can be externally enumerated by using a brute-force hashing
method similar to NSEC3 enumeration.

**Data Update Mechanism:** A transaction to update data in the public
registrar costs roughly 40000 to 100000 gas, depending on the type of
update (cryptocurrency keys or larger text records).  If the only use
of Polygon was Unstoppable Domains this would support a global update
rate of roughly 45 to 90 names/second before the cost per transaction
grows exponentially.

Unstoppable Domains is just one use of the Polygon blockchain.  The
Polygon blockchain regularly experiences fee spirals where the cost
per transaction can double in moments.  For users operating through
Unstoppable Domains’s custody service this isn’t a problem as
Unstoppable Domains pays the gas fees and Polygon is currently
inexpensive to use but it does impose a global update limit.  However,
if gas fees on Polygon ever approach those of Ethereum, Unstoppable
Domains would probably increase their hosting prices substantially to
pass on these costs to the customers.

**Integration Into Existing DNS:** There is currently no suggestion that
Unstoppable Domains wishes to add a DNS gateway to their API, either
under their own domain names (e.g. `crypto.unstoppabledomains.com`) or
by registering as a new DNS Top Level Domain (TLD) registrar.
Implementing a DNS gateway under an existing domain would be
straightforward technically and legally, but it would make it clear
that the “Unstoppable Domains” namespace is separate from the DNS
namespace.

Unstoppable Domains could theoretically register as a provider for new
DNS TLDs, but there are several complications.  The names Unstoppable
Domains uses (especially `.crypto`, `.nft`, and `.888`) would be highly
coveted TLDs for competitors or others and for Unstoppable Domains to
represent DNS names they would need to control all TLDs they currently
implement.

But even assuming Unstoppable Domains could register as a TLD provider
it would probably be incompatible with their business model.  The very
premise, that domains are “unstoppable”, runs counter to the legal
requirements that a TLD needs to meet in the DNS system.  A registrar
for a TLD needs to provide a dispute resolution service, blocking or
removing names subject to various legal actions.  Acting as a TLD
registrar would mean Unstoppable Domains couldn’t be “unstoppable”.

**Overall Suitability:** Unstoppable Domains has several limitations that
may prove problematic if it seeks to act as a significant TLD.  There
exists no dispute resolution service to moderate disputes or recover
from errors such as lost or stolen cryptographic keys.  Adding such a
dispute resolution service runs counter to the “unstoppable”
narrative, and would require that Unstoppable Domains implement a
control layer in the APIs that are used to actress the data.
Unstoppable Domains appears to have no interest in integrating into
existing DNS.  The Polygon data storage layer also has problems.  Like
all “blockchain” systems it actually relies on centralized providers
to read data and has a limited update bandwidth.  Under current
conditions the Unstoppable Domains system is limited to a global rate
of roughly 90 data updates per second, assuming no other uses for the
Polygon blockchain.