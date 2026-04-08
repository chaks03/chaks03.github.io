---
title: 'Using post-mortem report as a medium to build trust across cross-team members'
date: 2020-10-17
permalink: /posts/2020/10/postmortem-trust/
tags:
  - communication
  - devops
  - sre
  - software
  - postmortem
---

We know that "post-mortem" is a term that is widely used in various criminal investigations. By analyzing the various subjects of crime and mentioning them in the "Post-mortem report" helps the mystery of crime to be revealed by conducting a thorough investigation on them. Today we will discuss how this "post-mortem report" plays an important role in modern software development as a medium of better communication across cross-team members.

Let's think of a simple scenario without going to a high level. Sometimes it may happen that when a new software module is released, it is seen that the software service is not as stable as before. The application or service begins to behave inconsistently. But yes, one thing we must keep in mind is that any inadequately tested software code, incorrect configuration of application runtime and dependencies on the server, or inadequate server resource (CPU, RAM, storage) design can also cause the software service to fall into such situation. It may also happen that the software system is relying upon third party services. And when the third party service is down, the software service stability becomes uncertain. If this circumstance keeps repeating again and again, then the number of software users will continue to decrease to an extent, which will eventually increase the business losses. At one stage, of course, the software development, quality, testing, and system operations team must come to terms with top management. And if there is no proper documentation and communication among the team, then there is a possibility of huge disagreement which is inevitably very terrible. One solution to this crisis is the "post-mortem" report.

**"In the midst of chaos, there is also opportunity" - Sun-Tzu, The Art of War Illustrated**

A postmortem report is a template that contains a detailed description of software misbehavior or software service downtime. The parts of the whole report are - Title, date, downtime status, summary, impact, detection, evidence, root cause, resolution, mitigating the problem in the future, and contributors. However, some parts may be optional. The early part of this report is presented in a very general way so that anyone from the non-technical team can understand the report at first glance. Another benefit of writing this report is that it makes you aware of the limitations of the team involved in the whole incident. Limitations may include - Team member crisis, lack of training, miscommunication, lack of good service monitoring system, etc. And if these limitations are clearly manifested, the top management has to be ready to come forward to solve them.

You might think that this "post-mortem report" is overkill. Its not actually. Because with a "post-mortem report" you can create a "blameless" culture. This will increase the trust among the team members towards each other. And "trust" is always blended in a successful teamwork.
