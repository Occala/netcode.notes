

Preface: This is basically custom networking in VRC. I was hesitant to call it custom networking at first, it seems like an odd claim. With that said, I only use udon's networking to send raw bytes over an event. I also use their PlayerIds and representations of players, because it'd be very indirect to abstract those and serve no purpose. This is kind of a toy network framework




I'll note a specific case of networking I tried, which turned out to be oddly difficult in udon networking

Imagine you want players to be able to walk around and mine rocks, when they mine rocks, smaller ores pop out. Players can carry around the ores, even players that didn't mine those ores should be able to do so. That doesn't sound especially hard, does it? In VRC's networking this is absurdly complex. Primarily, the lack of network spawning means you will just have a hard limit at some point for these resulting ores. They can't be PlayerObjects if you want anyone to be able to interact/pick them up, at least not easily; so you're looking at a central pool that you route into. Mined rocks that need to respawn have to run some sort of timestamping logic, so that anyone can interpret the respawn if they become the owner. It's such a simple-sounding scenario that ends up being convoluted at best in udon networking. At worst it's just not possible if you don't compromise on the idea of being hard-capped by pooling. This is the type of thing where udon networking fails and custom networking starts to win out

I tried this same scenario in FoxNet after it was stable, it just worked with minimal logic. There was no longer any complex timestamping logic, the host runs a cooldown on mined rocks. If a rock breaks, it yields network spawned ores. That's it

The real trade-off becomes the necessity of a consistent hosting player, I consider this preferable, but it doesn't align super well with VRC's very drop-in, drop-out nature

-----












A decent advantage is data reduction, relative to the large overhead/header in udon (this is very large for events, per object it's probably moderate). I only have to pay VRC's overhead once per bundle that goes out


Why make it?

I realized at some point that I was understanding the reasons behind network library design and felt like I could just try it myself. Udon's networking is fairly constrained and has limitations you can just bypass if you make your own handling, understanding every aspect also helps when implementing things. A network manager is not actually that hard, nor is network spawning, I think the structural decisions are the harder parts

To network spawn an entity, you just hold a shared idea of what prefabs can be spawned. Spawn one locally, give it a unique identifier and inform everyone. Now everyone has an object they can refer to as the same instance

Here's network object spawning in action, these boxes are not pooled and do not use VRC's networking just to be very clear. Networking transforms like this has some specific details I'll note below

https://github.com/user-attachments/assets/5165848b-c75f-476c-85aa-2fed392a1880

After spawning some boxes, the left client starts spawning gun network objects for the right client.
This specific network object gun is actually a gun and then a magazine which is network spawned and slotted in the gun within a frame. The gun itself has an event for firing, the magazine holds an ammo count as a synced variable (again, these are not VRC synced variables or VRC network events)

https://github.com/user-attachments/assets/8dd197fa-a31f-4364-9091-b071adb061da

Of specific note, the box shards (sub-objects) are actually pooled; the instantiation of that many objects at once would actually drop frames. I time-slice and maintain that the pool has some number of free box shards. The pool can expand because we have the ability to network spawn, it's nice (compared to udon networking where we would not be able to expand a synced object pool)

A special note on networked transforms, I use a sort of rate-limiting queue for them. This is to ensure players do not clog if they own too many moving objects. It results in discontinuous (teleporting) movement if it actively begins rate-limiting, but this is definitely better than network clogging

I use this style of "yielding" in a few places, things that are low priority can effectively yield/wait or limit themselves if they would otherwise overstep on outbound. VRC doesn't provide a great way to approximate this, so it's more like a switch

In the following video, the owner of a lot of boxes and box shards adds a large upwards impulse to boxes around them in a radius. The transform sync rate-limiting kicks in and the right client sees the results

https://github.com/user-attachments/assets/72e211e9-a958-47bd-a838-d096c85c11f7

Being able to spawn arbitrary objects at arbitrary counts felt very satisfying after being stuck in VRC's rigid networking. I paid the complexity once to make a general framework and it feels stable at this point, I don't feel the need to handle it carefully when interacting with it, which is a good feeling and probably a good sign


-----


placeholder related to custom networking in vrc, though i may not write a proper explanation for a while

after an initial version (which i call v1), i realized the need for a different approach. v1 handled network objects at network object level. their logic and state would exist on a derived network object. i learned that this was a mistake when it comes to sane structure and interacting with logic as a creator. network objects started to become monolithic and sometimes encompassed generic features that would be better suited as components in a composition style. i also used 1 udon/vrc network event per message, this proved to be extremely costly on metadata/header vrc side. when starting v2, the first thing i made was a way to bundle messages (basically structuring a packet) and handle them in a queued fashion. v2 also uses a network behaviour style, where network objects exist more as data holders/descriptors alone


why would you want custom networking when udon has networking?

first, it allows network spawning and despawning. this negates the strict need for rigid pools that hold all synced objects in advance (at editor/upload time)

pooling can be a pain to interact with, requires you route through a central source anyway to avoid race conditions, requires enough of every type of object in amounts that are enough for each player (kind of an n squared problem). It's also not exactly free for udon synced objects to just "exist" in an unused state afaik


more than that, it allows structuring logic in a client-server style which is much easier to account for than the typical distributed authority setup in vrc (this is actually the biggest benefit by far; you can do this in vrc, but structuring it in this way makes it much more clear-cut); we strictly require a consistent host player. logic reliant upon the host and many networking concepts will cease to work if the host leaves


a side benefit is the allowance of segmented serialization. udon/vrc networking does allow multiple synced components on an object, but a call to serialize will serialize all components on the object.
that means variables you set rarely (a name field that players can modify) must be serialized with variables you sync often (a position, a destination, etc.)

you can get around this in udon sync by splitting the synced components across multiple objects, but this has odd ownership handling across those objects and is more error-prone



late joiners are handled in a fairly specific way. first we snapshot all networked objects, then we pass it to the late-joiner over time. they queue session calls to process after they fully receive the snapshot. they continue to queue session calls until they get through all of them, enforcing order. conceptually not very hard, but it can be a bit difficult to structure and requires a decent understanding of how to order the things you want to do





