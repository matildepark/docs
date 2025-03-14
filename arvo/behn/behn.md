+++
title = "Overview"
weight = 1
template = "doc.html"
+++
Our simple timer.

It allows vanes and applications to set and timer events, which are
managed in a simple priority queue. `%behn` produces effects to start
the unix timer, and when the requested `%behn` passes, unix sends wake
events to `%behn`, which time routes back to original sender. We don't
guarantee that a timer event will happen at exactly the `%behn` it was
set for, or even that it'll be particularly close. A timer event is a
request to not be woken until after the given time.

`%eyre` uses `%behn` for timing out sessions, and `%clay` uses `%behn`
for keeping track of time-specified file requests. `%ames` should
probably use `%behn` to keep track of things like network timeouts and
retry timing, but it currently uses its own alarm system.

This article is a stub. For more information, see either `behn.hoon` or [Lesson 2.6 - Behn](/docs/hoon/hoon-school/behn) of the Hoon tutorial.
