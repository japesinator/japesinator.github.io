---
layout: post
title: Circle City Con CTF Writeup
tags: writeup
---

This past weekend, I made it out to Indianapolis to have a great time with some
great people at [Circle City Con](https://circlecitycon.com/). I made it out to
j0hnnyxm4s's excellent training, and caught a few talks, but, as has been the
trend at the past few cons I've been to, I spent way too much time playing the
CTF. What follows is a poorly organized set of quick writeups for all the
challenges my team got.

PHYSEC
------

I cracked locks one through four, and had a lot of fun. I learned how to
top-torsion, and that I had, in general, been way over-torsioning locks since
I learned how to pick. Lock five may have been broken at some point, as one of
the pins was sliding up and down depending on the angle, and all the locks after
that took significantly more lockpicking skill than I have, but the category in
general was a ton of fun.

Open Sesame
-----------

I actually had essentially nothing to do with this category, as reversing is not
a thing I am good at. I was told `strings` was used pretty heavily.

Forensics
---------

We actually swept this category! The first challenge was a reference to the
`4C444E` that appears on the pyramid on the opening page of the CTF, which is
hex for "LDN" or [La Dosa Nostra](https://twitter.com/ladosanostra). The eye of
of providence is the eye in the triangle, and the flag was right below that.

The next challenge was a pcap that took us way too long to find. The data was
just someone pinging google over and over again with the occasional spotify or
dropbox sync There was absolutely nothing anomalous about the packets except
for the actual data within them, and after only roughly 15 hours of analysis,
or team managed to look at the data in a packet using wireshark and find the
flag.

The next forensics one was called "The Fabric of Reality", and between that, the
clue about following the thread, and the announcement of "THERE IS A CRYPTO
QUILT IN THE CTF" we managed to figure out the quilt was related. After some
futile googling related to civil war slave quilts, grid ciphers, and spirals,
a tweet surfaced with three people pointing to their own name. With a few
characters in place, frequency analysis and a solid knowledge of movie quotes
got the rest of the flag.

The 50 point crypto was actually, in my humble opinion, the most over-valued
challenge in the whole con. Two equal-length nonsense ciphertexts in a crypto
completition are just crying out to be xored, and that just gives the flag
straight to you.

FindMeIfYouCan
--------------

I honestly expected this category to be significantly more social-engineer-y,
so when the first challenge was just EXIF, I was a bit surprised. There were
two dates in the image, which threw me off, but in the end, copy-pasting from
`strings` saved the day.

The next three challenges we actually did not get, so I can't speak to those,
but the fifth was quite interesting. A .wav turned out to be morse code to a
goo.gl link, but, morse code does not preserve case, and there were actually two
valid links of that string, modulo capitalization, which threw us on a day-long
false flag hunt fixating on a nissan micra manual on a freecycle site. Once we
figured out the actual link though, we quickly got the flag.

The first 40 point challenge was pretty easy. A quick exploration on twitter
revealed that the "Frank" mentioned was actually Frank the Tank, from LDN of all
places. Some more googling and twitter recon found his github, and then from
there it was just a bit of looking through commits.

The second 40 was considerably harder than the first. At one point, I actually
acquired part of the actual man in the picture's number, and had to take a
moment and avoid actually doxing someone for a CTF. As it turns out, it was in
reality much simpler than all that, and, looking in the EXIF again for strings
formatted suspiciously yielded a different phone number, which worked well for
our purposes.

The third forty I couldn't figure out, so I won't postulate on it, but I did
manage to get both the fifties. The first asked for a street address, and so, we
fearlessly delved into google maps, being all at least relatively familiar with
Chicago, and lined up the skyline with the picture until we found something that
looked not-wrong, plugged it in, and it worked!

The second fifty was actually a ton of fun. Harassing j0hnnyxm4s enough gave us
the email address "johnnyxmas@yahoo.com", and then viewing the yahoo calendar for
that account initially turned up nothing, but a second calendar could be found,
and, on the EXIF date of the meeting, an event with the flag was simply marked
on there.

Cryptology
----------

The first crypto challenge was an ascii string with "13" in the title, which is
generally a pretty solid indicator of ROT13, and was here.

The next challenge, "What's in the Box?", was entirely mysterious to all of our
team except one person, who immediately went "oh that's javascript!", solving
the challenge and also validating my decision to avoid webdev whenever possible.

The next challenge we solved was "This type of stuff will rot your brain," was
another simple cipher, this time monoalphabetic substitution instead of ROT13.
Throwing it into [quipqiup](http://quipqiup.com/) and then forcing the first
word to either "It's" or "He's" pops out a movie quote, which is easily
googleable, and the character referenced by said quote is the actual flag.

The final crypto challenge we managed to solve was the 40-point challenge, which
we got by virtue of the French string given as a clue. Translating it and poking
around on wikipedia eventually found us something called the Mary Queen of Scots
cipher. Just plugging this in gave us the flag, nicely formatted and everything.

All in all, it was a wonderful CTF and we had a great time. Thanks Circle City
Con for putting it on, and we're going to enjoy the hell out of those Derbycon
tickets.
