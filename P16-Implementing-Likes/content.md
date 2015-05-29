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
Let's add constants for all the different Parse classes and fields we are going to access! Since this isn't too exciting of an exercise, I'll provide you with the full list of constants that we need.

<div id="class"></div>
Add the following constants to the `ParseHelper` class:

    class ParseHelper {

      // Following Relation
      static let ParseFollowClass       = "Follow"
      static let ParseFollowFromUser    = "fromUser"
      static let ParseFollowToUser      = "toUser"

      // Like Relation
      static let ParseLikeClass         = "Like"
      static let ParseLikeToPost        = "toPost"
      static let ParseLikeFromUser      = "fromUser"

      // Post Relation
      static let ParsePostUser          = "user"
      static let ParsePostCreatedAt     = "createdAt"

      // Flagged Content Relation
      static let ParseFlaggedContentClass    = "FlaggedContent"
      static let ParseFlaggedContentFromUser = "fromUser"
      static let ParseFlaggedContentToPost   = "toPost"

      // User Relation
      static let ParseUserUsername      = "username"

      // ...

This approach has a third advantage that we did not discuss yet: all of the fields are now unambiguous. Before this change we could have used _"toPost"_ to refer to either the _toPost_ column in the _FlaggedContent_ class, or to the _toPost_ column in the _Like_ class. Through giving all of these constants a name, we have removed this ambiguity.

Note that the constant names follow this pattern: _Parse[ClassName][FieldName]_. Consistent naming is important to make your life as a developer easier!

With these constants in place, we can now update our timeline query to use them.

<div class="action"></div>
Change the timeline query to use constants instead of string literals:

    static func timelineRequestforCurrentUser(completionBlock: PFArrayResultBlock) {
      let followingQuery = PFQuery(className: ParseFollowClass)
      followingQuery.whereKey(ParseLikeFromUser, equalTo:PFUser.currentUser()!)

      let postsFromFollowedUsers = Post.query()
      postsFromFollowedUsers!.whereKey(ParsePostUser, matchesKey: ParseFollowToUser, inQuery: followingQuery)

      let postsFromThisUser = Post.query()
      postsFromThisUser!.whereKey(ParsePostUser, equalTo: PFUser.currentUser()!)

      let query = PFQuery.orQueryWithSubqueries([postsFromFollowedUsers!, postsFromThisUser!])
      query.includeKey(ParsePostUser)
      query.orderByDescending(ParsePostCreatedAt)

      query.findObjectsInBackgroundWithBlock(completionBlock)
    }

Same, same but nicer! Now we have a solid foundation to add more queries!

#Adding Parse queries for likes

There are three types of things our app needs to with likes:

1. Create a like, when a user likes a post
2. Delete a like, when a user unlikes a post
3. Fetch all likes for a given post

These three requirements translate directly into three different queries that we're going to
add to our `ParseHelper`.

Just as a refresher, here's how we are modeling likes in `Makestagram`:

![image](likes_model.png)

It's a pretty simple model, that only stores a reference to the user that performed the like and the post that has been liked (ignoring all of the fields that Parse provides automatically).




##Creating Likes
