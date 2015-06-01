---
title: "Pull-To-Refresh and Endless Scrolling"
slug: pull-to-refresh-endless-scrolling
---

You have done a lot of work on this app so far - in this step we wan't to make your life just a little bit easier.

We will implement a Pull-To-Refresh mechanism as you know it from most iOS apps. We will also implement a feature that will limit the amount of posts we download. Right now our approach is pretty inefficient; we download all posts in one query. Even though Parse automatically limits us to 100 posts, we are potentially downloading much more data then necessary.

After we are finished with this step, we will load posts in chunks of 5, only loading additional posts once the user reaches the end of the timeline. This behavior is well known from _Facebook_, _Instagram_ and many other apps.

To make your life easier, we have delivered a lot of this functionality as part of a class called `TimelineComponent`. So instead of implementing all of this from scratch, you will learn how to use the `TimelineComponent`!

First, we will change our timeline query so that we can load posts in certain ranges instead of all at once. Then we will walk through the different steps of adding the `TimelineComponent` to _Makestagram_.

#Making the Timeline Query more flexible

In order to load posts in chunks, we will need to change out timeline query to accept a _range_ parameter. That allows the caller to specify how many posts should be loaded.

Swift has a built in `Range` type that allows us to define a range with a start and an end index. We should change the timeline query to accept such a `Range` parameter.

Let's implement the change and discuss it afterwards.

<div class = "action"></div>
Modify the `timelineRequestforCurrentUser` method to look as following:

    // 1
    static func timelineRequestforCurrentUser(range: Range<Int>, completionBlock: PFArrayResultBlock) {
      let followingQuery = PFQuery(className: ParseFollowClass)
      followingQuery.whereKey(ParseLikeFromUser, equalTo:PFUser.currentUser()!)

      let postsFromFollowedUsers = Post.query()
      postsFromFollowedUsers!.whereKey(ParsePostUser, matchesKey: ParseFollowToUser, inQuery: followingQuery)

      let postsFromThisUser = Post.query()
      postsFromThisUser!.whereKey(ParsePostUser, equalTo: PFUser.currentUser()!)

      let query = PFQuery.orQueryWithSubqueries([postsFromFollowedUsers!, postsFromThisUser!])
      query.includeKey(ParsePostUser)
      query.orderByDescending(ParsePostCreatedAt)

      // 2
      query.skip = range.startIndex
      // 3
      query.limit = range.endIndex - range.startIndex

      query.findObjectsInBackgroundWithBlock(completionBlock)
    }

1. As discussed, we modify the method signature to accept a `Range` argument. That `Range` argument will define which portions of the timeline will be loaded. Ranges in Swift are defined like this: `5..10`.
2. `PFQuery` provides a `skip` property. That allows us - as the name let's us suspect - to define how many elements that match our query shall be skipped. This is the equivalent of the `startIndex` of our `range`, so all we need to do is a simple assignment.
3. We make use of an additional property of `PFQuery`: `limit`. The `limit` property defines how many elements we want to load. We calculate the size of the range (by subtracting the `startIndex` from the `endIndex`) and pass the result to the `limit` property.

This wasn't too difficult! Now we are prepared to use the `TimelineComponent` in our app!

#Incorporating the TimelineComponent

Now we will discuss, step by step, how to use the `TimelineComponent`. Remember this step so that you can come back here in case you want to use the component in your own app!

##Basic setup

The very first step is importing the `ConvenienceKit` framework into the `TimelineViewController`. That framework contains the `TimelineComponent`.

> [action]
> Add the following import statement to the top of _TimelineViewController.swift_:
>
>
    import ConvenienceKit

Next, our `TimelineViewController` needs to implement the `TimelineComponentTarget` protocol. The `TimelineComponent` needs our cooperation in a couple of different ways, this protocol defines which methods and properties need to be available on classes that want to work with it.

<div class="action"></div>
Extend the class definition of `TimelineViewController` as following:

    class TimelineViewController: UIViewController, TimelineComponentTarget {

#Conclusion

- PFQuery skip and limit
