---
title: Things I Learned Writing Trapifier.py
layout: post
---

Recently on reddit.com/r/hiphopheads, someone requested that a cover of a John
Mayer song, except with trap adlibs (i.e. gunshots, airhorns, "DAMN SON WHERE'D
YOU FIND THIS!?").  Since this was reddit, this request was obviously [quickly
fulfilled](http://www.reddit.com/r/hiphopheads/comments/1vxdag/guys_i_need_a_favor/cewq9ao)
and the result was as hilarious and awesome as one would expect.  In the
thread, people also posted a
[bunch](https://www.youtube.com/watch?v=Hu8yGh5aEwA)
[of](https://soundcloud.com/hennessyyoungman/cvsbangers)
[links](https://www.youtube.com/watch?v=LNT-b-yfM58&feature=youtu.be) to other
trap remixes of popular songs were posted.

I saw these and naturally thought it would be pretty fun to make my own trap
remix, but I A: didn't really want to play around in audacity and try to get
all the samples reasonable, and B: couldn't decide on just one song. 
Naturally, I thought to make a script that would "trapify" songs for me, which
would solve both my problems at the same time.  I sat down in front of vim and
got to work.

The actual code for trapifier.py is fairly simple, and it's all up on my
[github](https://github.com/japesinator/trapifier.py).  While a code analysis
could be done, in my opinion, a far more interesting analysis is that of the
design and distribution of the code.  Thus, below is my list of things learned
writing trapifier.py.

*   Python has libraries for *everything*

I was at first a little worried about actually handling the audio files in
python, since my knowledge of audio codecs and how to  manipulate them is fuzzy
at best, but a quick google search for "mix audio in python" turned up
[pydub](http://pydub.com/), which ended up solving pretty much all my audio
problems at once.  Later, when I was trying to make it work well from the
command line and failing miserably to get optional arguments to work like I
wanted, another google search turned up
[argparse](http://docs.python.org/dev/library/argparse.html),
which might be the single most useful python library I've seen since os.

*   Computer-timed isn't always better than random

When I first started the project, I thought that I would make the overlaying
function context-dependant, so that for instance when the song crescendoed, the
algorithm would place "bigger" samples.  But, as it turns out, this kind of
analysis is hard, and the changes in songs significant enough to be picked up
by my (admittedly clumsy) algorithms were so infrequent as to place samples far
too infrequently.  Instead, I chose to place a sample every 1.5-6 seconds and
hope that it would sync well.  As it turns out, this actually works about as
well as manual placement, albeit much faster.

*   r/hiphopheads has a surprising number of programmers.

For music-focused subreddit, the number of intelligent programmers on
r/hiphopheads is surprisingly high.  While there were some comments along the
lines of "You go to MIT?!?!" (because I MIT licensed the code) and "Super dope
script bruh. Algorithmically this is some e=mc2 shit," there were also a
surprising number of helpful comments about the actual code and what to do with
it.  People talked about refactoring code and using `os.path.join` for more
cross-compatibility and generally gave helpful advice about the design of the
program, something I did not see coming from where it did.

*   Windows is hard.

For reference, I don't actually own a Windows machine I can test on, so maybe
I'm exaggerating here, but it seems to me that it would take more time to get
the script to work on Windows (without python already installed) than it would
to write it in the first place.  At some point I should probably learn Windows,
as pretty much everything I do with computers I do worse on Windows, but that
point has yet to arrive.

*   Programming is really fun

I suppose I didn't really *learn* this, since I had known it before, but the
actual writing of the code for this project was one of the most enjoyable
things I had done in a while.
