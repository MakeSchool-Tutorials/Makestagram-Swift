---
title: "Keeping the UI up to Date"
slug: pare-ui-updates-likes
---

In the last step we have completed the _like_ feature of _Makestagram_. However, users currently cannot see
which users have liked a post. And the like button won't activate, and become red when a user likes a post.

In this step we will make some more use of our _Bond_ framework, to update the UI whenever users like posts.
We have already defined the `likes` stored within each `Post` as `Dynamic`. As you might remember from the implementation of the lazy image loading, this allows us to listen to changes on the `likes` property.
