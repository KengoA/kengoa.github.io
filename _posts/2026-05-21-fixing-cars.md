---
layout: post
title: "Fixing Cars"
date: 2026-05-21 12:21:00 +0100
category: essay
feature: true
---

During my recent camping trip to Wales, I noticed that my car boot was making an abnormally loud clunk when opened by the power liftgate. I called my Mazda dealer and was quoted 190 pounds for _investigating_ the issue, without a guarantee of a fix.

Having some time off between jobs and feeling somewhat outraged by the price, I decided to see if I could have a go at this myself. For context, this is my very first car which I bought earlier this year, and I didn't know the difference between a wrench and a spanner.

It turned out that this was a common defect, and Mazda had published a technical service bulletin [^1] for technicians, which I found through a Reddit thread [^2] from another CX-30 owner who had faced the same issue. Having gained enough confidence that this also applied to my car based on the diagnosis and the affected models, I took a very small leap of faith and bought a plastic panel remover and a 10-mm combination spanner on Amazon for a combined total of 13 pounds.

The actual root cause was that one of the three bolts on the liftgate was loose due to a manufacturing error. The fix was as simple as tightening it with a spanner, and then magically, the loud noise was no more.

With no prior experience in fixing cars or even knowing about car parts, this entire process felt like software problems I encounter every day. For a given problem I had, other people had already faced it and shared how they addressed it on the Internet, and I could refer to a public document from the first party to compare the details and find the actual issue that led to the fix.

Some of the hurdles you need to jump over to get to a solution like this are often less financial or technical than mental. After all, all it took was a small spanner to tighten a small bolt. Still, many people would rather pay other people to do it and not worry about this themselves at all. While this is more than reasonable, if you are the type of person who would not hesitate to open a small PR to fix a bug or even a spelling mistake in an open-source software dependency, I would recommend trying to treat a physical problem like this like it's a software problem. It might give you a sense of satistifaction of problem solving, and better yet, it might give you back some of the money you lost hosting your free software.

---

[^1]: [Mazda Technical Service Bulletin](https://static.nhtsa.gov/odi/tsbs/2023/MC-10232281-0001.pdf)

[^2]: [Fixed My CX-30’s Loud Clicking Liftgate – Saved $645](https://www.reddit.com/r/MazdaCX30/comments/1q1kqer/fixed_my_cx30s_loud_clicking_liftgate_saved_645/)
