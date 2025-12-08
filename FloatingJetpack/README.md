Placeholder for my jetpack related stuff, originally made for VR purposes of vehicle testing. I found it unsatisfactory to test extrapolation / dead reckoning with the vehicles I could find, plus vehicles aren't really my thing

Primarily, cars and air vehicles typically move too quickly to work well within the sorta small float precision available in unity. Consider that a jet can cross 4km in less than 20 seconds.
We could limit them to this small range, but things like jets aren't especially good at vectoring in the way we'd want, it's hard to test them outside of side-by-side flight.
Nowadays I do sometimes use shifting origin and sync in a separate coordinate space, but I don't know if I'll go into that much just yet.

So this is to say that I started making a physical jetpack for the purpose of testing sync. It lead to a lot of the concepts I now use in what I'd consider my modern style of extrapolation.
I prefer to call it dead reckoning sometimes because it's more clear what I'm actually referring to, to people that understand.
The premise is to position an object as close to its current realtime position as possible, despite latency, despite update rate.
You could call it extrapolation, but it has other associations in netcode and beyond.

It's controlled by positioning the hands relative to the jetpack body; the triggers control the amount of thrust, which emits from the palm.
I'll leave a video here for now to showcase what it looks like, but I'd like to expand on the steps it took to make and eventually expand on what extrapolation actually looks like (concepts, etc.)
to achieve something that's positioned at everyone's relative realtime "now", but looks reasonable, despite constantly mispredicting past data.

