---
layout: post
title: "Why diff tools aren't the fix to the CL/RF git issues"
date: 2013-06-12 14:12
comments: true
categories: [Thoughts, git, line endings]
---

## Why diff tools aren’t the fix to the CL/RF git issues

Scott Hanselman had a great post today on the problem with line endings in git.

[Hanselman Blog](http://www.hanselman.com/blog/YoureJustAnotherCarriageReturnLineFeedInTheWall.aspx)

People in the comments seem to think that this is a problem solved by a diff tool. This has very little to do with the diff view, and more to do with the fact that this is a systemic problem in git. The fact that you even have to decide what type of line endings to set is slightly ridiculous in this day and age. Especially for languages where this has absolutely zero affect on the code.

However as Scott points out there are fixes, but the real thing is that we shouldn't have to be dealing with these fixes in the first place. A point that many missed.





