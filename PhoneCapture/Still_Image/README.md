I've wanted (for a while) to do a world-based camera with late-joiner support, this was my implementation of it when I felt I'd be able to do it. It's lazy pixels in a 565 bit format, so it takes a while to propagate, but the image quality is reasonable.

I made the phones player objects, so each player had one. They could store up to 3 images and cycle between which was being actively displayed. I made it sort of scan-line fill in the pixels incrementally, because it was more interesting and because it shows progress

https://github.com/user-attachments/assets/63d9f8ff-cbfe-4eae-8af8-6a17fdfebf0b

