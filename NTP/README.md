Clock sync! In VRC?

I think it sounds really scary as a topic, but it's super simple. It's like 4 numbers and you do a tiny bit of math and it gives you a clock offset and the round trip time

We align this towards the master, so an unfortunate consequence is that unless you align to something that should typically be close enough (like UTC), it requires you reset the sampling if the master changes. It may be possible to slowly drift the clock instead (this would be fine in UTC imo), but I didn't like how it felt

The actual difficulty I had in implementing this was in measuring what was a quality sample.
The wikipedia page will show you the NTP formula, but it won't necessarily give you practical implementation of quality sampling

Some basic rules can help you discard extreme outliers, exchanges with a RTT > 1 or < 0 are extreme, we discard these prior to anything else

I ended up using standard deviation for the primary filter.
First, we collect n samples (I typically used around 12). After collecting them in a buffer, we filter them.
Afterwards, we commit both the RTT and clock offset to working averages.
I used a moving mean style of averaging I believe, but EMA or even just real averaging can work

The shared timespace becomes localClock + clockOffset when you need to poll it.


VRChat provides a synced time source, so why wouldn't I just use that?
- I usually do
- VRChat only samples their network time once, on join. This is susceptible to clock drift
- Sampling once means the initial read isn't necessarily accurate
- It uses an older clock sync scheme afaik (i forget the name, but basically you send a packet, receive a time back, then measure how long it was from send to receive to estimate the RTT. targetTime = T2 + (RTT / 2))
- It can only resolve millisecond accuracy
- It wraps eventually, based upon the underlying server. So this can happen at any point, requiring the use of deltas only, you shouldn't really use absolute stamps
- It advances in realtime, this is the most dense topic on this page, but for brevity i will just say that is not great for network times actually

I use NTP schemes in worlds where I actually need the accuracy. Usually this is for entities that move either very quickly or require high precision, sometimes both
