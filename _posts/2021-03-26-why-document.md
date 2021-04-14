---
layout: post
title:  "Why document what you know?"
date:   2021-03-26 00:00:00 +0000
tags:
  - Documentation
  - Ways of Working
category: Documentation
---

I've been thinking for a while about writing this post and some recent realisations in my professional life has spurred me on to finally do it.

This is just my view and my way of working. Your mileage may vary.

## Why documenting what you know benefits everyone.
So why document what you know?

As people in the tech profession, especially as part of a multi-discipline team (i.e. not just pure dev, but supporting customers and systems, fixing weird infrastructure issues, devops, etc), we are always learning.

```
"Every day is a school day"
```

It might be how to find a specific piece of information in a system you work with.

It might be figuring out a process to carry out a new task that the business needs you to do.

It might be figuring out how to fix an obscure issue on Legacy Service Y that crops up at seemingly random intervals.

It could be anything, we're always learning and as a result, we are always accumulating new knowledge that, if we are not documenting and sharing it, we are unconsciously hoarding it.

### So why is that bad?

The best way to explain this is using the concept of the [Bus factor](https://en.wikipedia.org/wiki/Bus_factor).

Put simply, `The bus factor is a measurement of the risk resulting from information and capabilities not being shared among team members, derived from the phrase "in case they get hit by a bus."`

So if you get hit by a bus, how screwed is your team or company?

How much knowledge has disappeared in to the ether as a result of you and a bus attempting to occupy the same point in space and time?

How many critical business processes are now exponentially more difficult or even impossible and how much time are your colleagues going to spend trying to figure out how to do all those things that only you knew how to do?

If the answer to these questions are `very` and `a lot`, your bus factor is 1. That is a bad place to be.

### Why don't we document?
There are many good and understandable reasons why we don't document things as much as we should.
* There's no time - When you're putting out fires one after the other, ain't no one got time to right stuff down.
* We don't have a good place to put the documentation.

There are also some really bad reasons not to document things.
* Self-preservation - If only I know how to do it, I am irreplaceable.
* Gatekeeping - I had to go to the effort to figure it out, everyone else can too.

Let's cover each of these reasons.

### There's no time
This is the easy excuse that we all tell ourselves and sometimes it is even true, but it is ultimately a cop-out, and I say that as someone who has said this to myself many times.

Documenting is hard, it takes time, I personally have often not documented something because I didn't have time to do what I considered a proper job.

But here's the rub. When everything is on fire and you're on holiday and your Legacy Service Y has broken and you're normally the one to fix it, your colleagues will not care if you didn't take the time to put in lots of pretty pictures, meticulously crafted flow-charts and long explanations of every step of a process.

If you've just put down a few bullets telling them where to click and what to enter when Legacy Service Y starts throwing random exceptions, they will thank you.

And so will future you.

### We don't have a good place to put the documentation.
This probably should have been the first item. If you've got nowhere to put docs, fix that first.

If you use Onedrive or SharePoint, create a shared drive and get everyone to sync it to their computers.

If you have Github or Azure DevOps or anything that has a Wiki or shared docs feature, set it up and tell everyone it is there and make sure everyone knows how to use it.

Every time you add something, tell your team, preferably via something that is searchable such as Teams or Slack. They may not look then, but in a few month's time when Legacy Service Y breaks, a light-bulb may go off in their head where they remember you posting that you'd documented the fix process and they can go and find it.

Now on to the not so good reasons for not documenting.
### Self-preservation - If only I know how to do it, I am irreplaceable.

If this is your mindset, ask yourself the following questions:
 * Would I like to have undisturbed holidays and weekends?
 * Would I rather be spending my time doing new and exciting things rather than just doing the same thing repeatedly?
 * Do I really think my place in my company depends on being the all-knowledgeable guru?

If the answer to either of the first two is yes, then by not documenting what you do, the main person who suffers is you. If you are too busy doing all these little jobs no one else knows how to do, you'll never have the time to do new and exciting things. 

Eventually, working on new things will go to other people because you don't have time and no one likes watching their colleagues working on the new hotness while they are stuck fixing Legacy Service Y for the one-hundredth time.

If the answer to the third question is yes, then I'd probably be thinking about if I really wanted to be with that company.

### Gatekeeping - I had to go to the effort to figure it out, everyone else can too.

You spent a load of time learning how to do all these things right? You spent hours Googling, experimenting, maybe even decompiling code and analysing stack traces.

Why should everyone else get the easy path?

Simple answer is that every member of a team having to learn something the long way is inefficient and a waste of everyone's time.

Once you know something, pass that knowledge on to someone else. Save them the pain you had to go through so they can put their efforts towards something more useful and productive.

If someone else had already solved the problem you spent a week figuring out, wouldn't you rather they recorded what they did so you could avoid repeating their pain?

## What to document.
Hopefully if you've got this far, I've convinced you of the benefits of documenting what you know. So how much do you document?

Simple answer is...there is no simple answer to that.

In broad strokes, I take a process or a task and document that in isolation.

In the last week, I have documented the following tasks that I do:
* Clearing website CDN cache - the easy way.
* Clearing website CDN cache when the easy way doesn't work.
* What build pipelines do we have? What is their purpose and how do I use each? What would I like to add to them in the future?
* Same for release pipelines.
* How to upload files to cloud storage to share with third-party support teams.
* Website failover procedure (both emergency and non-emergency procedure).
* New APM setup, what is there, what isn't, checklist of what we want to add.

The above covers a few different categories of docs
* How to do X.
* How X works.
* How we would like X to work in the future.

All of these add value in different ways, but importantly they all add value.


Don't let `perfect` be the enemy of `good enough`. If what you really need is a hundred page masterpiece to fully describe the ins and outs of a system, use the documentation of the individual tasks and processes as your starting point. Don't put off writing anything because you don't have the time to write War and Peace.

## Strategies for documenting.
This is my general approach. Sometimes I'll skip steps, sometimes I'll repeat steps. Do what works for you.

### Step 1
As I am doing a task, pop up notepad and jot down the steps. If the task needs images for the steps to really make sense, I'll sometimes use Word instead. These are just the quick-and-dirty temporary stores of this information, they are not the final form.

### Step 2
When time allows, look at what is there. Have I put enough for someone with no knowledge of any of the concerned systems beyond some basics you can reasonably assume to achieve that task?

If yes, great, add it to the Wiki.

If No, add it to the Wiki anyway with a note that it is not complete and make a mental note to carry out Step 3 next time.

### Step 3
Next time I do that task, I run through my own instructions, taking care to follow them to the letter and not doing anything that is not in the docs.

I will often find that I missed some steps the first time around, so add those in.

**I will sometimes repeat Step 3 a few times until I am happy that I can give this to someone else and they have a reasonable chance of following it.**

### Step 4
Next time that task needs doing, ask someone else to do it using your instructions.

Depending on the importance of said task, sometimes I will virtually look over their shoulder just to make sure I didn't miss out anything really important.

As they are running through, if they have to ask me a question, that means I have missed something and I will add that in.

### Step 5
Our documentation should now be at the point where anyone on the team with the aforementioned reasonably assumed minimal knowledge can do it.

If your team has a support rota, it can now be carried out by whoever is on support when the issue comes up, meaning they are empowered to be more effective in that role and you can focus on other tasks.

You can also now go on holiday and if this task needs to be carried out, your team can get on with it rather than adding it to your to-do list when you get back.

## So when am I done?

Never.

You are never done. There are always new processes, new problems, and new tasks that need learning and documenting.

You may have a new starter who doesn't have the basic knowledge you could originally assume was there, so you can now detail that and your docs will only continue to get better and more accessible.

Documentation is a continual process, but it is absolutely worth taking the time to do.

This may mean that a task that takes you 10 minutes takes 15 the next few times you do it. However, if that means your team don't spend 30 minutes trying to find you to do it next time and can just do it themselves, it'll pay for itself very quickly.


## Final words
What started out as a well-structured thought in my head has turned in to a long rambling poorly-structured blog post, but hopefully the point has still come across.

I could think of a number of other reasons to document, but that would only make the post even more rambling, so I'll leave it there.

Hopefully you've been convinced to start documenting more consistently. Your team will thank you, your company will thank you, and future you will thank you.