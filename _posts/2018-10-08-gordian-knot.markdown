---
title: "The Gordian Knot of Enterprise Architecture"
layout: post
date: 2018-10-08 12:01
image: /assets/images/markdown.jpg
headerImage: false
tag:
-
blog: true
star: true
author: masonrichins
description: A Description of the Problems Facing Fixing and Refacting Enterprise Infrastructure
---

## What is a Gordian Knot?

Sometimes I find myself on projects where the summation of hundreds of small technical decisions bring the project and its infrastructure to its knees. On a previous project, between banging my head on the desk out of sheer frustration, a coworker remarked that the project was like "facing a gordian knot". Despite knowing a decent bit about Greek and Roman mythology, I didn't immediately understand what he said, but on further research I realized that he was right on the money for that particular project, but also for a whole category of projects I had dealt with before. 

It wasnt until that observation that I realized this problem had faced me on numerous projects. This mental model improved my understanding of infrastructure problems and has helped me shape my approach to fixing enterprise architecture.

## The Legend Itself

Although the ancient Greeks were hardly reference material on enterprise software devlopment and operations, there are lessons to be learned from their stories. Their philosophy has proven durable of millenia, and we still use some of their mathematical innovations; there must be valuable learnings from their less technical writings as well.

The legend of the gordian knot itself goes something like this: 
An ox-cart (or chariot) had been tied with a famously impossible knot that confounded the Phrygians for generations. It was prophesied that the person who could undo this knot would be the king of all of Asia. It wasnt until around 333 B.C. that Alexander the Great came upon the Gordian Knot. Wanting to prove his might and intelligence, he attempted to untangle the knot, but failed like so many before him. In the end, he raised his sword and cut into the knot itself, breaking it into pieces. While not the conventional solution to untieing a knot, slicing it in half did technically solve the challenge. Furthermore, Alexander DID end up ruling a significant portion of Asia, so the prophecy held true to an extent.

While the vercity of the story of Alexander cutting the Gordian knot is up for debate (he, in fact, had very good publicists) It remains an early story idolizing the concept of thinking outside of the box to solve a seemingly intractable problem. 

## How to Know You're Facing a Knot

Most projects of a reasonable size will end up with a complex web of dependencies that increasingly makes changes harder to fix and failures more likely. This is just like accumulated tech debt slowing down feature delivery in software projects, but instead this problem manifests itself much more dangerously on the infrastructure side. If you ever go in to fix one problem, but a series of unrelated issues cause you to go on a bug fixing quest on the scale of the Odyssey, you're probably facing this problem. (also see this Malcolm in the Middle clip as a real world parralel: https://www.youtube.com/watch?v=8fnfeuoh4s8)  Simply put, as you pull on one part of the infrastructure knot trying to untangle it, you tighten the rope in other sections, or worse cause more issues.

Here are a couple more explicit signs that you are looking at a knot situation. 

1. One sign I see all the time, is a manual process is part of regular operations. The stated reason is generally that this particular task "not automatable". While it is certainly possible that there are some tasks that are outright impossible to implement in an automated fashion, this is really a smell that the design of the system does not facilitate automation well enough. There will always be parts that are extremely difficult to automate, but you should look to optimize the ease with which you can automate something. Now sometimes I do need to remind people, when dealing with authentication and secrets, that some human interaction is required to kick off a series of actions, and that decision taking should be the maximum amount of user interaction when it comes to configuration management and deployment. 

2. Another sign is an unsolvable "Chicken and egg" problem, where two supposedly indepdendent services are so dependent on each other they cannot start up independently of each other. In a world of so called "microservices" there are so many that will outright fail if something is amiss. Normally there is a way to get two co-dependent services up that involves bouncing back and forth between the two and failing at different points. But this process is still extremely error prone and full of opportunities for misconfigurations. And lastly this process will not get exercised often enough to catch new bugs.

3. And one last sign, and probably the most  is a high reliance on "Cowboy Coding" or "Cowboy Operations" where problems are fixed with individual heroism and quick thinking rather than rigorous process. Now, some of the best people I have ever worked with take the bull by the horns and wrestle with massive technical problems (or major bugs) by hand. But this solution just does not scale in any way. In fact it doesnt only not scale in terms of hours, but also in knowledge sharing; even the best technologists will miss details in wiki pages, or postmortems. It is inevitable. These knowledge silos dont just limit the ability of a project to have anyone pick up available work, but also puts the entire project at risk to the bus problem. While you can still work around these knowledge silos, they also have the function of slowing the overall progress of work, as all decisions need to be filtered through their vast knowledge of the current state.   

## Taking Alexander's Approach And Untangling the Knot

After running into roadblock after roadblock, taking Alexander's approach might sound tempting. However, I would warn against totally severing the knot in its entirety. The analogous approach is totally rebuilding your infrastructure from scratch. This might sound like a nice way to tackle your problems without any of the other constraints preventing you, and if you are able to accomplish this quickly, go for it! In most cases, though, the time cost of building a new set of infrastructure is large, and the benefits are only realized toward the end. The fundamental problem with this approach is that its impossible to incrementally deliver value while reimplementing everything from scratch. Instead, this problem demands a solution be perfectly innovative to solve the issue, while not affecting any of the other constraining factors. 

Looking at the examples from the previous situation:

1. Replacing a manual process with a scrupt that automatically handles the 80% use case. Even if the manual process is full of decision points, it is generally easy to make a script that makes decisions for you which is just a wrapper on the current manual process. If you know enough to take the manual process, or need to because this is an edge case, by all means do! But for people who just need to get stuff done, offering a script that gets you where you need to be is a lifesaver, and an important part of building automation throughout a project's infrastructure.

2. Automating around the chicken and egg problem using scripts. Now if these two services were ever stood up without each other, you know it is possible. You simply want to be able to reliably get the twinned services up and running using your automation tool of choice, and any implementation details on how that actually needs to happen should be suppressed until the fundamental issue can actually be solved.

3. Relying on your automation to manage issues in later environments. Making it a policy (and a desired end goal) that noone should be manually connecting to boxes, and instead should rely on your delivery pipeline to effect change. Now this is the biggest change to make, because it also requires an adjustment in the underlying culture. While this can be a scary change, since it is basically handing over control of the applications. 

Now these are just the first thoughts that come to my head, your situation is undoubtably different than this, but its important to look at building a solution that is disruptive within its own context, but consistent with your overall approach to fixing your accumulated infrastructure tech debt.


## Conclusions

Now you can see the importance automation has to the successful development and operations of an enterprise application, but the nature of each solution is that is radical per se, but not radical throughout the entire system. You will of course eventually move down the path of "doing things the right way". 

In all, you need to take a rigorous look at the problem and find a way to gracefully deprecate portions of the infrastructure while leaving others intact. Your approach might need to be as innovative as cutting the rope with a sword, or it could be simply leaning on the tools you have been using from the beginning. By looking at the problem holistically, you should be able to plan a dynamic approach to deprecate whatever bad implementation or habit is causing your pains. Unfortunately, there dont seem to be many solutions where there is a solution as simple as Alexander's, but its important to always be reminded to think outside of the box.
