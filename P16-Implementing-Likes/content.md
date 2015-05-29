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

Take a look at the query we have right now:

    static func timelineRequestforCurrentUser(completionBlock: PFArrayResultBlock) {
      let followingQuery = PFQuery(className: "Follow")
      followingQuery.whereKey("fromUser", equalTo:PFUser.currentUser()!)

      let postsFromFollowedUsers = Post.query()
      postsFromFollowedUsers!.whereKey("user", matchesKey: "toUser", inQuery: followingQuery)

      let postsFromThisUser = Post.query()
      postsFromThisUser!.whereKey("user", equalTo: PFUser.currentUser()!)

      let query = PFQuery.orQueryWithSubqueries([postsFromFollowedUsers!, postsFromThisUser!])
      query.includeKey("user")
      query.orderByDescending("createdAt")

      query.findObjectsInBackgroundWithBlock(completionBlock)
    }

**What could be improved?**

Well, we have a ton of _Strings_ inside of this method. Using strings in such a manner can cause multiple problems.

Firstly, typos can cause bugs that are difficult to debug. The compiler won't verify any of these strings and won't be able to identify if you are querying for _"Uzer"_ instead of _"User"_.

Secondly, if we use those strings in multiple places, we need to remember to update every occurrence of them if something in our Parse backend changes.

Instead of using plain strings it would be much better to use constants!
