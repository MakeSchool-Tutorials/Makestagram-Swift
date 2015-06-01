---
title: "Pull-To-Refresh and Endless Scrolling"
slug: pull-to-refresh-endless-scrolling
---

You have done a lot of work on this app so far - in this step we wan't to make your life just a little bit easier.

We will implement a Pull-To-Refresh mechanism as you know it from most iOS apps. We will also implement a feature that will limit the amount of posts we download. Right now our approach is pretty inefficient; we download all posts in one query. Even though Parse automatically limits us to 100 posts, we are potentially downloading much more data then necessary.

After we are finished with this step, we will load posts in chunks of 5, only loading additional posts once the user reaches the end of the timeline. This behavior is well known from _Facebook_, _Instagram_ and many other apps.

To make your life easier, we have delivered a lot of this functionality as part of a class called `TimelineComponent`. So instead of implementing all of this from scratch, you will learn how to use the `TimelineComponent`!

The component will be responsible for storing all of the posts displayed on the timeline. It will also be responsible for triggering requests whenever a user refreshes the timeline. This means we will restructure a fair amount of our existing code in the `TimelineViewController`.

First, we will change our timeline query so that we can load posts in certain ranges instead of all at once. Then we will walk through the different steps of adding the `TimelineComponent` to _Makestagram_.

#Making the Timeline Query more flexible

In order to load posts in chunks, we will need to change out timeline query to accept a _range_ parameter. That allows the caller to specify how many posts should be loaded.

Swift has a built in `Range` type that allows us to define a range with a start and an end index. We should change the timeline query to accept such a `Range` parameter.

Let's implement the change and discuss it afterwards.

> [action]
Modify the `timelineRequestforCurrentUser` method to look as following:
>
    // 1
    static func timelineRequestforCurrentUser(range: Range<Int>, completionBlock: PFArrayResultBlock) {
      let followingQuery = PFQuery(className: ParseFollowClass)
      followingQuery.whereKey(ParseLikeFromUser, equalTo:PFUser.currentUser()!)
>
      let postsFromFollowedUsers = Post.query()
      postsFromFollowedUsers!.whereKey(ParsePostUser, matchesKey: ParseFollowToUser, inQuery: followingQuery)
>
      let postsFromThisUser = Post.query()
      postsFromThisUser!.whereKey(ParsePostUser, equalTo: PFUser.currentUser()!)
>
      let query = PFQuery.orQueryWithSubqueries([postsFromFollowedUsers!, postsFromThisUser!])
      query.includeKey(ParsePostUser)
      query.orderByDescending(ParsePostCreatedAt)
>
      // 2
      query.skip = range.startIndex
      // 3
      query.limit = range.endIndex - range.startIndex
>
      query.findObjectsInBackgroundWithBlock(completionBlock)
>    }

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

> [action]
Extend the class definition of `TimelineViewController` as following:
>
    class TimelineViewController: UIViewController, TimelineComponentTarget {

Now we are ready to implement the `TimelineComponentTarget` protocol!

##Implementing the TimelineComponentTarget protocol

Let's first take a look at what the protocol looks like. You can _CMD+Click_ onto the protocol name and Xcode will jump to the protocol definition:

    public protocol TimelineComponentTarget: class {
      typealias ContentType

      var defaultRange: Range<Int> { get }
      var additionalRangeSize: Int { get }
      var tableView: UITableView! { get }
      func loadInRange(range: Range<Int>, completionBlock: ([ContentType]?) -> Void)
    }

We can ignore the `typealias`, which leaves use with four requirements for classes that implement the `TimelineComponentTarget` protocol:
1. **`defaultRange`**: A property that defines how many posts should be loaded initially
2.  **`additionalRangeSize`**: A property that defines how many additional posts should be loaded once the user reaches the bottom of the timeline
3. **`tableView`**: A reference to a `UITableView` - we've already set that up!
4. **`loadInRange`**: A method that loads a certain portion of the timeline and calls the `completionBlock` when it's done.

This means we need to add two properties and one method to implement this protocol.

Let's start with the two properties:

> [action]
Add the following three properties to the `TimelineViewController` class:
>
  let defaultRange = 0...4
  let additionalRangeSize = 5

We are defining that we start by showing the latest 5 posts (index 0 to 4). Whenever a users reaches the end of the timeline, we load an additional 5. You could change the behavior of your timeline by simply changing these values!

To conform to the `TimelineComponentTarget` protocol we need to implement one more method: `loadInRange`. Currently we are performing our timeline query inside of the `viewDidAppear` method. We first perform a query, then we update the TableView.

When working with the `TimelineComponent`, we are no longer responsible for updating the TableView and starting the queries. Instead, that will be handled for us by the component.

All we need to do, is implement the `loadInRange` method, so that the component can call it and receive the posts on a user's timeline.

**Based on the query code in the `viewDidAppear` method, can you come up with an implementation of `loadInRange`?**

> [solution]
> One valid solution looks like this:
>
    func loadInRange(range: Range<Int>, completionBlock: ([Post]?) -> Void) {
      // 1
      ParseHelper.timelineRequestforCurrentUser(range) {
        (result: [AnyObject]?, error: NSError?) -> Void in
          // 2
          let posts = result as? [Post] ?? []
          // 3
          completionBlock(posts)
      }
    }
>
1. We start by calling the `timelineRequestforCurrentUser` method. Earlier we have extended the method to take a `range` parameter. We now simply pass on the range that we received in the `range` argument.
2. In the callback of the query we check whether or not we have received a result. If the result is `nil` we store an empty array in the `posts` variable.
3. We pass the `posts` that have been loaded back to the `TimelineComponent` by calling the `completionBlock`.

Now our class fully conforms to the `TimelineComponentTarget` protocol! However, there are few additional changes we need to make to the `TimelineViewController`.

##Informing the TimelineComponent about events

There are two events that the `TimelineComponent` needs to know about in order to do its job correctly:
1. The component needs to know when the TableView becomes visible, so that it can load the initial set of data
2. The component needs to know which cells are currently being displayed, so that it can load more cells as soon as the user has reached the latest cell in the Timeline

Let's start by informing the component when the TableView becomes visible.

###Triggering the initial timeline request

The `TimelineComponent` wants us to call the `loadInitialIfRequired()` method when that happens. We can implement that method call inside of the `viewDidAppear` method, which is called as soon as the TableView becomes visible.

Since we have implemented the timeline query inside of the `loadInRange` method, we also no longer need it in the `viewDidAppear` method. So this is a good chance to clean that method up.

> [action]
Change the `viewDidAppear` method of the `TimelineViewController` class to look as following:
>
    override func viewDidAppear(animated: Bool) {
      super.viewDidAppear(animated)
>
      timelineComponent.loadInitialIfRequired()
    }

Now the `TimelineComponent` will make an initial timeline request, if no data has been loaded so far. If the component has already queried the server and stored a user's posts, this method call does nothing at all. After the initial load, posts will only be reloaded if the user manually chooses to do so (by using the pull-to-refresh mechanism).

Next, we need to inform the `TimelineComponent` which cell is currently visible.

###Informing the TimelineComponent about visible cells

Whenever a cell becomes visible, we are required to call the `targetWillDisplayEntry:` method and pass the index of the currently displayed entry.

**Where can we place the code that calls that method?**

One option would be the `cellForRowAtIndexPath:` method that is called whenever the TableView requests us to create a cell. However, there are some cases where this method is called but the requested cell is not actually displayed - so this solution could lead to some bugs in our app.

Instead, there's a method that's part of the `UITableViewDelegate` protocol that is perfect for our purposes:

    func tableView(tableView: UITableView, willDisplayCell cell: UITableViewCell, forRowAtIndexPath indexPath: NSIndexPath)

This method is called whenever our TableView is about to display a cell! So let's implement that method!

> [action]
> Add the following `extension` **after** the class definition of `TimelineViewController`:
>
    extension TimelineViewController: UITableViewDelegate {
>
      func tableView(tableView: UITableView, willDisplayCell cell: UITableViewCell, forRowAtIndexPath indexPath: NSIndexPath) {
>
        timelineComponent.targetWillDisplayEntry(indexPath.row)
      }
>
    }

As mostly, we are implementing a new protocol in a separate class `extension` - that's not required but keeps our code better structured. The implementation of this method is fairly simple - we directly call the `timelineComponent` and inform it that a cell has been displayed.

Note that this code won't work yet. Firstly, we haven declared the `timelineComponent` property yet. Secondly, the `TimelineViewController` isn't the `delegate` of the TableView yet. Currently it is only the `dataSource`.



##Initializing and storing the TimelineComponent

We will also create a property that will store the `TimelineComponent` object.

> [action]
Add the following property to the `TimelineViewController` class:
>
    var timelineComponent: TimelineComponent<Post, TimelineViewController>!

Next, we will add code that creates an instance of the `TimelineComponent`. We'll add that to the `viewDidLoad` method. As soon as our view is loaded, we want the `TimelineComponent` to be available:

> [action]
> Extend the `viewDidLoad` method so that it initializes a `TimelineComponent`
>
    override func viewDidLoad() {
      super.viewDidLoad()
>
      timelineComponent = TimelineComponent(target: self)
      self.tabBarController?.delegate = self
    }

The `TimelineComponent` only takes one argument when it's being initialized: the `target`. The `target` is the object to which the `TimelineComponent` shall add it's functionality. In our case that's the `TimelineViewController`, so we pass `self` to the `initializer`.

#Conclusion

- PFQuery skip and limit
