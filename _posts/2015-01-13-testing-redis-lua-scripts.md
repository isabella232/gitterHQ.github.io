---
layout: post
title: Unit Testing Redis Lua Scripts
date: 2015-01-13
---

![Coverage](/images/2015/Jan/coverage.jpg)
**Increase your test coverage!** (Photo credit: [John Proctor](https://www.flickr.com/photos/zabdiel/3028620509/))


At Gitter, we **love** Redis: it's an indispensable tool for us. Redis Lua scripts allow us to execute complex logic 
in a fast, consitent, atomic manner. Unfortunately, some of our scripts 
have, over time, grown pretty complex, increasing their [cyclomatic complexity](http://en.wikipedia.org/wiki/Cyclomatic_complexity)
and making thorough testing difficult. 

For this reason, I wanted to create a Lua environment which matches the environment available when running the
scripts in Redis. This allows us to:

* Write unit tests
* Use Lua debugging tools, like [ZeroBrane](http://studio.zerobrane.com/) to debug scripts outside of the Redis environment
* Use functionality not available in Redis while debugging, such as `print`. *Obviously you'll need to remove these statements before running the script in Redis.*

When running your tests in Lua, they're not atomic, so be aware of this.

<!--more-->

### Caveats

My knowledge of Lua is *limited*. In fact, before writing this test harness, every Lua script I've ever
written has run inside a Redis EVAL. There are almost certainly better ways of much of what I'm presenting here,
so please let me know if you have feedback.

However, it's probably the case that many Redis users have the same level of experience of Lua as I do, 
which is why I thought this post might be useful to others.

#### Dependencies

Firstly you'll need to install some dependencies. 

* **lua 5.1** (Redis currently uses lua 5.1)
* Some lua modules:
  * **luacov**
  * **busted**
  * **redis-lua**

If you're working on a Mac and have HomeBrew, you can install everything using:

* `$ brew install lua51`
* `$ wget -O http://luarocks.org/releases/luarocks-2.2.0.tar.gz`
* `$ tar -xvzf luarocks-2.2.0.tar.gz`
* `$ cd luarocks-2.2.0`
* `$ ./configure --lua-version=5.1 --versioned-rocks-dir` 
* `$ make build`
* `$ make install`
* `$ for i in luacov busted redis-lua; do luarocks-5.1 install $i; done`

Note: Luarocks is available in HomeBrew, but I had some trouble getting it to work with lua 5.1 
(the HomeBrew package installs it for 5.2), hence the custom build.

### The Redis Script to Test

For this example, we'll use an extremely simple script from this article at [RedisGreen](http://www.redisgreen.net/blog/intro-to-lua-for-redis-programmers/).

All this script does is increment one key and set a hash value.

{% gist suprememoocow/4e36e6f29ee07160d42c file-incr-and-stor.lua %} 

### Building a Test Harness

This code originally came from this blog article: [Make LUA debugging easier in Redis](http://www.trikoder.net/blog/make-lua-debugging-easier-in-redis-87/).

The harness exposes the `KEYS`, `ARGV` and `redis` global variables available to our Redis Lua scripts. 
If you use any of the additional libraries available to Redis scripts (documented [here](http://redis.io/commands/EVAL)),
you'll need to expose them as global variables too.

It exports a single function `call_redis_script(script, keys, argv)` which we'll be using in our tests.
It sets the global variables `KEYS` and `ARGV` and then executes the Redis Lua script, returning the result, for
us to perform assertions on.

{% gist suprememoocow/4e36e6f29ee07160d42c harness.lua %} 

### Writing Tests

For the tests, we'll use **[Busted](http://olivinelabs.com/busted/)**, a popular, easy-to-use Lua testing framework.

The unit test is fairly self-explanatory:

{% gist suprememoocow/4e36e6f29ee07160d42c test-file-incr-and-stor.lua %} 

To run the tests, run 

`$ busted -c test-file-incr-and-stor.lua`.

This will produce the following output:

{% gist suprememoocow/4e36e6f29ee07160d42c busted-output.txt %} 

### Coverage

The `-c` flag tells Busted to record coverage information. You can convert this into a report using `luacov`. 

Luacov will produce a report including all your modules, so it's best to give it a regular expression to limit the report to your redis scripts.

Be aware too that each each time you run your tests the coverage stats are added to cumulatively, so you'll need to delete the stats after each run.  

This command will generate the report, print it and then delete the stats file:

`$ luacov '^incr\\-and\\-stor.lua' && cat luacov.report.out  && rm luacov.stats.out`


This will produce the following output:

{% gist suprememoocow/4e36e6f29ee07160d42c luacov-output.txt %} 

Woot! 100% code coverage! Okay, obviously obtaining 100% code coverage for this 
script is trivial, but for more complex scripts, this technique can be extremely useful.

By using these scripts and increasing our test coverage, we've been able to debug quicker and
increase quality. Hopefully you'll find them useful too. If you do, or if you have any comments, 
come chat to us in **[gitterHQ/gitter](https://gitter.im/gitterHQ/gitter)**.

About the Author
=================

<img alt="Andrew Newdigate" src="http://www.gravatar.com/avatar/2644d6233d2c210258362f7f0f5138c2.png" style="float:left; padding-right: 1em">

__Andrew Newdigate__ is co-founder of Gitter, where he spends most of his days writing stuff in Node.js, Redis, MongoDB and any number of other technologies.

<a href="https://twitter.com/suprememoocow" class="twitter-follow-button" data-show-count="false" data-lang="en">Follow @suprememoocow</a>
