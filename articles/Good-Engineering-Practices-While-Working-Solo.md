> * 原文地址：[Good Engineering Practices while Working Solo](https://blog.bitsrc.io/good-engineering-practices-while-working-solo-ad872e727af4)
> * 原文作者：[Shalvah](https://blog.bitsrc.io/@shalvah)
> * 译文出自：[阿里云翻译小组](https://github.com/dawn-teams/translate)
> * 译文链接：[https://github.com/dawn-teams/translate/blob/master/articles/.md](https://github.com/dawn-teams/translate/blob/master/articles/.md)
> * 译者：[靖鑫](https://github.com/luckyjing)
> * 校对者：[]()

---

# Good Engineering Practices while Working Solo

When you’ve got to go it alone, how do you make the most out of it?

Most developers work as part of a team. However, at some point in our careers, we’ve had to (or we’ll have to) work alone. And while much of product development involves being able to manage or work with the rest of the team, it’s equally as important to develop good practices while working solo.

## A quick note about “working solo”

Solo typically means doing something alone. But we’re using it in this context to mean anything that’s done by a small number of people in an unofficial, unstructured environment. It could be just you, or you and a few other folks. Examples include:

* An open-source project, such as a package or library
* A personal product, which could be commercial or free
* A freelance gig for a client

The common thread here is that there are no established rules like you’d typically have in a company.

## Why should I bother about my engineering practices

Here are a few reasons why they matter:

**You’ll be an asset to your team.**

Let’s consider the possible scenarios that could ensue when you join a team:

* You join a team that follows the practices you’re used to. Great! It means you’ll be ready to start contributing from day one.
* You join a team that follows a different set of practices. If you’ve grasped the general concept of good engineering practices, it shouldn’t take you too long to get used to it, and you’ll become productive soon, too.
* You join a team that doesn’t follow any good practices (oh no!). In this case, depending on the team, you might be able to bring your knowledge to bear and improve their processes. Otherwise…maybe just leave.

Either way, it’s a win-win.

**You’ll be a better developer.**

Software engineering is about more than just coding. There are lots of moving parts involved in bringing a product from conception to launch, and then keeping it running beyond that. Inculcating best practices will help you keep on track and avoid frustration.

If you love coding (like me), there’s always the temptation when working on something new to jump right in and start coding. But I’ve seen over time that having some kind of structure in place, while still maintaining the flexibility that comes with working solo, helps me to avoid many bumps in the road.

Let’s consider some of these good practices.

## Stick to a workflow

A workflow is the series of steps that are involved in transforming an idea in your head to a finished product. Some of the things your workflow should cover are:

* when a change is decided upon, what processes do I follow to implement it?
* how do I deliver new versions of this product to the end user?
* how do I track changes made to this product?
* how do I track bugs, problems, and future plans for this product?

**Why?** It might seem faster to get things done without a defined workflow (“just write code and push to master”), but as a project increases in complexity, you’ll find that having clear definitions makes it easier to determine what needs to be done and to focus on it. When working on a team, the workflow becomes a pipeline that helps each member be more productive.

**What you can do:**

* *Use issues* (GitHub, Gitlab, BitBucket or wherever your code is hosted). Issues help you keep track of bugs, feature requests and major changes to the project. Also, writing out an issue title and description force you to crystallize the thoughts in your head and define exactly what the problem is and what the solution should involve. You can also take this one step further by adding project management tools like Trello and GitHub Projects.
* *Use pull requests.* Many devs prefer to push directly to master when working alone. However, there are benefits to making changes via pull requests. It’s easier to see all the changes involved in that feature or bugfix and how they’ll affect the codebase when merged. Also, when pull requests are paired with issues, it makes it easier to observe how a project has evolved without having to read the code and figure it out.
* *Review your code.* It might sound strange, but you should review your code like it was written by a different person. Some folks suggest making a PR and walking away for a few minutes or hours, then coming back to review the code before merging or updating. Code review, away from your IDE where you are king, allows you to see your code with (somewhat) fresh eyes, helping you to catch bugs and identify things you forgot. Code review also forces you to question yourself. Why did I implement this that way? What could go wrong? Is there a better way?
* *Maintain meaningful Git history.* Try to commit like you would if you were on a team — avoid large commits, keep commits focused, meaningful commit messages. Like code review, writing a descriptive commit message forces you to slow down. What was I trying to achieve in this commit? How did I attempt to achieve that?

There will be instances where you can allow yourself to break the rules. For instance, you might decide that for really minor fixes (such as correcting a typo), it’s okay to push directly to master, in order to avoid slowing things down.

Regardless of what you incorporate into your workflow, it’s key to be intentional. Successful products don’t just happen; they’re built with love and care. Being intentional means stepping back, looking at the pain points you face and thinking of ways and tools to improve them.

## Compose reusable components and modules

Think in accordance to the DRY, SOLID and FIRST principles. Compose software out of smaller, encapsulated, atomic reusable components. Use tools like Bit to create a collection of your favorite building blocks and keep them updated. You’ll end up building better software much faster.

## Write documentation

Ugh. Documentation. Many know of it, few do it and fewer still like it. After the exciting rush of writing code, it’s often a struggle to document it. How do I capture all the intricacies of my code in prose?

**Why?** Humans are fallible. We’ll forget things, we’ll have sick days, or we’ll move on from a project. Documentation ensures knowledge isn’t tied up in a human. Documentation also helps developers to get the big picture and stay focused.

**What you can do:**

* *Realize that you’re not writing a book.* Documentation doesn’t have to be a literary masterpiece. No one is grading you. Don’t try to explain everything. Keep it concise and clear. Sometimes a couple of bullet points is all you need.
* *Document before coding.* Document the interface of your product — how it will work from the outside. This serves as a spec and will guide you when building.
* *Document after coding.* Again, write like you’re an outsider looking in. What is important? What do you need to know to use this (or contribute to it)? The spec may change while building, so it’s important to check that any documentation you wrote before coding is still correct when you’re done.
* *Document while coding.* This mostly applies to writing documentation in code. There are many articles on documenting code, so I won’t go into detail on this.
* *Work in units.* All of the principles above apply to units. For instance, a unit could be a function, a class, a new feature, a change in behaviour, a module, or an entire product. If documenting a piece of work seems extremely difficult, try breaking it up into smaller units (or alternatively, going up the hierarchy of units).

## Communicate

Communication mostly applies when working with a small team or for a client.

**Why?** Transparency leads to accountability. When you know you have to report your dealings to someone (be it your peers or your boss), it helps you to remain focused. This is one reason why many companies have standup meetings.

Another reason is that so often, one member of the team runs into an issue that could be easily resolved by communicating with the client or another team member. I’ve lost count of the number of times I’ve yelled in frustration, “My changes aren’t showing up on live”, only for another team member to tell me, “Oh, I think I made some changes that might have nullified yours”.

**What you can do:**

* Let your team and client know when you’ve run into unforeseen snags.
* Give your client regular updates on the progress of the project. Try to keep these updates from getting too technical, though.
* Let your team know when there’s been a change in plans.
* Remove obstacles to communication so everyone can easily find out what anyone is working on. Find and use whatever tool works best for you — WhatsApp, email, Slack, and so on.

Essentially, keep the feedback loop alive. It removes a lot of friction between all involved parties.

## Monitor

**Why?** Things will go wrong. Deployments will fail, mistakes will be made, exceptions will be thrown, things will break. It’s important to have provisions for monitoring so that you can be better equipped to tackle these failures. Monitoring is another part of the feedback loop; it prevents you from building in an ivory castle, unaware of how your product is actually performing.

**What you can do:**

* *Capture logs and metrics.* Don’t be ashamed to console.log() where necessary. It’s better to log too much information than too little. However, be sure to avoid polluting the logs with unnecessary details, so it’s easier to sift through the logs.
* *Know where your logs go and set up a system to easily view them.* This could be something as basic as SSHing into the server to tail a log file or something as advanced as sending your logs into the ELK stack. But it is important that when you write console.log() (or whatever means you use to log), you know where to look for those logs.
* *Don’t swallow errors.* While your application should be resilient (able to recover in the event of an error), it’s a good idea to log errors you don’t expect. For instance, if you’re calling an API to fetch user-created content (such as tweets), you should be ready to handle cases of 404 Not Found responses. But a different, unexpected error from the API should be logged. I’ve been in such a situation, and because I wasn’t logging these errors, I didn’t know that I had exceeded my rate limit, leading to my app being blacklisted from accessing the API.
* *Check logs and metrics you’ve captured.* This can be either manually or via automated means. I’ve been in situations where I’ve implemented a “simple” fix and deployed it and went on my merry way, only for me to later realize it wasn’t working. Since then, I picked up the practice of monitoring application logs for a while after a deploy to verify that things were working as expected.

Monitoring strategies can take different forms, from simple console logs and text files to external providers like Sentry, Bugsnag and Elastic APM. Pick a stack that suits you, and go with that.

## Observe and iterate

This is a best practice as well as a guide for all the rest. There’s no one-size-fits-all formula that works for everyone. People will get used to different workflows, monitoring strategies, documentation styles, and so on. This is why it’s important to always observe and iterate.

Observation involves watching behaviour or performance critically. With observation, you connect what you see with what you know and reason out the facts from those. When working solo, you typically don’t have the benefit of advanced case studies and A/B tests, so you pick up clues from “informal” sources such as people’s comments, issue reports, and logs.

After observation comes iteration. Iteration is about making improvements to your product in response to observations, and then observing again, and so on. After making your observations, the next thing is to come to conclusions, then implement them. But it doesn’t end there.

An example scenario: I have an app that displays a list of items and the time they were created. This time is in UTC, though, so for many people using my app, the time shown is wrong, and it often gets them confused.

I decide to fix this by adding “ UTC” to the display (so it shows “5:30 pm UTC”, for instance). This works, and there’s less confusion. But I eventually realize, why should I make the user do the work of converting to their time zone? So I change it to convert the time displayed to the user’s timezone. Much better.

After talking with people who use my app, I realize that the most important thing for them is knowing roughly how old an entry is, not necessarily the exact time. In response to this observation, I push out an update that changes all times to relative times (“5 minutes ago”, “2 hours ago”). The time accuracy is gone, but it makes things much simpler for my users.

This applies to your internal processes too. None of these guidelines is set in stone. As you choose practices, it’s important to observe what’s working and what isn’t, and iterate based on that. Keep making changes until you find what works for you.

## Conclusion

Ideally, in a structured product development environment, there are many different specialized roles that play important parts, from the product owner to the developer. When working solo, it’s important to realize that you’re filling many (possibly all) of these roles, so you have the freedom to run things as you wish. It’s best to use that freedom to create a structure that makes you more productive. It might involve a little extra time and effort, but I assure you it’s worth it!

阿里云翻译小组