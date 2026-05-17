title: 'Writing Blogs From the Terminal — My Tiny Publishing Workflow'
slug: blogging-from-terminal-astro-github
publishDate: '9 May 2026'
description: 'A reflection on how I write and publish blog posts using Astro, Git, and the terminal and why I enjoy keeping the workflow simple and manual.'

## My blog lives in the terminal more than in a browser

Most people think of blogging as something you do in a rich editor like Notion, Medium, or a CMS dashboard.

My workflow is a bit different.

I write my blogs in plain markdown, inside an Astro project, and everything gets pushed through Git and deployed via GitHub Pages.

Most of the time, I never open a publishing dashboard.

I open the terminal instead.

## The flow is simple (and intentional)

My typical process looks like this:

* Write the post in a markdown file
* Preview locally with Astro
* Commit changes using Git
* Push to GitHub
* Let GitHub Pages handle deployment

That’s it.

No buttons. No “publish” UI. No distractions.

Just version control and a terminal.

## Why I like this approach

At first, it wasn’t about preference, it was just how I set things up.

But over time, I realized I actually enjoy it.

A few reasons why:

### 1. It feels closer to the work

I’m already in a technical environment most of the day. Writing and publishing from the terminal feels consistent with how I build software.

### 2. Everything is versioned

Every blog post is just a commit. That means:

* I can track changes
* I can roll back ideas
* I can see my writing evolve over time

It makes writing feel like development.

### 3. Fewer distractions

No dashboards. No formatting tools pulling attention away.

Just text and structure.

## Writing becomes part of my workflow, not a separate activity

One thing I’ve realized is that I don’t “go to a blog platform” to write.

I just write where I already am, in the same environment I use for building, testing, and shipping software.

That makes it easier to stay consistent.

If I have an idea, I don’t need to switch contexts. I just create a file and start typing.

## The moment I actually “publish”

The funny part is that there is no big publish button moment.

It’s usually something like:

```bash
git add .
git commit -m "new blog post"
git push origin main
```

And that’s it.

Somewhere in that process, the post becomes public.

No ceremony. Just deployment.

## Final thoughts

I didn’t build this workflow to be aesthetic or different.

I built it because it is simple, fast, and aligned with how I already work.

But over time, it has also changed how I think about writing.

It feels less like “publishing content” and more like shipping thoughts, the same way I ship code.

And for me, that makes blogging more sustainable.
