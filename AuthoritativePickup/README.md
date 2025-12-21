I'll probably write about authoritative pickups at some point, because they're not the most straightforward thing to do predictively in VR

If you use the standard distributed authority model in VRC you'll encounter race conditions when players try to pick things up at similar times (or it could even be stolen, seconds later by someone dropping packets, there's nothing precluding that)

This is relevant in desktop games too (think picking up a prop in a source game); though I would say that the perceived response time is (in my opinion) more important in VR games



https://gafferongames.com/post/networked_physics_in_virtual_reality/

This article is actually a very good exercise on it, though the authority model is a bit different to ideal imo

It discusses ownership and authority as separate concepts, I find it more clear to think of them as soft-authority and hard-authority

A user with soft-authority can manipulate a pickup, but cannot make hard state changes or transitions (i.e. firing a weapon, changing an attachment, etc.), this is something you must logically account for, but I think it's worthwhile for responsiveness
