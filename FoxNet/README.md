placeholder related to custom networking in vrc, though i may not write a proper explanation for a while

primarily, allows network spawning and despawning. this negates the strict need for rigid pools that have all synced objects in advance (at editor time)

more than that, it allows structuring logic in a client-server style which is much easier to account for than the typical distributed authority setup in vrc

a side benefit is the allowance of segmented serialization. udon/vrc networking does allow multiple synced components on an object, but a call to serialize will serialize all components on the object.
that means variables you set rarely (a name field that players can modify) must be serialized with variables you sync often (a position, a destination, etc.)

you can get around this by splitting the synced components across multiple objects, but this has odd ownership handling and is more error-prone.
Though it does technically have some slight atomicity implications for components on the same object, but this would be odd to rely on cross-component.
