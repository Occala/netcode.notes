Because I should write about this at some point;

Shifting origin (sometimes called floating origin, i think it's more technically correct to call this specific thing shifting origin though, people seem to get upset about the distinction)
is a silly technique to maintain the player near origin to mitigate floating point precision error as distance from zero increases

It's actually pretty simple, but in VRC especially there are some execution order related issues if you're doing it with vehicles. Nesting objects is also a slight concern

If the player moves more than some amount from origin on any axis, we shift backwards on that axis by the threshold we define. Let's imagine that's 1_000

Player 1 crosses the threshold on the x axis while using a vehicle, ending up at 1050, 0, 0 in terms of position

At this time, we shift objects back by this amount. This includes dynamic objects (other vehicles, the player's own vehicle) and things like terrain/environmental features.
We also increment Player 1's virtual position by 1 on the x component

I prefer interpreting the position of terrain personally, rather than doing an incremental shift each time (which i reserve for dynamic objects typically), because I'm wary of accumulating error simply from shifting the terrain around. In practice, this is probably not a large concern if you keep sane (whole number) threshold shifts and don't have an especially large environment.

Interpreting the position post-shift involves caching the absolute position of a static object at editor-time or early into your startup. Then, upon shifting, interpreting where it is relative to the local player's virtual offset. More on that later though

Player 1 ends up at 50, 0, 0 positionally, with a virtual coordinate of 1, 0, 0


If player 1 wants to inform others of their location, they advertise both their virtual offset (1, 0, 0) and their remainder in world space (50, 0, 0).
This is under the assumption of integer offsets, where each unit represents one threshold shift. You can also use doubles directly and represent the final absolute position over network, because it's sometimes easier to work with.

Now player 2, who is sitting near origin, is given a snapshot regarding Player 1. They themselves are at 0, 0, 0 in grid space and 0, 0, 0 in world space.
They know that the delta between Player 1's virtual offset and their own is 1, 0, 0. As a result, they initially describe Player 1's vehicle as being at 1000, 0, 0 based on the threshold value we use. They then add the remaining float components, resulting in a position of 1050, 0, 0. We want to convert to this relative position prior to working with the value, at which point we can treat it much like a normal positional target for the sake of interpolation or anything else

Again, it's pretty simple as a concept, I may need to describe the specifics related to VRC at some point though. The premise is relative positioning of dynamic bodies for the most part, while maintaining the local player near origin to preserve precision

There are some extra things that are needed if you're interpolating or extrapolating. Consider that you may be working with some interpolation target (positional); at the point of locally shifting, any targets in world space must be shifted back by the amount of a shift. Might be a bit confusing to explain

