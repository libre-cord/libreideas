# Libreideas

Libreideas is the idea archive for [Librecord](https://github.com/libre-cord/librecord) and everything that may grow around it. It is where designs are written down before they become code: what the thing would be, what already exists to build on, what it would honestly cost, and which rules keep it from quietly turning into something else. Nothing in this repository runs anywhere. It is the thinking that comes before the code.

An idea lives here while it is still a design: it can be argued with, redrawn and parked for years without anything breaking. When an idea earns its cost, it graduates to its own repository and its document becomes that project's founding design note, the way librecord itself carries its architecture guide at the root.

> Ideas are cheap to store and expensive to rush. The whole point of this archive is that parking an idea costs nothing and losing one costs everything.

## What's here

| Idea | Document | What it covers |
| --- | --- | --- |
| The central node | [central-node-design.md](central-node-design.md) | A hub-and-spoke network over sovereign Librecord instances: a capital that is itself a locked-down instance, a directory that crawls every instance's well-known payload, a passport, residence instances that hold accounts and encryption keys without hosting communities, and a blind switchboard for cross-instance messaging. Staged from an afternoon of work to a multi-year client project, with every stage usable and survivable on its own. |

New ideas arrive as one document each, linked from the table above.

## How an idea is written

Every document here follows the same shape, so that a reader can judge a design without knowing its author:

- **The model in plain sentences.** What the thing is, who runs what, and what happens when each piece disappears.
- **What exists today.** Every part of the current software the idea can stand on, with names, because a design that ignores what is already built is a wish.
- **The tiers, cheapest first.** Each stage has a cost, and the cost is stated honestly, including the ones that make the whole idea a bad deal.
- **The invariants.** The rules the design must never break, stated once, so that future convenience has something to collide with.
- **The build order.** What to do first, what to defer, and what to decide only once real usage has had its say.

Documents in this repository are prose with no code. When an idea needs code to be judged, it is not ready to be judged.

## When an idea becomes code

An idea graduates when its first tier is worth building: it moves to its own repository, its document moves with it as the founding design note, and the row in this README becomes a link to the living project. The archive keeps nothing that runs, on purpose, so that graduating never means deleting, and parking never means rotting.

## License

[GNU Affero General Public License v3.0](LICENSE), matching librecord and libreapps, so anything carried out of this archive inherits the same terms.
