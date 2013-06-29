---
layout: post
title: "Keeping Your Git Forks In Sync"
date: 2013-06-29 11:00
comments: true
categories: [Git, Tricks, Hints, GitHub, Forking, Sync]
---

## Keep Those Forks In Sync

Have you ever tried to submit a pull request to a project on Github to only get the comment.

 > Hey your code can't be merged can you get latest and resubmit?

If you didn't have a fork, which is not exactly a branch, but looks like one. But instead had a branch

	git rebase -i branch_from_whence_you_forked

Since you do have a fork, you look on Github for the *Sync My Fork* button and find none.

## Solution

After dealing with this myself for the cycle of 2-3 reforking projects to push my changes and keep things latest, I settled on the following pattern.

### Set Up

 1. Fork as normal

 2. Clone your local repo

 		git clone github.com/myforkroute

 3. Create a Branch called fork

 		git branch Fork

 4. Update my remotes

 		// Here we update origin to point to the repo we forked

 		git remote set-url origin github.com/forkedroute

 		// Here we set up a new remote that points to my fork

 		git remote add fork github.com/myforkroute



### Workflow

Now keeping your repo up to date with the repo you forked is as simple as

	git branch master

	git pull origin master

	git branch fork

	git rebase master

I would still suggest working your pull request in branches off of the fork branch and doing the same with them so

	git branch my_pull_request

	// do some work

	// update fork if you need to 

	// switch back to your pull request branch

	git rebase fork