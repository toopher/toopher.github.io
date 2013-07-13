---
layout: post
category: 
tags: []
author: Seth
---
{% include JB/setup %}

Travis CI is an amazing service for the open source community! CI, Continuous Integration, runs your tests on every commit, notifying
the team of any errors. This is a great practice that helps elevate the
health of your software, which (hopefully) leads to more green builds.
Travis does that for open source projects on GitHub, and it's rad.

Last week we hooked up Travis CI for a few of our repos, namely

 - [Toopher Ruby](https://github.com/toopher/toopher-ruby)
 - [Toopher Python](https://github.com/toopher/toopher-python)
 - [Toopher PHP](https://github.com/toopher/toopher-php)

## Quick overview

To enable Travis CI on your repo, you'll need sign up then create a `.travis.yml` file in
the root of your directory. 

Below is our [PHP setup](https://github.com/toopher/toopher-php/blob/master/.travis.yml). Note that we're using [Composer](http://getcomposer.org/) to manage dependencies and [PHPUnit](http://phpunit.de/manual/current/en/index.html) for unit tests (which are located in a `test` directory).

    language: php
    php:
      - 5.5
    before_script: composer install
    script: phpunit test/test_toopher_api.php

Pretty straightforward, eh?

## Pros and Cons

Should you use Travis CI? Probably, but I'll let you decide ;-) Below are some pros and cons.

### Pros
 - Free
 - Fast
 - GitHub integration, including cute embeddable `build passing` indicator lights
 - Popular and trending
 - Hosted, so you don't have to maintain an additional machine

### Cons
 - Only supports GitHub (no Bitbucket integration, no Mercurial support, etc)
 - Limited control
 - Limited extensensibility

## CI systems we evaluated

If you're like me, you learned about CI through the lens of Hudson (now
Jenkins). Jenkins is still the most popular CI system, but that is changing quickly. Here's an overview of continuous integration servers:

 - [Jenkins](http://jenkins-ci.org/) (formerly Hudson) - Powerful, heavily configurable
    enterprise class CI. There are a number of plugins to further
increase functionality. Lacks polish compared to newer CI systems. Hosted instances are around $45/month--you're really meant to download and administer it yourself.
 - [TeamCity](http://www.jetbrains.com/teamcity/) by JetBrains (makers of IntelliJ IDEA, PyCharm, RubyMine, and
other nice IDEs) - Seems like a solid enterprise product, which is to
say it's overkill for us. Hosted instances could cost $50 or more per
month--you're really meant to download and administer it yourself.
 - [Bamboo](http://www.atlassian.com/software/bamboo/overview) by Atlassian - Well liked by a couple people I know at Atlassian
shops. We are using a couple Atlassian products, so Bamboo would integrate easily. For better or worse, using Bamboo would further lock us into Atlassian. As with other Atlassian products the OnDemand version costs about $10/month.
 - [Drone.io](https://drone.io) - This looks promising. It's like Travis CI for business (instead of open
source). They support Bitbucket and GitHub. At $25/mo, Bamboo is
cheaper and it's more widely used.
 - [Travis CI](https://travis-ci.org/) - Travis CI Pro should be out anytime now. Rumor has it, Travis will support private repos then,
although it's only for GitHub (for now). Free for public GitHub repos.

## Your turn

If you have some time and you want to be pro, enable Travis CI for your
repo! Start with the [Travis CI getting started guide](http://about.travis-ci.org/docs/user/getting-started/), then the sky's the limits.

