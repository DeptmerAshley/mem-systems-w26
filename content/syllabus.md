+++
title = "Syllabus"
[extra]
hide = false
+++
# CS/ECE 4/599: Modern Memory Systems

(Please first read the formal syllabus on [Canvas][syl].)

This is an undergraduate/graduate level, research-intensive course in computer
systems, with a focus on techniques and technologies that enable today’s
high-performance memory systems.  

## Communication
We will use Discord for course communication, and [GitHub](https://github.com/khale/mem-systems-w26/discussions) for online paper and lesson
discussions. You can find the Discord server sign-up link in a Canvas announcement. 

## Class Sessions
Check the [schedule][] for each day of class. Class sections will typically (though not always)
involve a first part where I lecture, and a second part where you present papers and we all
discuss them. 

You are expected to do the relevant readings **before** class so that everyone can participate. 

## Work You'll Be Doing

* *Paper reading:*
 We'll be reading a lot of research papers, and we'll be discussing them both asynchronously online (GitHub/Discord) and syncronously during class discussions. 
* *Blog contributions:*
  You'll be writing summaries, critiques, and discussion notes in the form of blog posts for each paper we cover in class. This will be a team effort. 
* *Course project:*
  The biggest part of your grade will be an open-ended, ambitious, research project that culminates in a codebase, a presentation, and a final blog post. This will also be a team effort. 

### Paper Reading & Discussion

Most of our in-class time will be spent discussing research papers. 
When we read a paper, you will need to participate in online and in person. 

#### If you are the paper leader

- One week before we discuss the paper, create a discussion thread for the paper in GitHub [discussions][paper-disc].
- Watch the discussion thread and answer any questions you can 
- Work with your lab to assign the roles of blogger/reviewer and scribe.
- Present a few slides (~10 is good, more is okay) on the paper and lead the in-class discussion. If the paper
already has a slide deck available online, feel free to use that, or a subset of it. 
- Gather everyone's work from your lab no later than one week from discussion day into a blog post on the [website github][gh].
  Make clear everyone's roles. You submit the blog post via a pull request to the github site.

**Everyone should be a paper leader/co-leader at least once.**

#### If you are in the presentation group/lab

- **Blogger**: submit a summary of the paper (and your interpretations/opinions/thoughts about it)
  to the paper leader. It should include
  * Background you think the audience needs to understand the paper
  * Detailed summary of the main contributions of the paper. Don't just summarize; include your thoughts
  * Important results and what they mean
  * Context of where the work fits in and why it's important for the field. 
  * Identify strengths and weaknesses of the paper (both those you've identified, and those
    identified by others in in-class and online discussions)
- **Scribe**: Take good notes of the in-class discussion and submit to the blogger to include in the blog post

**Everyone should try to take on each of these roles (even as a team of more than one) at least once.**

#### Everyone
Participate in the online and in-class discussions. Once the paper leader has created the discussion thread,
give your thoughts on the paper. This does not need to be long, just a few sentences is okay. You can 
also respond to others' posts, and this counts as well. 

### Research Project

You will do a systems implementation research project that will involve a significant amount of
systems building and coding. This can be any open-ended and open-source project you choose that
is related to memory systems. This could involve OS, compiler, hardware, and programming language work. 
The final product is a blog post, your code, and a presentation. 

You can either work individually or in groups with up to 2-3 people. 

#### Proposal 

You must first submit a project proposal. This is due on Friday, January 16 (at the end of week two). I
will post a list of project ideas soon, though you are free to come up with your own ideas. [Open a GitHub issue][proposal]
answering these questions:

* What will you do?
* How will you do it?
* How will you empirically measure success?

You should also list the GitHub usernames of everyone in the group.

After you create the issue on GitHub, submit its URL to the "Project Proposal" assignment on Canvas. I will give
feedback on GitHub on the project. 

[proposal]: https://github.com/khale/mem-systems-w26/issues/new?labels=proposal&template=project-proposal.md&title=Project+%5BNUMBER%5D+Proposal%3A+%5BTITLE%5D

#### Implementation
Create a plan for implementing the different parts of your system. Large systems will always have to be
broken down into smaller pieces. Build the skeleton code first, then work from there. 

#### Evaluation
In your evaluation, think about:
* What benchmarks will you use?
* How do you test for correctness? 
* How will you measure success? What will your figures of merit be?
* How will you present your results and findings? 

You can look to SIGPLAN's [empirical evaluation guidelines](https://www.sigplan.org/Resources/EmpiricalEvaluation/) for
hints on how to do this well. 


#### Final Report

For the final report, you'll summarize your project's outcomes, findings, and results in the form of a post on the [course blog][blog].
Your writeup should answer these questions in detail:

* What was the goal?
* What did you do? (include both the design and the implementation.)
* What were the hardest parts to get right?
* What was surprising?
* Were you successful? (Report rigorously on your empirical evaluation.)

You are welcome to optionally include a video to go along with your blog post.

To submit your report, open a pull request in [the course’s GitHub repository][gh] to add your post to the blog.
In your PR description, please include “closes #N” where N is the issue number for your proposal.
The repository's README relevant has instructions.

[schedule]: @/schedule.md
[syl]: https://canvas.oregonstate.edu/courses/2028992/assignments/syllabus
[lessons]: @/lesson/_index.md
[blog]: @/blog/_index.md
[paper-disc]: https://github.com/khale/mem-systems-w26/discussions/categories/paper-discussions
[gh]: https://github.com/khale/mem-systems-w26
