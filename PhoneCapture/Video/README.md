Phones, similar to the still-image capture shown in this folder, but this time they capture video. This was incredibly cool to me, it resembles a very low-quality gif. This is propagated to late-joiners as it's held on buffers

It takes a while to push all of the data. I used raw pixels (again in a 565 bit format), wrote to textures, then used them like video frames; so it was not especially efficient. I limited the length of recording to around 10 seconds. Video playback was looping and aligned across all clients in a simple way via network time as a modulo source

The video may be a bit confusing so I'll summarize:

- Both clients start near client 2's phone, which is showing a video that pans upwards
- Client 1 rejoins, they go grab their own phone and record a short clip of client 2
- They both observe the phones for a bit
- Client 2 decides to rejoin, then walks over to view the video that client 1 had recorded

https://github.com/user-attachments/assets/0b3e50d7-8d7b-468a-a49d-245a36a6abc6

