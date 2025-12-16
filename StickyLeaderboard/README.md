this will be more conceptual

VRC lacks a way to save data to a world. I'll probably write something about persistence itself at some point, but this is about the idea of a propagated leaderboard. The data becomes very sticky, as it's aligned across any players who encounter it

Each player holds on persistence, the top n entries, pre-sorted by score

This is obviously unsafe (in the sense that players could cheat or spoof entries if they really felt like it, it would probably require ongoing maintenance), but is very technically interesting. It's probably much more safe to use string-loading and manual entry submission for a safe leaderboard if that was the intent

For my setup, I decided to record entries with a name, timestamp and score. Limiting it to 1000 entries, people who encounter data must check it against their own, sorting it by top 1000 entries (if that many even exist)

Newest timestamp wins if there are duplicate entries for a player when resolving incoming data (session update) or from wholesale updates (taking in another player's leaderboard entries, to subsequently sort into your own)

I treated incoming checks in a queued fashion, they could be iterated over multiple frames if needed until the data was clear (to prevent slowdown from checking many entries)

Displaying the data was more of a pain than setting up the data filtering, I ended up using a scrolling UI which understood where the scrollbar was and displayed n entries at that portion of data, to avoid actually holding 1000 ui elements at once
