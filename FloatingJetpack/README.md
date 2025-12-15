Placeholder for my jetpack related stuff, originally made for VR purposes of vehicle testing. I found it unsatisfactory to test extrapolation / dead reckoning with the vehicles I could find, plus vehicles aren't really my thing

Primarily, cars and air vehicles tend to move too quickly to work well within float precision of a standard setup. Consider that a jet can cross 4km in less than 20 seconds.
We could limit them to this small range, but things like jets aren't especially good at vectoring in the way we'd want to facilitate testing, helicopters tend to be too heavy-feeling, cars are difficult to make.
Nowadays I do sometimes use shifting origin and sync in a separate coordinate space, but I don't know if I'll go into detail on that style of thing yet.

So this is to say that I started making a physical jetpack for the purpose of testing sync. It lead to a lot of the concepts I now use in what I'd consider my modern style of extrapolation.
The premise is to position an object as close to its current realtime position as possible, despite latency, despite update rate. The prediction portion is trivial, so what's the catch?

Obviously we can't just position on the prediction each frame or it'd teleport at each new snapshot arrival; the real art becomes resolving the past misprediction into a new prediction.
You could call it extrapolation, but that has other associations in netcode and beyond (client prediction, interpolation, general data);
I prefer to call it dead reckoning sometimes because it's more clear what I'm actually referring to, to people that would understand.

This type of sync is not as common, but it is very relevant in certain types of games (flight sims are probably the most prominent type where this would be necessary); they each tend to roll their own bespoke solutions as far as I can tell, with little commentary on how it's done.

There are separate concerns that may not be immediately apparent, timing is a large factor. Network timing, as well as frame-timing become critical for something I'd consider "correct", but there's a lot of room in the space; there's not an overwhelming single/best solution.

The jetpack is controlled by positioning the hands relative to the jetpack body; the triggers control the amount of thrust, which emits from the palm.
At some point I'd like to explain the steps it took to make and eventually expand on what extrapolation actually looks like to me (concepts, etc.);
to achieve something that's positioned at everyone's relative realtime "now", but looks reasonable, despite constantly mispredicting from past data.

I'll leave a video here for now to showcase what the floating jetpack looks like, though I don't think it really captures what the user (me!) is doing to control it

https://github.com/user-attachments/assets/e5440e54-c2c0-4fa3-a2a3-77d988a1dd7d

And another that actually shows some of the hand motion involved

https://github.com/user-attachments/assets/35f60632-2e18-4438-93d3-fdb425323bbf



