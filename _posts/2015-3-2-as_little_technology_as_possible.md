---
layout: post
title: As Little Technology as Possible
tags:
---

As Little Technology as Possible
================================

_or_ How to Turn a Pathological Hatred of Technology Into a Hackathon Victory
-----------------------------------------------------------------------------

This past weekend, my friend and I went to
[HackIllinois](http://www.hackillinois.org/) in search of fame, glory, and
ideally free T-Shirts. The event was great, the people were awesome, and there
were indeed free T-Shirts (I think I got 15 or so).

We decided that we would try for the reverse-engineering prize
[Affirm](https://www.affirm.com/) was offering, because security is cool, and
since we run a security club together, anything sufficiently cool that we built,
we could use in real life. We put aside our initial idea of "Tinder, but for
drones", and decided to build something like
[Codecademy](http://www.codecademy.com/), but for reversing. The general idea
was that the user could log in, and, inside of a couple clicks, start
interactively using real tools in a guided way.

We knew that the implementation would be tricky. We wanted people to work on
our project in a command-line environment, because we wanted things like
`strings checker | grep "password"` to work, but we knew that giving people
arbitrary command execution on our system could lead to some bad things. Like the
responsible security professionals we are, we solved this problem by pretending
bad things don't exist and agreeing to add jails or something at a future point.

Having started off with a decision as good as that, all we had to do for our
webapp was set up some kind of user system and have people SSH into it.
[GateOne](https://github.com/liftoff/GateOne) solved that for us, leaving our
most complicated web task using [jekyll](http://jekyllrb.com/) to build a static
landing page and letting us spend the weekend focusing on backend.

Backend engineering had some obvious challenges. Users had to each have their
own shell, have some kind of guidance, and ideally a "level system" that would
reveal more to them as they progressed. We considered and analyzed a wide
variety of options (Docker, Puppet, Ansible, Chef), all of which were designed
with problems like this in mind, but realized that both of us more or less hate
technology, and as such, would prefer to use shell scripts for everything.

As a result, our system was designed as follows:

  1: User logs in to an account called `jump_user`

  2: When they log in, the `.bashrc` file runs a script to create a new user
     account with a random name, put the appropriate files in it, and switch
     them over to that account.

  3: The user solves the problem.

  4: The user runs `levelup $PASSWORD`, and the same script as in step 2 creates
     them a new account with the new challenge files.

This system had both advantages and disadvantages. On one hand, configuration
was dead simple, and portability was easy for any system with bash installed. On
the other hand, we had as many as 4000 users on a tiny digitalocean box, which
was aggressively insecure thanks to random additions to the sudoers file and the
general aforementioned assumption bad things don't exist.

As luck would have it, people listened to our pleas of "please please please
don't be a dick you can totally own us but please don't", and the only people
breaking our system over the weekend were us. Once we had "solved" all of our
devops problems for the weekend, we could focus on user guidance. We wanted our
users to be at least sort of guided through the process, ideally in a way that
paid attention to what they were doing, but monitoring user activity is hard,
and once again our hatred of technology lead us to a simpler solution, that being
the `alias` command.

We accomplished our entire user guidance system with a combination of `echo`
commands in the aforementioned `.bashrc`s and aliases, as shown below.

{% highlight bash linenos %}
lsfun () {
  unalias ls
    echo "You see a list of files, reproduced below. You're slightly curious as"\
    "to whether the file called \`PASSWORD.txt' could contain anything you"\
    "might find useful, but you doubt it"
    echo ""
    ls
}
alias ls='lsfun'
{% endhighlight %}

This made for a quite interesting demo day. The judges typically asked "so what
technology did you use to build this?" and our answer of "shell scripts, mostly"
contrasted sharply with the more usual "Mongo, Angular, Express, and Node" type
answers, to the point of occasionally inspiring skepticism.

In the end though, our deliberate apathy towards anything invented after roughly
1970 paid off though, and (to our slight surprise) we won the prize, so thanks
to affirm for that.

For those interested, code from the event is scattered across the following
three github repos.

  * [gucci](https://github.com/japesinator/gucci)

  * [retro](https://github.com/japesinator/retro_site)

  * [retrogarde](https://github.com/japesinator/retrogarde)
