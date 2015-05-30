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

So, how can we create a method that adds likes to a certain post? **I want to give you a try here!** Based on your experience with Parse so far, try and see if you can build a query for this on your own! We will play this game for all queries in this step.

If you come up with your own solution that seems to work but looks different from ours - please replace your solution with the provided one. That way we can avoid that you run into issues later on.

<div class="solution"></div>
Here's our solution:

    static func likePost(user: PFUser, post: Post) {
      let likeObject = PFObject(className: ParseLikeClass)
      likeObject.setObject(user, forKey: ParseLikeFromUser)
      likeObject.setObject(post, forKey: ParseLikeToPost)

      likeObject.saveInBackgroundWithBlock(nil)
    }

It is pretty straight forward! the method takes a `PFUser` and a `Post` reference. Then it generates a `likeObject` based on these two input parameters and saves it.

##Deleting Likes

Deleting a like is a tiny bit trickier. We need to first build a query to find the like object before we can delete it. Can you still come up with the `unlikePost` method on your own?

<div class="solution"></div>
Here's our solution:

    static func unlikePost(user: PFUser, post: Post) {
      // 1
      let query = PFQuery(className: ParseLikeClass)
      query.whereKey(ParseLikeFromUser, equalTo: user)
      query.whereKey(ParseLikeToPost, equalTo: post)

      query.findObjectsInBackgroundWithBlock {
        (results: [AnyObject]?, error: NSError?) -> Void in
         // 2
          if let results = results as? [PFObject] {
            for object in results {
              object.deleteInBackgroundWithBlock(nil)
            }
          }
      }
    }

1. We build a query to find the like of a given user that belongs to a given post
2. We iterate over all like objects that met our requirements and delete them.

Technically, there never should be more then one like for a given user on a given post. Our like code will ensure that. However, there are little guarantees in software development and especially when working with networking code, there are a ton of possible sources for issues. We could do some more error handling here, and log an error message if we find more then one like that - but that's well beyond the scope of this tutorial!

##Fetching all likes for a given post

It's once again on you! Try to come up with an implementation for the `likesForPost` method.
Hint: it should take a completion block and call it when the query completes! We already have implemented one Parse request that does this...

<div class="solution"></div>
And here's our solution:

    // 1
    static func likesForPost(post: Post, completionBlock: PFArrayResultBlock) {
      let query = PFQuery(className: ParseLikeClass)
      query.whereKey(ParseLikeToPost, equalTo: post)
      // 2
      query.includeKey(ParseLikeFromUser)

      query.findObjectsInBackgroundWithBlock(completionBlock)
    }

There are two interesting aspects that should be highlighted:

1. Our method is taking a `PFArrayResultBlock` as an argument. We've used the same approach in our `timelineRequestforCurrentUser` method. The `PFArrayResultBlock` has the following signature:

    ([AnyObject]?, NSError?) -> Void

That's the default signature for the callback of most Parse queries. It returns an _optional_ result and and _optional_ error.
By taking this default block as argument, we can hand it directly to the `findObjectsInBackgroundWithBlock` method! This way, whoever has called the `likesForPost` method will get the results in the callback block that they provide.

2. We are using the `includeKey` method to tell Parse to fetch the `PFUser` object for each of the likes (we've discussed `includeKey` in detail when building the timeline request). We want to fetch the `PFUser` along with the likes, because we later on want to display the usernames of all users that have liked a post. Remember, without the `includeKey` line we would just have a reference to a `PFUser` and would have to start a separate request to fetch the information about the user.

##Summing it up

Awesome! We now have request to add / delete and fetch likes. Hopefully this section has helped to get a little bit more comfortable in writing and understanding Parse queries.

Just to make sure we're on the same page, here's what all the queries that we just added to the `ParseHelper` should look like:

    // MARK: Likes

    static func likePost(user: PFUser, post: Post) {
      let likeObject = PFObject(className: ParseLikeClass)
      likeObject.setObject(user, forKey: ParseLikeFromUser)
      likeObject.setObject(post, forKey: ParseLikeToPost)

      likeObject.saveInBackgroundWithBlock(nil)
    }

    static func unlikePost(user: PFUser, post: Post) {
      let query = PFQuery(className: ParseLikeClass)
      query.whereKey(ParseLikeFromUser, equalTo: user)
      query.whereKey(ParseLikeToPost, equalTo: post)

      query.findObjectsInBackgroundWithBlock {
        (results: [AnyObject]?, error: NSError?) -> Void in
          if let results = results as? [PFObject] {
            for object in results {
              object.deleteInBackgroundWithBlock(nil)
            }
          }
      }
    }

    static func likesForPost(post: Post, completionBlock: PFArrayResultBlock) {
      let query = PFQuery(className: ParseLikeClass)
      query.whereKey(ParseLikeToPost, equalTo: post)
      query.includeKey(ParseLikeFromUser)

      query.findObjectsInBackgroundWithBlock(completionBlock)
    }

A short side note: We haven't discussed the `// MARK:` feature of Xcode yet. It allows you to group your methods into different sections which can be extremely useful! A click into the _Jumpbar_ in the top right corner of Xcode will show you an outline of methods you've add to your class:

![image](jumpbar.png)

If you include `// MARK:' sections in your source code, they will show up as headers in this view - great for navigating through more complex classes!

With all of the queries in place, we should think about how we want tie them into the rest of our code, next!
Where should we place the code that adds and removes likes from `Post` objects?

We're going to add it directly to the `Post` class and you'll shortly see why!

#Extending the Post class

In most object oriented programs we wan't to couple the information that an object stores along with its behavior. If I have access to a `Post` object I would like to be able to like it or unlike it with a simple method call. I would also like to be able to access of all of the likes of a `Post` directly through a simple method call.

Now we're going to add this functionality to the `Post` class. It consists of two different parts:

1. Storing likes
2. Adding / removing likes

Let's start with storage. **Why do we want to store likes in the first place?**

##Storing likes

We want to avoid to perform network requests, every single time we want to access the likes that belong to a post. Instead, similar to the post's image, we want to cache the information we have fetched.

Towards the end of this tutorial we will add a pull-to-refresh mechanism that allows users to refresh the timeline. We want to cache all information until such a refresh happens.

In which format should we store likes? For our purposes the best format is an array of users that have liked a certain post.

Just as with the image of a `Post`, we won't load all of the likes directly with the timeline query. Instead, we will load them lazily as soon as a post is displayed. This means, we once again need to deal with data is available _asynchronously_. Our favorite tool for such cases is the `Dynamic` wrapper - as we discussed in detail when we implemented the image download.

With all this in mind, let's add the `likes` property to the `Post` class.

<div class="action"></div>
Add the following property to the `Post` class:

    var likes =  Dynamic<[PFUser]?>(nil)

We create the `likes` property as a _dynamic_, _optional_ array of `PFUser` - now that's a mouthful! However, we've used all of these concepts before. We make the property `Dynamic` so that we can listen to changes and update our UI after we've downloaded the likes for a post. We make it _optional_, because before we've downloaded the likes this property will be `nil`. 
