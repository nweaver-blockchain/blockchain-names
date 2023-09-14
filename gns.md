---
layout: base
---

# GNU Name System

**Name-Service Concept:** The [GNU Name System
(GNS)](https://lsd.gnunet.org/lsd0001/) is designed to offer a
distributed, censorship-resistant naming scheme that is
cryptographically authenticated.  A zone is identified not by human
readable names but by a public key used to sign the data, forming the
only globally consistent namespace.  Instead, users are intended to
create a local configuration mapping human readable zone names to
public keys, creating a deliberately inconsistent global namespace.

The GNS system is also incomplete: it only specifies the wire format
and the resolution client.  Actually obtaining the data elements
themselves relies on the presence of a separate distributed hash table
(DHT) to load and store key/value pairs.  Current implementations use
R5N, the GNU Network distributed hash table.

**zTLDs and "Pet Names"**: The primary concept behind GNS as a naming
system is that primary names (zone Top Level Domains or zTLDs) are
self attesting as they are effectively the public key associated with
all records for that zTLD.  By signing all data with the corresponding
private key a client can be assured that any records fetched relative
to the particular zTLD maintain cryptographic integrity.

However, this approach results in the top level domains of the global
namespace only consisting of non-human-readable entities, such as
`example.000G006K2TJNMD9VTCYRX7BRVV3HAEPS15E6NHDXKPJA1KAJJEG9AFF884`.  A
GNS client however can internally specify “Pet Names”, a mapping of a
DNS style namespace to a particular zTLD, so `.gnu` might refer to
`000G006K2TJNMD9VTCYRX7BRVV3HAEPS15E6NHDXKPJA1KAJJEG9AFF884`, so
`example.gnu` could be resolved by a GNS client.

The result is that although the global cryptographic namespace is
consistent among all clients, any fully human readable names will be
inconsistent unless all users use a common source for their pet names.
The GNU Name System's [GNUnet Assigned Numbers Authority (GANA)](https://gana.gnunet.org/) offers
a first-come, first-served mapping for mapping human-readable
subdomains in an `.alt` pet-named GNS to zTLDs.

Note that although GNU Name System could be used to create a globally
consistent, human readable namespace in the same manner that DNS does,
by having a common root of trust that effectively everyone agrees to
use, doing so would require the creation of a central authority.  The
use of a central authority runs counter to the decentralization and
censorship-resistant goals of GNS.

**Cryptographic Primitives:** The primary cryptographic primitives are
based on public key operations, although there are some special
primitives needed for GNS to operate.  In addition to the standard key
creation (creating a private key ***d*** and corresponding public key
***pk***), cryptographic signatures using ***d*** to generate and
***pk*** to sign, there is a key derivation function that accepts both
***d*** or ***pk*** and a label (e.g. `example`) to create a
corresponding blinded public/private keypair ***d'*** and ***pk'***.
These blinded keys are unlinkable to the original key, meaning that
without knowing (or brute-force searching) on the label it should be
impossible to link ***pk*** and ***pk'***.

These blinded keys can also cryptographically sign and verify.  There
are also two other functions, one to take the public zone key, label,
and timestamp to perform symmetric encryption on records (`S-Encrypt()`)
and the hash of ***pk'*** identifies the location in the DHT for storing
records associated with the label.

The intent of this approach is to provide some level of privacy.
Without knowing the label it should be impossible for an
honest-but-curious DHT to link records, as each different label will
produce a different blinded ***pk'*** used to both store and authenticate
the associated data records.  However, given the tendency for names to
represent human meaningful, low entropy sources it is likely that a
privacy-invasive DHT participant could simply brute force all the
possible ***pk'*** based on human-readable labels in order to map the
namespace of a zTLD.

**The Distributed Hash Table:** The primary storage for GNS is
unspecified, as it instead relies on the presence of a separate
distributed hash table to store records.  This DHT must accept 512b
data keys and store sufficiently large data values at that location.
To prevent further confusion this document will refer to the DHT key
as an "index" rather than a key since it is not a cryptographic key
per-se.

**Record Types:** GNS supports all DNS records as well as its own
additional records.  These additional records can include delegation,
redirection, and redirection to an external DNS server specified by IP
address.

**Name Storage & Resolution:** GNS uses these cryptographic primitives
to build its secure domain resolution.  Given a name such as
`a.b.{zTLD}`, the recursive lookup will first derive the ***pk'*** for
`b.{zTLD}`, which the hash is used for the index in the DHT.  The data
stored in this location is all the data for `b.{zTLD}` contained as a
sequence of records.  Each record has, in plaintext, the expiration
time, the size of the record, the flags, and record type.  The records
themselves can be arbitrary DNS records or GNS specific records.

The data record block itself contains the blinded key ***pk'***, a
signature over the remaining encrypted data, and the expiration time.
The remaining data in each record is encrypted with the `S-Encrypt()`
function, which since it includes both the blinded key ***pk'***, the
label, and the expiration time, it can only be decrypted by someone
who knows the original ***pk*** and knows or brute-force searches the
label but anyone can verify that the block is valid without knowing
the original ***pk*** since it includes (in plaintext) the blinded key.

Since records can include aliases and redirection, it needs to proceed
in an iterative process, first resolving `b.{zTLD}`, checking to ensure
that there are no aliasing or redirections (cname-like) records,
before proceeding to resolve `a.b.{zTLD}`.

Record Privacy: This scheme is heavily focused on record privacy.
Without knowing the original ***pk*** (the zTLD itself), it is impossible to
know what names a client is looking up (except perhaps by using global
traffic analysis of all records as they are added and accessed).  Even
someone who knows pk also needs to know or guess the label itself
associated with records.

The scheme also never requires storing the original ***pk*** in the
DHT, it is only used as the top level domain in records as they are
set and looked up, so the DHT should never learn the original ***pk***
from internal information.

However, for a particular ***pk'***, the number, type, and size of
records is visible in plaintext, as otherwise a client would need to
decrypt all the records for a name to resolve any given record.

**Enumerating Records:** It is impossible to enumerate the space of zTLDs
from within the system, as the zTLDs themselves are never transmitted
and everything uses blinded derived keys.  For a given zTLD it is only
possible to enumerate labels when the labels are low entropy, as the
derived key (and storage locations) are a function of the zTLD and the
label itself.

**Censoring Records:** The actual records stored in the DHT are
self-attesting: the blinded zone key ***pk'*** is used to create both the
index and to cryptographically sign the record block which includes a
copy of the blinded zone key.  A GNS aware DHT can ensure that only
valid records are stored in the DHT as it could reject any record
blocks that fail to validate.  In the absence of such a mechanism,
however, anyone could arbitrarily censor known labels on a targeted
zTLD by simply overwriting invalid garbage into the DHT.  Thus
availability requires that the DHT explicitly understand the GNS
records stored.

Additionally the standard DHT eclipse attacks still apply, which means
availability needs to depend on the availability of the underlying
hash table.  There is no NSEC/NSEC3 style provable denial of
existence.  If a client attempts to fetch data at an index and
receives no data, it has no way of determining if there is no data or
if the DHT itself is blocking access to that data.

Finally, the censorship resistant property remains only as long as
there is no attempt for globally consistent, human readable names.  If
there is a central authority that is a commonly trusted root of trust,
that central authority can and likely will have to remove
human-readable names subject to various legal processes.

**Abusing the DHT:** Even a DHT that only supported GNS records could be
abused by a GNS-derived service to store arbitrarily large data blocks
throughout the DHT, allowing the DHT to be silently co-opted to
provide a large data pool for distributed storage.  Thus even a GNS
specific DHT could be potentially hijacked and abused in ways that the
DHT would have difficulty distinguishing from normal operation.

Assume that the DHT supports only 10kB record blocks.  A scheme could
take a file at a GNS location and split it into a series of
cryptographically derived labels (with redundant labels to handle
reliability issues).  The data itself could then be stored in opaque
GNS data record blocks which, without knowing the labels, are
indistinguishable by the DHT from normal GNS records.

**Dispute Resolution:** There is no method for dispute resolution in this
system.  The zTLDs themselves are not human readable and thus probably
do not need a dispute resolution system, but any central provider of a widely
used TLD pet-name service (such as GANA's .alt) would be able to
implement a dispute resolution system.

**Recovery from Cryptographic Failures:** If a zone operator loses
their cryptographic key for a zone it is unrecoverable.  Anyone
relying on the zone would need to be notified using some out of band
method about the new zTLD.

A stolen key can be revoked but this marks the domain as
non-resolvable.  Overall revocation occurs outside the framework of
the normal data storage.  Instead all GNS clients need to maintain a
local copy of all still-valid revocation messages ever sent to ensure
they don’t resolve a revoked zTLD, or there needs to be a trusted
ledger to maintain all these records.

These revocation messages include a proof of work to prevent flooding
this space, but this scheme also has revocation messages expiring
after a year.  So in order to ensure that a revocation actually
persists a longer proof of work needs to be generated or the 'revoked'
key needs to be maintained to create new revocation messages on an
annual basis.

It is unclear whether the proof of work scheme is an effective
spam-prevention method, as it is generally assumed that attackers can
have access to orders of magnitude more resources (by using botnets)
than legitimate users, limiting the effectiveness of the proof of work
scheme as a DOS prevention.

GNS depends on an unspecified broadcast to send revocation messages,
outside the unspecified DHT storage method, which also offers the
possibility of an attacker disrupting this process and maintaining
control over the domain for an additional period of time.

**Security of Name to Data Mappings:** The data records in GNS are self
attesting: the zTLD is the public key, while any given record is
cryptographically signed and can be verified by the derived ***dk'***.  However, any central mapping service mapping pet names to zTLDs has to be trusted, because the central mapping service sets the name to zTLD mapping, which is effectively a similar trust model to DNS.

**Interacting with DNS:** GNS, unlike other alternate naming systems, is
designed to integrate with existing DNS.  For names specified by zTLD,
or where the user has configured a local pet-name for a zTLD, a GNS
resolver can use GNS to resolve records.  Otherwise, a GNS resolver
should use ordinary DNS.  GNS specifically supports the entire 16b
space of DNS RTYPES, and can include redirection records in GNS that
redirect to DNS resources.

**Conclusions:** The largest concern with GNS is the lack of a
globally consistent human-readable namespace.  It is impossible to
ensure that pet-names are consistent, thus all GNS records should be
identified by zTLD.  The only way to avoid inconsistency amongst pet
names would require a common central authority which runs counter to
the goals of decentralization and censorship resistance.  This is
effectively unusable if a GNS name needs to directly interact with a
human, such as in a URL or other resource where a human is expected to
understand, remember, or transmit a name.

The other concern is that GNS is currently an incomplete system: it
only specifies a data format and resolution process, the key problem
of distributing records relies on an external distributed hash table.
If the DHT is not aware of GNS then it is trivial to censor records.
A GNS aware DHT is harder to censor but still vulnerable to potential
attacks.  In particular even a GNS aware DHT could be abused to offer
a large data distributed storage service, which would impose a
substantial load on the (volunteer) computers running the DHT.
