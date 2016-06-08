---
layout: post
title: Keeping Track of Time Around the Globe
date: 2016-06-07 12:00:00 -0600
year: 2016
month: 6
day: 7
published: true
tags: time unix timezones IANA
summary: it's complicated
---

"_Even simple-looking things can sometimes have a
surprising twist in software. And everyone who thinks
that programming dates is easy to get right the first
time probably hasn't done much of it._" <br>
-- Peter van der Linden (Expert C Programming)

### Main Problem and Motivation

Recently, a colleague of mine introduced me to [World Time
Buddy](http://www.worldtimebuddy.com/), a convenient world
clock and time zone converter used to schedule online
meetings. I liked the application and I decided that
I would like to develop a clone of it that would work
locally with no internet connection and output the specified
time zone data as plain text in the command line.

So I made a quick utility that does this exact thing
(or at least I have some confidence that it does). The
problem was that I did not know how
does my computer have all this information about time
zones used by different countries. My mind was boggling.
I was trying to think of all the different edge cases that
can show up on my simple world clock and at the same time
I was wondering if my computer can handle them all by
default.

### What Can Go Wrong

There are many timing problems that can show up within
a single computer system (e.g., the RTC and the system
clock drifting apart from each other). Fortunately,
there exist mechanisms in the OS and hardware that
take care of most of them for us. The main thing I was
unsure of was the following: how does one go about making
an application that maps all (or at least most) locations
to a time zone and a Daylight Saving Time (DST) schedule,
if applicable, at any momment.

As an example, let's consider a clock that shows the
current time in Chicago(US) and Athens(Greece). Athens
is 8 hours ahead of Chicago normally and both cities
observe DST. The issue is that their DST schedules
are not synchronized. At the year of this writing, Chicago
turned their clocks 1 hour forward at March 13th  and it will
turn them back 1 hour at November 6th. At the same time,
Athens turned their clocks forward at March 27th and will
turn them back at October 30th. This means that between
March 13th and March 27th the time difference was 7
hours instead of 8 and the same will happen again between
October 30th and November 6th.

After thinking of the above example, I wanted to search
for other edge cases and get a better understanding of
the challenges of building a world clock. It
turned out that these edge cases are so many I can't
even list them all. For example, several areas recognize
partial offsets within a time zone (e.g., 30 and 15-minute
offsets). Another example is that there exist places
that are on the same time for half a year and an hour
apart for the other half because even though they are
all in the same time zone, some of them do not adhere
to DST. As one reads more examples that differ in
nature like the previous two, it becomes obvious
that these "irregularities" are not edge cases but the rule.

So the next question is the following: Even if we
were to store all these information somewhere, how
often would we have to change it? In other words,
how often do the different areas around the globe
decide to change their time policy?

It turns out to be more often than most people think.
To give an example, last year (2015) around 15 different
places (countries or areas within countries) changed
something about their time policy, either temporarily
for that year or "more permanently" for the years to
come. Chile abolished DST while Turkey delayed its DST
end time by two weeks and North Korea introduced its
own time zone.

### How Do We Manage

Most operating systems today keep the system time as
UTC and instead of having a single time zone set for
the whole computer they allow the user to configure
the time as they wish. In Unix-like systems, a user
generally sets the timezone during the installation
of the OS and can later vary the time zone of a
process using the TZ environment variable. This means
that the OS uses some kind of local database for
countries and time zones. On my Linux machine this
database exists in `/usr/share/zoneinfo/` and gets
updated during my system updates by my distributions's
updating mechanism/application.

This local database with all the timezone information
generally comes from the
[IANA time zone database](https://www.iana.org/time-zones)
(sometimes called OlsonDB) which is an ongoing
collaborative compilation of information about all
the timezone and DST cases intended to be used with
computer programs and operating systems. This database
does not only contain the current time rules in the
different areas, but also the history of time changes
from decades ago. This can be useful for applications
that focus on exact duration of events rather than
specific points in time.

If your OS does not ship with any of those
databases and has some other mechanism you can still
develop applications that use them. Many programming
languages in their attempt to be more portable,
generally ship with a copy of IANA or similar.

### Final Note

You can read more about IANA and the data of its data on its
[wiki page](https://en.wikipedia.org/wiki/Tz_database)
or learn more about the procedures of adding and changing
time rules in it by reading the archives of its
[submissions mailing list](http://mm.icann.org/pipermail/tz/).
[TimeAndDate](http://www.timeanddate.com/) is also a
useful resource to keep track of the changes in time
in the different areas.

The utility mentioned in the introduction that I've
been trying to write is incredibly simple considering
its requirements and therefore it suffices to
just plug it into IANA. There exist more complicated
groups of programs out there (e.g., vehicle or package
tracking) whose requirements may not be that simple.
In that case, the programmers should be more aware of
the pitfalls of keeping track of time in their software.

