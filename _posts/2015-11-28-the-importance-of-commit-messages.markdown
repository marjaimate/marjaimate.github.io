---
layout: post
title:  "The importance of commit messages"
date:   2015-11-28 16:17:02
tags:  git commit history
---

# It's history for you

Commit messages are not a burden, despite how some software developers treat them. They are integral part of any software project, let it be a single-developer writing a tool, or a big team of people collaborating on a project.

You have to take into account that commit messages should be there to help you, should there be an issue with the software and you need to check the history. If the commmit messages are meaningless to you, then it won't help you (or the code reviewers) when going through them.

When you leave a project to rest for a few weeks or months, commits you've done with proper messaging and tagging become godsend. Not only it is easier to see what has been done, but each commit has a small story behind those changes.

# It's history for others

And never forget: Commit messages are for others too! Even if you start alone, at some point someone will join the company/team and they will start looking at what you've done before. In this scenario commit messages are the *first impression you make on them*. The clearer is your code history, the easier it is for anyone (managers included) to read them as a changelog. Or even for the lazy ones like me, use them to generate release emails to management.

A readable repo history is priceless. If you have to deal with a history like this, run:

![commit](/images/commits.png)

# Worst offenders

## Fix

Applicable to: one line changes, one character changes, or 200 lines changed. Nope. Not an option.

## Magic

OK, so you've done your "mad hax", coded up something while in the zone. DO NOT CALL THIS MAGIC. It's most likely *Work In Progress* at best.

## Bump

What? Really? Is it too hard to write: "Upgraded component X." or "Version number set to N.M"?

## \*Some swear words\*

Being rude is never good, especially in such a public forum as a repository's history. Don't even consider this option.

# My Checklist

* Stop and think for a few seconds on what you're about to commit. This is a small break in development, and a chance to properly finish the work you've just done.
* Never use one-word commit messages - see list above
* Try to make small commits, code changes that belong together. Avoid 'umbrella' commits, where loosely related changes are coupled.
* Attempt to describe your changes with a full sentence. Makes it easier to read for release managers, and creates and excellent source for release notes later.
* Tag your message if needed with bug number, issue tracker, etc. Creates a searchable history where you can plug in your project management tool(s) as well.
