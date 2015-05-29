---
title: "Implementing the Like feature"
slug: parse-implement-like
---

In this step we will tackle _the_ core feature of _Makestagram_. Liking posts!
We will need to tackle that feature from three different angles:

1. We need new Parse queries to fetch/add/remove likes of posts
2. We need to extend the `Post` model to store the likes that belong to it
3. We need to connect the UI to our new code. That means, for example, displaying the red heart if a user
has liked a certain post

We will start with adding new Parse queries! However, I first want to suggest some further cleanup of our existing Parse query!

#Cleaning up the Timeline Query - once again

Take a look at ...
