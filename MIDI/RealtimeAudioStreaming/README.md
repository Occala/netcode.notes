very placeholder for now, this was the most interesting technical application of midi i think

the premise was that i could push bytes into vrc via midi, so why not try realtime audio? like my own desktop audio for instance. i didn't end up doing that, but what i had was something that could take in an mp3 or wav file, break it into pcm data, send that as bytes into vrc, then create audioclips from the pcm data

i generally used low quality audio to facilitate realtime playback. i'd buffer some data, then when i had enough i'd start allowing clip assembly. i'd create chunked audio clips from the incoming data, while it was still coming in. i'd play them in a queue and keep adding to this queue, it was a little scuffed, but it was actually realtime playback of streaming data

the extra part, was that because i was pushing in low quality audio, i could just rebroadcast the bytes to other clients from my local client, via udon sync. they could assemble the clips in the same exact way. they'd be behind by latency obviously, so a little bit further back, but they were able to stream in the audio that i was streaming in. i really enjoyed this, so i might add some example videos of it in action
