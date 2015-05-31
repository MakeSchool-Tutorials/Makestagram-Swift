---
title: "Pull-To-Refresh and Endless Scrolling"
slug: pull-to-refresh-endless-scrolling
---

You have done a lot of work on this app so far - in this step we wan't to make your life just a little bit easier.

We will implement a Pull-To-Refresh mechanism as you know it from most iOS apps. We will also implement a feature that will limit the amount of posts we download. Right now our approach is pretty inefficient; we download all posts in one query. Even though Parse automatically limits us to 100 posts, we are potentially downloading much more data then necessary.

After we are finished with this step, we will load posts in chunks of 5, only loading additional posts once the user reaches the end of the timeline. This behavior is well known from _Facebook_, _Instagram_ and many other apps.

To make your life easier, we have delivered a lot of this functionality as part of a class called `TimelineComponent`. So instead of implementing all of this from scratch, you will learn how to use the `TimelineComponent`!
