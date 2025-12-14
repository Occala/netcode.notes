I wanted to explain some things about world osc, which is kinda possible now since the introduction of world dynamics.

This is not practical, but is technically interesting. It's odd that VRC doesn't expose world-side osc to allow easier data relay

For one, if you have a compatible world and avatar setup, you can push data in either direction. To test this, I sent OSC data from a unity scene to an avatar, my corresponding world then read the avatar's input bytes. This sent around 10 bytes per command and required very explicit acks to align timing.
There's 1 byte of header denoting the data type, I decided I'd support up to 8 bytes of data and implemented doubles to test it. 1 byte was reserved for denoting the bytes as atomically ready to push (because osc sends are not atomic, we change the header, then the data bytes, then flag the send as valid with the trailing byte).

Data is interpreted via proximity to origin, the avatar holds senders that the world can read. Each sender is effectively a byte.

I thought it would be neat if I had my sending unity scene print the doubles it was randomly generating in a very similar format to the VRC world, so that's what I did to test it.

https://github.com/user-attachments/assets/12e522e2-d4c7-43eb-8a60-5bd03e0351ab

I tried a few other things, but the most interesting was a slider on my avatar. The intent was like a remote control that allowed me to control some volume in the world. Obviously this would be much easier if I just used world dynamics on a pickup probably, but I thought it'd be more interesting if the remote was on the avatar itself. It had both a radial slider for changing skybox color and a standard avatar slider for denoting volume. The standard slider could also be short pressed by a different finger to toggle the muted state of the audio source.
It follows a similar origin reading scheme for senders/receivers, so I won't detail it.

https://github.com/user-attachments/assets/3c85e9bc-79fb-49d4-80d3-3d65898eb295





