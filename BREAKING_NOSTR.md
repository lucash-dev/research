# Breaking Nostr
*attacking the claimed censorship-resistant alternative to Twitter*

*by [lucash.dev](github.com/lucash.dev)*

## Introduction

Nostr is a new protocol/standard that bills itself as "a truly censorship-resistant alternative to Twitter that has a chance of working" [[1]](https://github.com/nostr-protocol/nostr). It has recently gained notoriety as the creator of the project received a significant amount of donations by former Twitter CEO Jack Dorsey [[2]](https://www.coindesk.com/tech/2022/12/15/jack-dorsey-gives-decentralized-social-network-nostr-14-btc-in-funding/)

In this article we'll see how *in it's present state* Nostr is extremely unlikely to work as
a an alternative to Twitter, due to scaling issues and DoS vectors -- and **isn't censorship-resistant in any practical sense of the word**.

It's name stands for "Notes and Other stuff Transmitted by Relays" which is *in fact a much fairer
description* of what the standard *actually does* (and sounds a lot lower-lever than an alternative
social media platform/protocol).

A very simplistic bullet-point explanation of how Nostr works:

- The network is composed of two types of entities: *clients* and *relays* (servers).
- Clients connect to relays using *Websockets*.
- Each client can connect to any number of relays freely chosen by the user.
- Clients publish to relays *events*, that is structured messages, that have authorship attested
by cryptographic signatures.
- Clients can fetch from relays *events* produced by any author, indexed by the author's public key, or by the hash of the event data.
- There are many *kinds* of events, the most important being *text note* which corresponds to the concept of a Tweet by an author.
- There's no standard (or implementation so far) for *relays communicating with each other*.
- For an author's note to reach a follower (a user that is constantly fetching the author's notes
from the relays), it is necessary that the author and the follower be connected to *the same relay* (they can be connected to as many other, not-shared relays as they want though).

For further and more precise details about the functioning of the protocol refer to official documentation [[3]](https://github.com/nostr-protocol/nips/blob/master/01.md)

## What Nostr *does* accomplish

Before we go on explaining Nostr's shortcomings I think it's good to acknowledge what it *does* accomplish:

- Nostr sets a standard for message formats for Twitter-like platforms that's minimalist an elegant.
- It gives **strong cryptographic guarantees** of the *authorship* of messages. Messages are **non-forgeable** and **tamper-resistant**.
- Establishes *self-sovereign identities* that can be easily carried from one platform (relay) to another.
- Allows for enhanced privacy by allowing new identities to be generated as often as needed.

I would in general classify the Nostr *standard* (rather than the current state of implementations) as significant but incremental improvement over the status quo -- having multiple accounts in different centralized, non-standardized platforms and possibly
manually validate identities over them using PGP.


## Basic issues of scale

Before we move on the more complicated matter of censorship-resistance, let's get out of the way
the obvious fact that neither Nostr implementations nor protocol specification are right now prepared
to deal with even 1% of the scale of Twitter.

As I don't believe there is any doubt about the statement above, let's just quickly enumerate a few points supporting it.

- Twitter deals with hundreds of millions of daily active users -- with an IT infrastructure cost that presumably costs at very least many millions of American dollars a year.
- Nostr doesn't have any way to *load balance* authors or requests between different relays.
- There is a strong incentive for authors to publish their notes to many relays, and for followers to query many relays.
- It follow that likely each relay will have to deal with a large portion of the total number of notes being generated.
- There's at present no standard for *charging clients* for either publishing notes or for querying them, relays are likely to run as not-for-profit endeavors.
- There's also no complete anti-spam mechanism in the protocol, though some incipient standardization is being constructed [[4]](https://github.com/nostr-protocol/nips/blob/master/13.md).

All things considered, it looks like the most likely scenario if Nostr gains millions of users as it is *right now* it will completely stop working, discouraging the new users for a long time.

Fixing the scaling problem in the sense of having relays capable of handling the amount of traffic is *relatively simple* in the sense that it doesn't need any new technical breakthrough -- it only takes a *business* breakthrough and small tweaks to the protocol (e.g. better spam protection). If developers can raise enough capital these are issues likely to be solved -- though it would also likely completely change the "decentralization profile" of the network.

Which brings us to more interesting issues.

## Censorship-Resistance

Even if Nostr is modified to be able to handle traffic at least comparable to Twitter (say 1M active daily users), it's still far from clear how it can possibly achieve its stated goals.

The boldest claim made by Nostr authors is that it is **censorship-resistant**.

While there isn't a universally accepted definition of "Censorship Resistance", there's quite some scientific literature on the subject, and various attempted definitions and *mathematical proofs* of certain protocols being or not being censorship-resistant.

For interesting examples, see [[5]](https://www.freehaven.net/anonbib/cache/ih05-csispir.pdf) and [[6]](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=5ca963f25befa66a80a2511734e65c2b8128c9f5).

While no specific definition of the term **censorship-resistant** is given the author(s) of Nostr (and no proof or other arguments to support their claim), I think it's fair to compare it to a protocol widely considered as censorship-resistant (Bitcoin [[7]](https://bitcoin.org/bitcoin.pdf)), given the overlap of the Bitcoin community and Nostr community. For example the main author of the protocol has made contributions to multiple Bitcoin-related projects [[8]](https://github.com/fiatjaf)

(incidentally, the author of this document has made contributions to Bitcoin-Core in the not-so-distant past as well)

Let's make some informal definitions that would apply both to Bitcoin and Nostr as "messaging systems".

- **Author**: any entity that produces and wants to transmit messages.
- **Receiver:** any entity that *wants* to receive a set of messages with certain properties, e.g. from a certain author.
- **Censorship**: means some adversary being able to select some set of messages and prevent them from being received by some set of receivers.
- The system is said to be **Censorship-Resistant**, under a set of assumptions (about authors, receivers, adversary capabilities), if the probability of the adversary achieving **Censorship** is low.

In the case of Bitcoin block mining (let's forget about transactions for this example), we can say:

- The **authors** are the nodes mining blocks.
- All nodes are **receivers** that want to receive blocks belonging to the block-chain with most proof-of-work.
- An **adversary** might control a subset of nodes, but can't fake proof-of-work.
- **Censorship** would mean preventing the chain with the most proof-of-work being received by a subset of *receivers*.
- The system is intuitively **censorship-resistant** if:
  * Between each **author** and **receiver** exists *a path of honest nodes not controlled by the adversary*.
  * Honest nodes control a *majority of hash power*.
  * Even if a node isn't presently connected to any honest peer, it is still possible to achieve *censorship-resistance* if nodes can **detect attempted censorship and repeatedly connect to more peers until an honest peer is reached**.

If we add the assumption that each node has a good estimate of total hash power in the network -- then *censorship can be easily detected* by a significant drop in the total proof-of-work in the best chain they receive, which means the system is *censorship-resistant* under relatively *weak assumptions*. Also, it is impossible to censor without spending *considerable real-world resources*.

Similarly for Nostr:

- **Authors** are holders of private-keys that want to publish broadcast their notes to followers.
- **Receivers** are clients querying relays for notes by specific authors.
- An **adversary** might control a subset of clients and relays, but can't forge digital signatures.
- A *relay* controlled by the **adversary** can censor individual *notes* and *authors*, for any subset of *followers*. This comes at **no cost** in terms of real world resources -- on the contrary, it might be less expensive to censor than to act honestly.
- The system is intuitively **censorship-resistant** if:
   * For all sets comprised of an author and the corresponding followers there is **at least one honest relay all elements in the set are connected to**.

Please note that "one honest server all clients are connected to" is the very definition of a trusted third-party (TTP).
TTPs are widely regarded as security holes in the context of decentralized and censorship-resistant systems [[9]](https://nakamotoinstitute.org/trusted-third-parties/). This is **much stronger and less realistic** assumption the ones above listed for Bitcoin. The condition of a TTP is also the same model used for all centralized platforms.

To be fair to Nostr and it the stated censorship-resistance-enhancing property of easily switching between relays, let's grant the following assumption:

- Even without a single TTP, Nostr is still **censorship-resistant** if both:
  1. Censorship can be detected *with high probability* by *both authors and followers*.  
  2. Clients can, with high probability, repeatedly connect to more relays until a *honest relay is found*.

While I'll offer no *proof* these conditions are necessary in the absence of a TTP, I sustain it's safe to assume so, and let it for
challengers to offer evidence of alternative ways of reaching censorship-resistance *without substantial changes to the Nostr protocol or adding new protocols on top of it*.

We'll now offer informal arguments that both conditions above are *unlikely to hold true without substantial change to the protocol*.

## Detecting Censorship

As far as this author can tell -- there isn't any mention in all Nostr documentation about how to detect
censorship.

There are numerous suggestions on actions in case of being *banned* which is a particular sort of censorship
when relays explicitly disallow a user of using its services.

A few different examples of censorship (not exhaustive):

- *Shadow ban*: relay stops serving certain notes, without any explicit mention of the fact to the authors. The next examples are sub-times of shadow ban.
- *Follower-subset censorship*: relay stops serving certain notes only to a (possibly small) subset of followers, identified
   by IP addresses or any other means.
- *Temporal censorship*: relay stops serving certain notes during specific intervals of time.
- *Query censorship*: relay removes serving certain notes as results of specific queries, but not when directly queried by author or id. For example, a note might be censored when searching for a certain tag, or if the search is by prefix rather than exact match.

There's no part of the protocol for implementing mechanisms for detecting any type of non-explicit censorship. In itself this fact
already disqualifies any idea that the protocol is censorship-resistant in the sense defined above.

Nevertheless let us see if there are any intuitive ways for detecting censorship, even if not specified in the protocol.

For a *follower* to detect notes by a particular author are being censored, it is a necessary condition that the *follower*
**knows** about the existence of the notes. Based on that fact, a simplistic protocol that would allow a *follower* to detect censorship could be

- *Author* an intended *rate* at which it will publish notes, e.g. as a maximum interval between notes.
- *Author* includes in each note a sequence number.
- *Author* publishes notes at least at the advertised rate. Blank notes are published if necessary.
- *Follower* queries relays periodically for new notes by the author.
- *Follower* detects censorship (or failure) if there's a gap in the sequence numbers of the received notes.
- *Follower* detects censorship (or failure) if there's the time since the last note is more than the maximum interval advertised.
- If censorship (or failure) is detected, the *follower* will attempt to reach out to more relays until the missing notes are found.

While simple to implement this protocol has some important limitations:

- Doesn't allow for detecting *query censorship*.
- Requires a liveness commitment by *authors*.
- Can be abused by *authors* to force *followers* to consume resources looking for more relays.
- Can't be used by *authors* to detect censorship.
- Can't distinguish censorship from *author* failure.

As noted before it's essential that both *author* **and** *followers* detect censorship so they can both
look for new relays (and hopefully find one in common). So a way for *authors* to detect censorship is necessary.

A naive way of performing that detection would be to use the same simple protocol above, but having the *author*
perform the *follower* actions as well.

The problem of course is that relays might detect the requests are being made by the same client that originally
posted the note, by IP address or otherwise, and bypass the censorship routine in these cases.

Even if we assume both *followers* and *authors* use Tor[[10]](https://www.torproject.org/) or a similar service, obtaining a minimum level of
reliability in the detection of censorship would require continuously accessing the relay using
*every exit node available*.

While such a scheme *might* work, that isn't obvious or guaranteed, and the precise details of how it is implemented
might be difference between resisting censorship or not.

That means the failure to address censorship detection in any detail in the protocol is a fatal blow to any claim of censorship
resistance.

However, even if we assume the problem of detecting censorship (or failure to connect) *is solved* -- we would still need
a way for both *author* and *followers* to reach a consensus on which *relay* to use, in order to claim censorship resistance.

## Censoring and Spamming Relay Discovery

One important feature of the Nostr protocol not mentioned before is **relay hints**.

In many different places in the protocol, the author of a note can insert URLs for locating relays expected to store
certain notes. Authors are able to publish *relay recommendations* as well. These hints and recommendations can be automatically interpreted by clients to obtain a list of possible relays if needed.

The expectation is that such suggestions would allow for a reliable list of all available relays, and that
in case of censorship or failure, clients can try the other relays until a reliable one is found. This idea is limited by two
immediate problems though:

1. Authors can publish invalid or malicious relay hints and recommendations -- and these are not validated by relays
2. Relays can censor relay hints and recommendations

This means that the number of relays known might be intractable with the client's resources, and even if it's possible
to try all of them, the relay you need to find might not be in the list, since the *censored author* the *follower* is trying
to reach can't be counted to post information on the relay it intends to post to.

From the point of view of an *author* the issue is even worse. There is simply no algorithm whatsoever in the standard for
selecting which among a vast number of possible relays to connect.

**Followers don't post relay recommendations, and authors don't know who follows or might follow them**.

The only accessible heuristic is *using the same relays as other authors you think might have a similar follower base*.
Any other relay the *author* uses is unlikely to be accessed by followers in a censorship scenario, unless the author
somehow convinces other *authors likely to be popular to use recommend them too* (and there's no way to independently verify
how popular an author is).

The more resource-intensive is posting notes to relays (network resources, PoW, payments) -- the smaller the incentive to add
new relays.

## Centralization Risk

The incentives described above already strongly point to centralization of the network in few relays or groups of relays.

Accounting for the costs of running a relay in any significant scale only makes things worse.

If number of users (legitimate or spammers) increase to the order of magnitude of 1 million active daily users, it will
be beyond the ability of the average enthusiast to run a relay. On the other hand, businesses with large amounts of capital,
that profit by either charging for services or commercializing data mined from notes would likely be able to keep their relays
-- as gains of scale dominate. The same might be true of state actors and particularly wealthy non-profits.

## Sybil, Parasite and Byzantine Relays

While the endgame for Nostr relays likely looks like only a handful of relays being around -- any client using
relay hints and recommendations will likely end up with an intractable list of thousands or possibly millions of
available relays **even after removing unreachable or unresponsive URLs**.

The reason is that while running a honest relay is expensive **running a dishonest relay is cheap**.

Executing a Sybil attack by creating 100 aliases for a single "real relay" is considerably cheaper than
running 100 relays, as the internal processing of connections can be performed by the same infrastructure,
and a single database could be used to store the same notes just once. Sybil attacks can be used to charge multiple times for the same service (a perfect crime if anonymous payment is possible) or DoS clients in the case HashCash is used.

**There is nothing in the protocol that allows for distinguishing a Sybil from a "real" relay**. A client
might push their notes and queries to 1,000 relays and in fact add absolutely no redundancy.

Moreover, creating a "Parasite Relay" -- a relay that simply delegates queries to other relays -- is even cheaper
if payment for using services is required.

**And all the thousands of Sybil relays can easily collude to censor content.**

This might be countered by implementing some anti-Sybil extension to the protocol -- like requiring
relays to post a Bitcoin bond. Besides the fact that would amount to a major change to the protocol,
it further increases the tendency to centralize the network around a handful of relays.

Without any changes to the protocol, the best client would be able to do under such attack would be to
give up completely relay hints and recommendations and require users to manually select **trusted relays**
that will be exclusively used. These relays would need to be vetted out of band by users, by whatever
means available.

## Unspecified Censorship-Resistant side-channels

In discussions in the Nostr Github repository [[11]](https://github.com/nostr-protocol/nostr/issues/102)
an idea was brought up that other means outside of the protocol -- such as friend-to-friend sharing
by any communication means -- might be used to let followers know about what relay an author is using.

There are a number of issues with that idea:

- It still leaves unanswered the question of what relay an author *should* use.
- As an author doesn't know who the followers are, it's unclear why such social networking would be assumed
to be reliable.
- Above all, it's **illogical to claim a protocol's value proposition is being censorship-resistant then justify the claim with
reliance on an unspecified second censorship-resistant protocol**.

I suggest it *might be* possible to use a second (possibly more expensive) censorship-resistant protocol (adding data to
  Bitcoin transactions?) to bypass the censorship of relay recommendations -- however, such mechanism should be specified
  *as a major change to the protocol*.

Besides, solving that problem would still leave a number of major issues open.

## Putting all of it together

Let's sum up what is the likely scenario for Nostr if it's usage keeps increasing -- unless major changes are implemented (what exactly? that's still an open question)

- Relays will likely be unable to handle the number of requests
- Spammers might easily DoS the network due to lack of proper spam protection
- Detecting censorship in order to choose better relays will be very hard and not automated.
- If a few relays are able to properly scale, they are likely to be outnumbered by
  sybil and malicious relays, making the automated discovery of relays mostly useless.
- Authors will tend to choose only a handful of the relays.
- The relay space will be dominated by a handful of companies or other organizations.
- Users will need to manually select the relays they trust, and figure out what relays
  to trust by means completely outside of the protocol.

That points to relays being **trusted third-parties exactly like the centralized platforms are,
with as much power to censor and impose conditions on users**.

Redundancy between relays is in its essence (at best) the same as having accounts in multiple centralized platforms.
Without any way of detecting that multiple relays are actually separate entities, the user might be better off
just choosing well-known companies with some level of regulated governance (e.g. board of directors).

Self-sovereign identities defined by public keys are useful, but that's already possible by
using PGP to authenticate accounts in centralized platforms.

What Nostr *does* improve though, is making the trust model perhaps more explicit, and offering
a standard way of switching platform/provider when there's a breach of trust.

## Fixing it?

While I made numerous suggestions for fixes or improvements above, I believe making Nostr
truly censorship-resistant will require more than a number of adhoc fixes -- it will require a
breakthrough.

As with any project (especially young projects), things are in constant flux, and there might already
exist NIPs (Nostr Improvement Proposals) being written than address some of the issues.

I have been working on similar protocols for a few years as well, and plan to write a few more articles
-- and maybe some software -- with further ideas for stacking other protocols on top of Nostr, that might possibly help.

Whether my ideas or anyone else's will succeed in making Nostr achieve its stated goal is completely unknown.

## Conclusions

In this article we examined the claim that Nostr is "a truly censorship-resistant alternative to Twitter".

Several arguments were made to show that the claimed censorship-resistance has little chance of success in the present
version of the protocol, and the claims are little more than unsubstantiated hype.

A better, more precise description for Nostr might be **"a protocol for Twitter-like platforms based on self-sovereign identities"**.

I believe there's genuine effort and good-faith interest behind Nostr, and hope that
exposing its weaknesses in this article -- and other future contributions -- will help improve the protocol, and perhaps find different use cases for it.
