---
layout: post
title: "&#127467; Notes from R in insurance 2017"
---

This year the fifth [*R in insurance*](https://rininsurance17.sciencesconf.org) conference was in ENSAE, Paris. The first impression was: "Wow, that's a lot of people. Much more than the last year", and I hope my estimation is not biased. Thanks to organizers that was, indeed, a true pleasure to be on both sides, as a presenter, as well as a speaker. I really love that unique atmosphere and mixed audience: not many conferences offer the feedback from the academic and industry perspective at the same time.

![](https://irudnyts.github.io/images/posts/2017-06-14-r-in-insurance-17/rinins.png)

Writing this post I had fair options: either to cover the whole conference or one particular talk. I prefer the second case, as a most efficient, from my view. The first plenary talk was given by Julie Seguela (Covea). She shared with us her experience with textual analysis of expert reports to increase knowledge of technological risks. Well, the last sentence I copied form the official conference page, so let me describe it in simple words. 

Given is a database of reports of claim event description. Typically such reports are managed manually, by a human being. Julie proposed a way how to extract insights by using dozens unfamiliar for me packages. Very briefly, in simple words, for each report, one need to get read of the [stop words](https://en.wikipedia.org/wiki/Stop_words), and then apply such parsing techniques as [stemming](https://en.wikipedia.org/wiki/Stemming) etc. Finally, using black magic and playing around with machine learning, one can cluster and classify such reports. I think my summary is too snippy to give even a little taste. However, what I still can remember is a really good question/remark from the audience: leaving away stop words, unfortunately, the sense of the sentence can be changed. For instance, "Fireman come in time" and "Fireman does not come in time" after the tidying will have the same words, but completely different meaning. 

I was really excited by the talk, and cannot wait enough to put my hands on Julie's code. Perhaps, in the next post I will try to cope with the issue mentioned by one from the audience. 
Again, many thanks to organizers, expecially to Christophe and Markus. Looking forward to the next year *R in insurance*.


