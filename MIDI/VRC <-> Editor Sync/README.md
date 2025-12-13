this is a much older thing i worked on

First, a very old OSC sample that I made, which the idea evolved from. Changes in a unity client (separate to VRC) are relayed to an avatar in VRC over OSC.
I thought it was cool to edit something in a unity scene and push the changes to VRC. The scale component was a bit broken. It supported arbitrary object counts, through parameter cycling.
People do a similar thing better, in some modern setups, to save on synced params by maintaining saved and local avatar params that they cycle on to synced params; advertising them briefly.

Notably this used a prefab someone made to reassemble quantized positions in VRC (VRC unfortunately always limits avatar params to a byte, so float precision has to be reconstructed from multiple bytes effectively)

https://github.com/user-attachments/assets/ad7f2c02-a008-4300-a751-8e68066c4b26

At some point I started using midi to push data to VRC worlds, because that's even more interesting. But wait, we can read from VRC too with the debug log.
What if we could set up a pipeline, changes in a unity scene could be sent to a compatible VRC world, and changes in the world seen from the local perspective could be reflected in the editor scene. I thought it would be fun to try.
We just need a shared idea of which objects are which. I didn't know much about serialization or networking at the time, so it was pretty basic

The convoluted part is the pipeline probably, we don't really want to run a virtual MIDI driver in unity, I don't even know if that's possible (it probably is). I started trying standalone C# (outside of unity) for the first time.
This standalone app would act as the relay and logical portion, both the unity editor and vrc would just interpret the results.

So, changes in the unity scene were pushed over OSC locally to the C# app, the app then pushed raw bytes via MIDI into VRC. MIDI is a pain to work with like this, but it kind of works.
(i really don't recommend it)

Conversely, changes in VRC would be advertised over the debug log, the app would parse this and shuttle that data to the editor scene via OSC, completing the link in a way

But wait, how do we know if VRC saw it? Well, the app has to parse the debug log. VRC-side, via udon, we print to the debug log. This resembled simple acks. Additionally, if you push more than some amount of MIDI commands (it's around 100) to VRC, it simply crashes.
So the parsing is a necessity, to know when it's time to push more data and to know what the client has seen. There's a VRC/MIDI repository somewhere on github that has a decent example of this if you want to see an example

So here's how that looked, I put some cubes in the scene and started reflecting their transform changes

https://github.com/user-attachments/assets/8c7b24b3-007e-4dab-a1d7-227ce64feb54

Since I had a pipeline to send data about objects, I started trying to replicate other properties. Notably, color and collider state were good ones to test. The interpreting client can propagate the changes into the session (to other players) over normal udon sync, too

https://github.com/user-attachments/assets/f903494c-8ab9-423a-9acf-99908dae0a76

And an extra one with some more scaling

https://github.com/user-attachments/assets/da2aa651-7fc1-4448-bb98-e82d2c23970c

It was pretty rudimentary, but I think it was the start of my interest in serialization and some forms of netcode. Even now, I think it's still actually very technically interesting in VRC
