the name is really vague, but this was one of my more involved projects in terms of serialization; it's probably pretty boring to most, but i found it extremely interesting. So i'll explain some pretty technical portions. This might be very slightly relevant if you were trying to make a way to interpret your own scenes in VRC for some reason

a "slot" refers to something akin to a gameobject in resonite, but it's more of a synced container

in resonite (kind of a vr game with a lot of runtime creation elements, not quite a full unity editor though), players can create things together. They can work on the same entities at the same time, modify properties and nearly everything in the game is replicated

i wanted to create something like that in vrc, for now i'll try to outline how it works



All objects (slots) are defined as some base, synced type, these slots can be polled for their bytes. That's to say, each slot provides a serialization and deserialization method

Slots can hold "components" under them, VRC does not allow AddComponent calls, so this is done in a fairly specific way. Components are instead prefabs, added under a specific hierarchy within the entity and treated like components.

These virtual or abstracted components can also be queried for their bytes (they have a serialization and deserialization method), so if a slot is asked for its bytes, it provides its own, followed by all component bytes under it, in a rigid, ordered way

I'd also note, though it's not especially important for concept, that the physical slot object is actually a proxy. The synced version of the slot (really just the data and sync object) are somewhere, the proxy is somewhere else.
This allows us to actually set the proxy inactive without disrupting sync on the underlying sync representing that slot


The whole concept is runtime editing, more advanced than what would normally be the limit in VRC; the extent of which, is usually editing of baked objects that already exist in the hierarchy

Slots in this case do exist in advance (at least the data holder does), but the componentalization(?) of prefabs means they can be composed into new things.

Components have a few generic interface points, they can be queried for their properties. A "Cube" prefab might hold (in unity terms) a mesh filter, a mesh renderer and a box collider

The abstracted properties it contains are: Casts Shadows, Receives Shadows, Box Collider Enabled, and Color

These virtual components are referenced by index typically. You select a slot, then a component index (and then properties on that component by index). You can make multiple components on a slot. Slots can be nested

Slots, somewhat generically, encompass some common data related to both GameObjects and Transforms. They hold active state, name, Pos, Rot, Scale and ParentId.

Every property I've noted is synced, if someone selects a property field it is updated every frame, visually. Clients are kept at parity for any properties they view at the same time. If a player decides to modify a field, it is sent to a central player, who enacts the change on the property, the slot is then serialized. Clients then interpret the change: they initially purge all components. Then, spawn components as they see them come up in the byte block, followed by pushing the component's bytes into the component. They repeat this for all components in the byte block for that slot

I feel like i've failed to succinctly explain it, but there is a lot going on in a way. The result is that anyone can edit slots, or components, or properties of components.
These can all be edited at the same time as other players and by the nature of composing multiple basic prefabs together, more complex objects can be made


There is some heavy bit fielding that goes into the serialization bytes representing fields. Slots in the default state do not necessarily advertise all of their individual properties or default name for example. Unused slots effectively take 1 byte, as we pack them in order with *at minimum* their slot bit field (we explicitly do not skip entries, for the sake of knowing the slot order without explicitly saving an index)

I say packing, because all of this is capable of writing and reading in VRChat persistence, a persistence call looks like an iteration over all synced entities, to provide their serialization bytes. It should be no surprise then, that the entry point of startup from persistence looks like a byte block that is pushed into each entity. Versioning is kept on the outer most header of this byte block, once, to ensure upgrading components is possible and that we don't need to note component versioning individually or repeatedly

For the sake of data saving, a byte length is not prefixed to each entry. This is less safe, but saves on data across all available slots. A runtime error will not purge data at least
