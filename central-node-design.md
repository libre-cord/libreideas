# Central node design note

A design sketch for a central hub over Librecord instances. Kept for safe keeping: if this becomes real, it becomes its own repo, and this note is the founding document.

## The model

One central node over many sovereign instances. In governance terms: the central node is the federal government, the instances are local governments and autonomous zones. Every instance keeps its own database, accounts, rules and moderation, and stays reachable directly, forever. The central node never hosts anyone's community. It is a directory, a passport office and a switchboard, and nothing else.

Direct access to instances is a load-bearing part of the design, not a fallback detail: if the hub is down, blocked or seized, every instance keeps working and people keep talking, because they can always go to their own server the way they do today. The instance switcher in the desktop and Android apps stays.

## What librecord already provides

- Every instance publishes its dossier at `/.well-known/librecord`: name, description, API, gateway, CDN and voice endpoints, registration policy, invites, E2EE, contact and links. That is exactly the data a hub aggregates, so the instance side of the directory already exists.
- The desktop and Android apps already discover, validate and jump between instances, with saved instances and one-click switching. The traversal protocol is live.
- Instances have an `instanceId` and per-instance users, so a `@user@instance` handle scheme has raw material.
- The OAuth2 machinery (apps, tokens, scopes) is the scaffolding a passport would ride on, though the consumer side does not exist yet.
- Meowcord once shipped peer-to-peer federation; this lineage removed it. `REMOVED_KEYS` in `src/util/util/Config.ts` still lists its keys, and the old `meowcorded/meowcord` repository is the archaeology site: study what its authors built and why it was dropped before designing anything.

## Tiers, cheapest first

1. **Directory.** A small service that crawls every listed instance's well-known endpoint and renders the whole network: instances, their rules, registration, community stats. Jumping to one goes through the existing switcher flow. Works with the current clients and needs almost no instance-side work. This alone is most of the felt value of "see everything in one place."
2. **Passport.** One account at the hub that maps to your identity on each instance, and sign-in delegation so arriving at a new instance does not mean a fresh signup. This is the only real instance-side work: instances today authenticate only their own local accounts. The hard part is doing it without creating a forged-passport problem, since a stolen passport is worse than none.
3. **Cross-instance lookup.** Finding people and communities across instances from anywhere. Needs a handle resolution protocol (`@user@instance`) that does not exist yet.
4. **Cross-instance content in one view.** The expensive tier, below.

## Aggregating content: take the hub-as-client path, not federation

"One client showing all your instances at once" does not require rewriting the instances. The fork:

- The meowcord way: instances speak a federation protocol and push content toward the hub. Instance-side rewrites, trust relationships, sync semantics. Expensive, fragile, already abandoned once.
- The hub-as-client way: the hub holds your passports and logs into each instance as an ordinary client over the plain Discord protocol librecord already serves, one gateway and API session per instance, and multiplexes them into one surface for a new client. Instances stay untouched. From their side, the hub is just you, online from a weird browser. Sovereignty is absolute because the hub has no special power at all.

The cost of the hub-as-client path lands in the hub, which is a Discord-protocol client library running N sessions (gateway multiplexing, per-instance state, notification fan-in, rate limits), and in the new client UI that keeps N parallel worlds coherent. That is the project class of "writing a third-party multi-Discord client," which is a serious project, not a feature. None of it touches librecord itself.

## Encrypted chat between instances: the hub stays blind

The hub must be a blind rendezvous, never an endpoint. It introduces participants and either forwards opaque ciphertext it cannot read or hands the client the other instance's endpoint for a direct connection. Keys live on the two participants' devices and nowhere else.

"E2EE living on the central node" is a contradiction in terms. A hub that holds keys or decrypts traffic is not E2EE, it is TLS plus a trusted operator, and it forfeits the property librecord's encryption is built around: keys generated browser-side, fail-closed, the instance never seeing plaintext, and staff review working only because a reporter's own browser hands over the keys of the reported messages and nothing else. Bluesky is the cautionary example, not the precedent: its direct messages are not end-to-end encrypted today, and its central relay and identity registry are the standing criticism against it. The part of Bluesky worth copying is the directory and rendezvous design, not the placement of trust.

The honest cost of cross-instance E2EE is the key exchange. Librecord's protocol is per-instance today: identities, key backups and device approval all hang off one instance's API. Two people on two instances need a spanning trust model to verify each other and exchange keys. That is client-side protocol work, and it is the real engineering in this tier, not the hub.

## Risks to carry forward

- The hub is a single choke point: one legal target, one availability target, one moderation pressure point. Directory-only limits the damage; content transit multiplies it. Direct instance access is what keeps the blast radius small, so it never goes away.
- A hub that transits everyone's content makes its operator a party to that content. Blind relay is not just a privacy feature, it is the operator's legal shield.
- The Discord web client assumes one API origin. Anything that mixes two instances in one screen is new-client territory; do not try to force it through the patch layer.

## The capital is an instance

The central node starts as another official librecord instance, locked down: closed registration with instance invites and approval, so only staff hold accounts there, and since only staff hold accounts, only staff can create servers. No new software is needed for the capital's first two jobs, because closed registration and invite-only sign-up already exist in librecord and the directory starts as a page on the instance, listing the network and crawling every instance's well-known payload the way instances.md does by hand.

The passport, and later the hub-as-client if it ever happens, are features added to this instance in place, which keeps the whole project incremental: the capital exists from day one as a staff office with a phone book, and grows toward the rest of the design only as each tier earns its cost.

One rule keeps the design honest: the capital stays a staff office, never a public community. The day the hub starts hosting the public square is the day the takedown resilience argument erodes from the inside, since the hub is supposed to be the one node everyone can afford to lose.

## Residence instances

An instance can be server only: it provides accounts and ways of doing E2EE and hosts no communities. This is the cheapest tier of the whole model because librecord already separates identity from communities: the account, the E2EE device and key endpoints, backups and approval are all per-instance and work with zero guilds, so a no-guilds instance already is a residence zone. Formalizing it takes a config flag that turns guild creation off server side and marks the instance's kind in its well-known payload, a small client patch that hides the add-server and discovery entry, and nothing else. DMs, profiles and settings stay exactly as they are.

The governance fit: residence zones are where people live, community zones are where they gather, the capital is the switchboard between them. This is the Matrix homeserver pattern, identity and keys at home and rooms elsewhere, so it is a known-good shape rather than an invention. One dependency to respect in the build order: a residence instance is coherent on its own today (no servers, DMs live there), but it shines only once cross-instance messaging exists, so it slots between the passport and the lookup tiers.

## Replacability

The design survives takedowns on two independent layers, and each layer has a rule attached:

- The hub is a choke point but not a load-bearing wall: taking it down removes the phone book, not the network. This only stays true if instances always keep local login. The passport is a convenience layer and never the only key to someone's house, so a dead hub degrades convenience instead of locking people out of every zone at once. Since convenience beats prudence at scale, the resilient path cannot depend on users choosing it: signing into an instance through the passport should provision a local credential in the same flow, so even the laziest user ends up with a local key they never had to think about.
- The code is open, so the hub is as takedown-proof as the instances: someone else stands up a replacement. The directory half is stateless, since it only crawls well-known endpoints. The passport half accumulates state, so that state stays exportable, or a replacement hub starts empty and the passports die with the original.
- Open code preserves the project, not communities. A community survives on its operator's backups, not on the license, so backups are part of the survivability story and not an implementation detail.

## Build order, if this ever happens

1. The directory. Cheap, works with the current apps, most of the value.
2. The passport. The only substantial instance-side work, riding the OAuth2 scaffolding.
3. Decide, with real usage data, whether one-view aggregation is worth building the hub-as-client and a new client at all.
4. Cross-instance E2EE as its own protocol project, whenever cross-instance DMs become real.

Invariants, stated once: the hub never hosts communities, never holds keys, never becomes an endpoint, and never replaces direct access.
