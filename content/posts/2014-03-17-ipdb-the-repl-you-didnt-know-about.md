---
Title: IPDB - the REPL you didn't know about
Date: 2014-03-17T19:03:00-00:00
Tags:
  - juju
  - python
  - debugging
  - amulet
  - video
Category: Programming
post_id: ipdb-repl-didnt-know
---

> What's a REPL you ask?  Its a programming language tool, and the acronym stands
> for Read Execute Print Loop. It gives you an interactive shell to execute Code.

Often times when debugging in python as a newbie you'll take the javascript approach of putting print statements all over your code. When it comes time to release the modifications and push upstream, its not uncommon to have a stray print statement left in there.

Another approach would be to drop into ipython (or python) and run interactive mode, keying in your statements or set %cpaste - and paste in the code. But in the instance of Amulet testing, this means we have to sit through another environment standup, or we've broken stride with our normal development workflow.

I set out to find a tool that was akin to ruby's **pry** debugger.

### IPDB - the debugger I didn't know existed

IPDB is a great tool. It's *exactly* what I was looking for.

It's a typical module include at the top of your file, and you call a method
 `ipdb.set_trace()` to act as a breakpoint in your code.

I recorded a quick screencast to illustrate the usage of IPDB and how it applies to the amulet test writing workflow.
