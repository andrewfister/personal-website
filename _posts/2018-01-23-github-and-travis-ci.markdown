---
layout: post
title:  "Github and Travis CI"
date:   2018-01-23
categories: update
---
I've added the source files for my website to Github [here].
Now my site's source code is backed up and versioned. In addition, I've
added some minimal "continuous integration" via [Travis CI]. With
this service, each time I commit new code to Github, Travis will detect
it and build the site, and then check it with [html-proofer]. If I see
there is an error when Travis runs this, then there is a problem with
the site. Both Github and Travis CI are offered for free for public
projects such as this one.

[here]: https://github.com/andrewfister/personal-website
[Travis CI]: https://travis-ci.org/andrewfister/personal-website
[html-proofer]: https://github.com/gjtorikian/html-proofer