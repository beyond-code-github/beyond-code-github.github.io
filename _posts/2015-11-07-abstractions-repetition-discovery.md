---
layout: blog-post
title:  The Pit of Fail - Abstractions, Repetition and Discovery

tags:
- pit-of-fail
- abstractions
- discovery
- code
---

Hands up if you've been in this situation; you are building out a handful of new features that happen to share certain facets. To the customer they are distinct deliverables, but to you and your team it's a rinse and repeat type scenario with maybe one or two tweaks in each case.

The first one is straightforward - you add a view, a controller and so on. The second feature is much the same, view... controller... perhaps you spot some code or markup that can be pulled out into a dependency and re-used between the two.

Then we come to the third feature. You sit back and think to yourself - "these are really all variations on the same theme. I could introduce an abstraction to simplify this. Adding the remaining features afterwards will be really quick, and really simple."

Stop right there. Chances are you're about to make a grave error, and I'm going to explain why.
<!--more-->
#### Don't repeat yourself

I'm going to bring this to life with a quick example. The fictional project we're working on is for a school. They need to manage all sorts of lists of things which we are going to build from scratch. The backlog looks like this:

* SCH-001 - As the admin assistant, I want to be able to maintain a list of teachers
* SCH-002 - As a teacher, I want to be able to maintain a list of students in my class
* SCH-003 - As the head teacher, I want to be able to maintain a list of classes we teach

Edited for brevity, but you get the idea - we need to implement some lists. We're working on this as a team so it won't neccesarily be the same pair picking up each story.

First time around, we end up with some markup looking something like this:

<script src="https://gist.github.com/beyond-code-github/43df3b474234162f501c.js"></script>

The next story gets implemented, and looks very similar:

<script src="https://gist.github.com/beyond-code-github/367b446a51d83f19d136.js"></script>

Now the pair who picks up the third story takes a look at the implementation so far and identifies quite rightly that we've got some code re-use that needs attention. Both teachers, students, and the new classes list are all going to have a name and a notes field, as well as the configuration for the datatable and the boilerplate for the form.

In the spirit of making that smallest possible change that brings value, our pair introduces an abstraction that will eliminate the code repetition and also improve their ability to build any future lists at the same time.

<script src="https://gist.github.com/beyond-code-github/a663d86e6eb72cef7819.js"></script>

No more re-use... all we need to do is ensure we render this view for each list. Nice, right?

#### Write once, Maintain many

Lets take another look at our backlog again though. There are no more list stories present... in fact there are no more stories in the backlog at all. So the ability to build future lists quickly is - to the best of our developers knowledge - completely redundant.

This is a good example of when we should apply the You Aint Gonna Need It philosophy (otherwise known as YAGNI). Sure we've done a good job of applying Don't Repeat Yourself (DRY), but we've undertaken work that we didn't need to, and in the process we've taken up time that could have been better applied bringing more immediate value. This isn't great in itself, but actually another error has been made here, one that's much harder to spot.

It doesn't really matter how many list-based stories are on the backlog, neither does it matter how many lists we may end up having to write in the coming weeks. The fact is, the time spent writing the lists is always going to pale in insignificance compared to the amount of time we spend maintaining that code over the coming months and years.

Code is by definition write once, maintain many, and so it stands to reason that time spent increasing code quality and maintainability will almost always be more valuable than time spent reducing future keystrokes. As a rule of thumb, you should never implement an abstraction purely to save you time coding future features. That's not to say you shouldn't aim to improve you methods, but you should make sure to balance it with other concerns.

#### Logic vs Intent

But wait, there's more. By focusing so much on DRY we've lost a key feature of our code - discoverability. If I'm new to a codebase or a feature and tasked with maintaining the code, the first thing I do is this:

<img class="center-block" src="{{ "assets/img/abstractions-repetition-discovery/ctrl-shift-t.png" | prepend: site.baseurl }}" alt="ctrl-shift-t" />

The html/cshtml page is usually the entry point to a feature, so once we've found that the rest of the code is easy to trace.

Post refactoring though, all that happens is this:

<img class="center-block" src="{{ "assets/img/abstractions-repetition-discovery/ctrl-shift-fail.png" | prepend: site.baseurl }}" alt="ctrl-shift-fail" />

What's happened is that instead of being easily able to find the entry point to our students list, we have to know ahead of time that this feature is driven by the ListController/html file. We've removed some explicit knowledge from the codebase and made it implicit. Developers (and other stakeholders) now either need to be provided with this information from a colleague, or they need to take the time to discover it for themselves.

What a lot of people don't realise about DRY is that it's intended to mitigate the damage caused to maintainability by repetition of logic. In this case our views don't contain any logic, they contain intent. The views tell us what the page does, and what features are present.

However much we refactor intent, it never goes away. It just ends up getting moved to less accessible place. In our case, it ends up in the controller or the model:

<script src="https://gist.github.com/beyond-code-github/c3f88f173994e0623dcd.js"></script>

#### The leaky abstraction

You may be quite surprised by the amount of problems we've introduced already with such a bog-standard, everyday refactoring. But there are still more beasties lurking in the depths. Lets fast forward a few sprints in our example where we pick up the following card:

* SCH-021 - As a teacher, I want to be able to upload a list of students provided to me by the school administrator.
Now we need to add a feature to one of our lists but not the others - our use cases have diverged. So once again we make the simplest changes to satisfy our acceptance criteria:

<script src="https://gist.github.com/beyond-code-github/ba00446b167453097d2d.js"></script>

Spot the problem here? We've introduced a leaky abstraction. By virtue of our code re-use, a concern that would otherwise have been specific to one feature has now leaked into code used by another. This in turn increases the chance that we're going to introduce bugs into unrelated parts of our codebase.

You might be thinking "this doesn't seem so bad, there's only one conditional and it's still better than it was before with the duplication". But the danger is after a few more stories we end up with configuration that looks like this:

<script src="https://gist.github.com/beyond-code-github/1154b2264a0e5e2a4a95.js"></script>

This kind of approach can get away from you very quickly; it's ugly, the configuration is seperate and far away from the view that it relates to, we've introduced yet another level of indirection. To top it all off, all it really does is save us keystrokes writing the next list which as we've discussed is not a common occurance across the life of the project.