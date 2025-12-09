placeholder related to custom networking in vrc, though i may not write a proper explanation for a while


after an initial version (which i call v1), i realized the need for a different approach. v1 handled network objects at network object level. their logic and state would exist on a derived network object. i learned that this was a mistake when it comes to sane structure and interacting with logic as a creator. i also used 1 udon/vrc network event per message, this proved to be extremely costly on metadata/header vrc side. when starting v2, the first thing i made was a way to bundle messages (basically structuring a packet) and handle them in a queued fashion. v2 also uses a network behaviour style, where network objects exist more as data holders/descriptors alone


why?

first, it allows network spawning and despawning. this negates the strict need for rigid pools that hold all synced objects in advance (at editor/upload time)

pooling can be a pain to interact with, requires you route through a central source anyway to avoid race conditions, requires enough of every type of object in amounts that are enough for each player (kind of an n squared problem). It's also not exactly free for udon synced objects to just "exist" in an unused state afaik


more than that, it allows structuring logic in a client-server style which is much easier to account for than the typical distributed authority setup in vrc (this is actually the biggest benefit by far; you can do this in vrc, but structuring it in this way makes it much more clear-cut); we strictly require a consistent host player. logic reliant upon the host and many networking concepts will cease to work if the host leaves


a side benefit is the allowance of segmented serialization. udon/vrc networking does allow multiple synced components on an object, but a call to serialize will serialize all components on the object.
that means variables you set rarely (a name field that players can modify) must be serialized with variables you sync often (a position, a destination, etc.)

you can get around this by splitting the synced components across multiple objects, but this has odd ownership handling across those objects and is more error-prone.
Though it does technically have some slight atomicity implications for components on the same object, but it would be very odd to rely on cross-component atomicity in that way



late joiners are handled in a fairly specific way. first we snapshot all networked objects, then we pass it to the late-joiner over time. they queue session calls to process after they fully receive the snapshot. they continue to queue session calls until they get through all of them, enforcing order. conceptually not very hard, but it can be a bit difficult to structure and requires a decent understanding of how to order the things you want to do
