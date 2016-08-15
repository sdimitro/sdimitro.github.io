---
layout: post
title: "Notes on Testing: Legacy code in illumos-core"
date: 2014-12-22 12:00:00 -0600
published: true
tags:
 - illumos
 - testing
 - legacy code
 - wsdiff
 - warlock
---

Last Friday, Robert Mustacchi shared
[a great guide](http://illumos.org/books/dev/) that new developers
can reference when working with the illumos build system. As I was
reading it, I came across the section on
[testing changes](http://illumos.org/books/dev/workflow.html#testing)
in the basic
workflow chapter. In that section Mustacchi gave some general guidelines
on how a developer should decide the way in which their changes should be
tested.

Recently on illumos-core, I was working on an
[issue](https://github.com/gdamore/illumos-core/issues/10)
that had the goal of
removing some legacy code and required me to delete content from many files
(around 293). After I made my edits and did a full nightly build, no errors
showed up in my logs and I was able to boot into my new system.
Garrett D'Amore reviewed my code to make sure that nothing suspicious was
going on.

The problem was that many of the files that I had changed were drivers/systems
that my system didn't even use (random device drivers or SPARC-specific files).
Since I am fairly new to illumos and kernel development in general, I had no
idea how to test these specific changes. Luckily, D'Amore pointed me to the
[wsdiff(1)](http://illumos.org/man/1onbld/wsdiff) utility.

As you can see from the man page, if you keep your proto area before applying
your changes, you can use it with wsdiff to find differences in the binaries
between "before" and "after" your changes. One thing to remember is that DEBUG
builds tend to carry debug macros which emit line numbers (e.g `__LINE__` macro)
so if you run wsdiff between two DEBUG builds to test your changes, be sure that
some, if not all, differences will come from there. (Check option -d of wsdiff
for other relevant info)

Taking the issue that I mentioned above as an example, running plain wsdiff
after my changes, a report with a huge amount of differences was generated,
due to line numbers. Thus, I had to do two full non-DEBUG builds to make sure
that my binaries stayed the same after my changes. Apparently it is also
possible to invoke wsdiff through [nightly(1)](http://illumos.org/man/1onbld/nightly), looking at Example 4 from
the wsdiff man page.

Besides my case-study of removing legacy code though, I believe that wsdiff
has its place in testing for other situations as well. Examples can be changes
that touch a lot of files or have to do with libraries. Basically, any change
that risks changing binaries that you are not supposed to, by mistake.

**EDIT**: A big thanks to Nick Zivkovic for pointing out mistakes in the initial
version of this post.

**EDIT 2**: I suggested adding a section about this to the original guide.
The above information is now
[part of the dev-guide](https://github.com/illumos/dev-guide/commit/3fcf78337e4177a811013a4845166b58f76d2038).

**EDIT 3**: The original title on my old Wordpress website was
"Notes on Testing: Nuking Legacy code at illumos-core"

