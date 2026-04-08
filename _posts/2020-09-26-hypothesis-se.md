---
title: 'Some theories that I find practical in software engineering'
date: 2020-09-26
permalink: /posts/2020/09/hypothesis-se/
tags:
  - theory
  - devops
  - software
  - engineering
---

Today I will highlight a few theories that are well related to some of the extents of software engineering. "Software engineering" is a method that divides the whole development process into smaller modules and moves towards the solution through detailed analysis. Each of these small modules is the sum of one or more problems.

And one of the special things about "software engineering" is that here you have to think and study very well about both "problems" and possible "solutions". Because each solution will somehow create new problems, albeit in a very subtle way. Let's highlight the facts without exaggerating this time. Each theory will be mentioned with a link reference at the bottom of the post.

- "Something that can go wrong will go wrong" [1]
- "Simple things should be simple, complex things should be possible" [2]
- Cobra Effect [3]

The first theory that emerges for us is that any object that can behave erratically because it is not given enough attention will behave erratically, even at the most necessary time. Here are some examples - not keeping backups on time, not listing application bugs, not giving priority to fixing security loopholes, etc.

The second theory is actually much broader and does not need much detailed explanation here. In real life we all want complex goals to be achieved. The same is true when designing a software module but try to figure out how easy it can be. However, applying it in many cases becomes much more challenging. Even if a software technology/architecture is made simple you cannot always bet on the reliability. Things will anyhow get complicated.

The third theory seems quite interesting to me. I found this term when I was reading an article about marketing. The story behind this theory is that there were many cobra infestations in Delhi at the beginning of British rule. So the then government announced that instead of bringing a dead cobra, it would be rewarded. A few days later it was really noticed that the infestation of snakes had decreased considerably. As time went on, it became clear that the frequency of rewards increased again, which was unusual. After searching, it was found that some unscrupulous people used to breed and kill the cobras through cobra farming and take the rewards in return. The government then banned the reward program. Since there was no way to make a profit, the cobra farms were opened resulting in the presence of more cobra snakes than in the initial stage.

The essence of this theory is that when any method we apply to solve a problem exacerbates that problem after a certain period of time, it is called the "cobra effect". For example, if you fix a complex application bug without proper testing, and sooner or later if it starts leaking memory, will cause the application to crash. The purpose of the bug fix was to make the application better but it temporarily fixed the problem but in the long run, it made the problem even worse.

In addition to the above three theories, there are many other theories that are discussed in various fields of "software engineering". However, today's discussion will be limited to this.

- [1] "Anything that can go wrong will go wrong" - [Murphy's law](https://bit.ly/36575sR)
- [2] "Simple things should be simple, complex things should be possible" - [Alan Kay](https://bit.ly/3060ydy)
- [3] [Cobra Effect](https://bit.ly/3j65ruK)
