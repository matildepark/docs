+++
title = "Gall"

template = "doc.html"
[extra]
category = "arvo"
+++

**Gall** is the application-management [vane](../vane). Userspace apps –⁠ daemons, really –⁠ are started, stopped, and sandboxed by Gall. Gall provides developers with a consistent interface for connecting their app to [Arvo](../arvo). It allows applications and other vanes to send messages to applications and subscribe to data streams. Messages coming into Gall are routed to the intended application, and the response comes back along the same route. If the intended target is on another [ship](../ship), Gall will route it behind the scenes through [Ames](../ames) to the other ship.

Gall is located at `/home/sys/vane/gall.hoon` within Arvo.

### Further Reading

- [The Hoon Tutorial](/docs/hoon/hoon-school/): Our guide to learning the Hoon programming language that will give you the foundation necessary for app development.
  - [Lesson 2.7: Gall](/docs/hoon/hoon-school/gall): A Hoon Tutorial lesson that explains how to write a simple Gall app.
- [The Gall tutorial](/docs/userspace/gall/gall): A technical guide to the Gall vane.
