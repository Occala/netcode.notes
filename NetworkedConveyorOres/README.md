I recently tried a game demo which featured physical conveyors/objects, a lot of my learning and ideas stem from thinking about how to network something I've seen

I've also done a fair amount of physical conveyor related work in VRChat already, so that part is trivial to do in a basic way, it's the networked portion that's important

In the past, I've typically made objects on conveyors local, they were only pseudo-synced in the sense that the rate of dropping objects on them was known by all parties



I started out just trying a simple sample where I'd network spawn ores when they needed to drop and despawn these ores when they reached some endpoint.
These ore objects sync their realtime state at some rate, they can also be picked up and dropped.
I already have something to drastically rate-limit realtime networked transforms if they try to push more than I'd like, so I figured I'd give this simple approach a shot

NOTE: I'll be referring to things like network spawning and despawning that aren't features of VRChat, I run custom networking in some of my more modern setups,
it resembles a toy network library. It inherently bundles messages and dispatches at some rate (20Hz in this case), which is the crux of some this being possible in the first place, so I guess just bear that in mind



https://github.com/user-attachments/assets/49b7af6d-2d77-47da-bdb6-f8d8523bc670

We can see in the video, it kind of works! The left client (host) propagates spawn and despawn events, ore cubes are synced in realtime and interpolated on the right client.
Though we can see it's somewhat floaty, due to the low update rate. Towards the end we can also see that the host is very nearly capping out on bandwidth. Maybe not so good?




Let's try with a higher spawn rate to trial something that might be more typical with more ores being mined across multiple belt entry points

https://github.com/user-attachments/assets/4d9cac4d-1e86-441d-bb5c-ebb817896a87

We increased the rate about 5x (which is an interval of ~0.15s), now the ores are stuttering quite a bit on the remote client.
The host is also capping out and rate-limiting these realtime transforms (which is causing the sad interpolation).
We really don't want to run the host that close to the cap to begin with and this approach isn't going to work




Let's figure out what's actually important:
I'm doing this in VRChat, I have to decide what to prioritize; I feel it's important the ores actually be physical in a (primarily) VR game
and players should be able to pick them them up and see them networked. If the game demo I mentioned was networking this, in the spirit of that game they'd certainly be networked ores.

- This writes off making ores entirely local
- Realtime sync isn't going to work in VRC when we're capped somewhere around 8 to 11 kilobytes per second (though I would consider something like this in a real game, it depends)



What can we do?



I've considered how factory style games might network objects on belts before, but I believe they often virtualize these and treat them more like data.
With the focus on physicality, we can't afford to style it that way




I decided that I'd try to network the spawn events only, run the objects on the belts locally across all clients, but realtime network them if they're doing something important.
"something important" in this case, is if they're picked up

So let's try:
- Propagating spawn and despawn events (this gives us a shared idea of which objects are which on the conveyor)
- Run the physics of the conveyor locally
- Realtime sync any held ores
- ADDITIONALLY, we sync drop events with their position, rotation, velocity and angular velocity. This allows all clients to play out a drop event and resume local physics


https://github.com/user-attachments/assets/13af96cf-1553-4374-93af-f324cb283884

It works! Right?

Even at our higher 5x testing rate, things are looking smooth, pickups and drops work decently and we're down to ~2500 bytes per second on the host during normal play (I'd also point out for clarity that around 1300 of this is from VRC networking, unrelated to what I'm doing, but I'll just be referring to that final displayed value)

We can do better though, can't we?

Instantiation is expensive, VRChat likely does some runtime checking across all components on prefabs you want to instantiate. Instantiation is how my network spawning operates normally though.
Additionally, it sends more data than is really necessary if we could pool the ores instead.
The data for a network spawn looks something like this:

PrefabId | NetId | OwnerId | Pos | Rot | Scale | ParentNetId


That's a lot of unnecessary data if we can get away with pooling instead, it would be much less expensive on cpu cost too.
We really only need something like:

ActiveState | Pos | Rot

With pooling we just need to ensure we interact with the pool instead when we would have spawned or despawned the object.
We also need to be a little more mindful of resetting state when it's pulled, resetting things like velocity too.


I've done a pool in my networked environment before, it's a very lightweight pool that ticks at some interval.
On tick, we check if we need to spawn more pooled objects to maintain a target count, spawning n network objects (typically 1) if we're below the target. We also prune if we're n above the target count,
this allows a sort of floating count that's typically a bit above our intended free count

Note that this only tracks free count, the pool doesn't actually know how many are active, it only maintains a single "free" list (idk if this is a good term for it).
When something wants to pull from the pool, the pool provides the last entry of the list. If none are available, it spawns the prefab in place and provides it. Returning an object to the pool just adds it to the end of the free list.
I've found it works very well for the intended time-slicing nature needed in VRChat where udon overhead is pretty large,
we can't afford to freeze a client by doubling the pool or something


This requires a networked component on our ore, denoting if it's active or not. If it's active, we additionally read the position and rotation (pool spawn location).
I also decided to add some seeding to this component, so each time we pull something it rolls a seed from the host and applies seeded scaling and color for some nice variation in ores

https://github.com/user-attachments/assets/817b97fc-3b28-4ee0-8add-769739ed422a

We can see the network object count is staying stable in the video, that's the pool operating. Additionally I scaled up the conveyors a bit,
but we're saving bandwidth compared to the previous example (which had around half the amount of objects), so that's nice



I added some minor fixes, changed the conveyor length and started focusing on a new mechanic. Ore splitting/grinding -
Ores in the demo game can be ground down into smaller chunks, that's kind of a cool thing to try. We can support that pretty easily with pooling too.
With instantiation, this would've caused pretty active frame drops as it requires making multiple chunks when an ore is split

We'll add an ore splitter on top of the conveyor (it looks like a thin white cube on the conveyor in the following video), when ores exit it from the host's perspective, they split into 2 or 3 ore shards, these are pulled from a pool managing those smaller shards.
Outside of that, they look very much like the ores in terms of components

https://github.com/user-attachments/assets/45692c60-5a34-4e67-bfd9-c49b760771c2

I think it looks pretty nice, this is the current state of the sample as of writing this.
Obviously desyncs can occur, either in physics or due to collisions caused by player interaction;
though I do have basic handling for the case of players smacking objects via some types of collision (this propagates much like a drop event, with velocity and stuff)




Scaling: This wouldn't scale at some point, in the last example we're already using something like 30% of the host's available outbound.
This scales with how many ores are being pulled from the pool, returning them is relatively cheap. But it would scale well enough to make something resembling a game sample


Late joiners: They actually work decently, we advertise objects at their current position to late joins, so settled objects off the belt will generally align.
Objects on the belt may explode a bit because of physics, I could probably mitigate this with more handling, but they receive a refreshed picture when the ores do a full run


Latency: What happens if someone is holding an ore when it's despawned from the pool? We can tell clients to drop ores they see being set inactive, that solves a small portion. Imagine though, if someone were to pick up an ore that's about to return to the pool and is subsequently reused, the host might interpret it as snapping to their hand just after leaving the mining machine/dropper. To handle this, I provide an incrementing identifier on objects pulled from the pool. This increments when they're returned to the pool and only uses 1 byte. If state updates (realtime or drop) do not match the synced identifier, we discard the state update. This should handle the case of a laggy client (or even a normal client) grabbing an object that returns to the pool and is pulled again by the host, before that remote client interprets it as being returned. (Hopefully that makes sense, basically we increment something when pool state changes and we use this to lock out stale data)


In regards to latency, if someone dropped frames or drops packets, they may end up spawning cubes on top of each other. This kind of ruins the local simulation, it may be preferable to queue spawns in some way or resimulate physics for the amount of time lost, but it'd be more complex (a queue/consumption style for pooled spawns would probably be the best option there)

Future:
- Settled object propagation for clients in session:
It's probably relevant to network ores that drop off conveyors and settle, to provide a more clear picture on settled objects that happen to leave conveyors more naturally from the host's perspective. We could probably infer this from triggers or collisions and some timing, then propagate the settled state.

