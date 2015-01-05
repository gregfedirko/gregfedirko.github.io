---
layout: post
title: MongoDB | To Embed or To Reference?
---

Sooner or later when working with mongoDB, you are going to run into a situation where you will need to decide between embedding documents, or referencing them.  Here are a few things to consider.

## Considerations:

* ### How will you query the data?

  This is usually the primary consideration when deciding between embedding and referencing.  When dealing with documents that are usually dependent on their parents, for example comments of a blog post, It is usually a good idea to emebed them.  It takes one simple query to get a blog post and all it's embedded comments.  If on the other hand you require more flexibility for your queries (maybe you want to be able to search through all the comments on your blog) then references to documents in a separate collection would be a good idea.

* ### Document Size

  Although not usually a huge issue, it is important to note that MongoDB imposes a 16MB limit on the size of documents.  For perspective however, the entire works of William Shakespere are under 6MB.

* ### Nested Data Structures

  MongoDB does not impose any limits on the depth of nested documents.  But you should avoid constructing trees and other complex data structures because of performance cost of querying them.

## Resources:

* [mongoDB docs](http://docs.mongodb.org/manual/tutorial/model-embedded-one-to-one-relationships-between-documents/)
