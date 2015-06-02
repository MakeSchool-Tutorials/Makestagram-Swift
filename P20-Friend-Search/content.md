---
title: "Implementing a Friend Search in Parse"
slug: friend-search-follow
---

In this chapter we will add a feature that will allow users to find & follow friends. We will start by setting up the UI in Storyboard, then we'll fill in the implementation in code.

**The explanations and instructions in this step will not be as detailed as in the previous ones - the View Controller we are about to build behaves very similar to the main View Controller in the notes app. Therefore it won't be necessary to discuss all of the involved concepts.**

If you are incorporating any kind of search into your app, the code in this step should serve as a very good template!

#Setting up the Friend Search UI

This is what the final UI will look like, once the entire app is complete:

![image](friend_search_complete.png)

At the root level we only have two components: a Search Bar and a Table View.

##Adding a Search Bar

> [action]
> Add a Search Bar to the `FriendSearchViewController` as show in the video below:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/SearchBarConstraints_small.mov" type="video/mp4">
>
> Remember that the last step is to use the shortkey _⌘⌥=_ to update the frame according to its new constraints.

##Adding a Table View

> [action]
> Add a Table View to the `FriendSearchViewController` as shown in the video below:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/FriendSearchTableViewConstraints_small.mov" type="video/mp4">

##Adding a Custom Table View Cell

> [action]
> Add a Table View Cell to the `FriendSearchViewController` as shown in the video below:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/CustomCellFriendSearch_small.mov" type="video/mp4">

#Creating Code Connections

We'll need multiple Code Connections to generate the cells from code and to implement the _follow_ button. We'll also need code connections for the Search Bar.

##Cell identifier

Let's start by setting up an identifier for our new Table View Cell.

> [action]
> Set the identifier for the `FriendSearchViewController`'s Table View Cell to _UserCell_:
> ![image](custom_cell_identifier.png)

##Custom Cell Class

Next, create a new class for this Cell.

> [action]
> Create a new `UITableViewCell` subclass called `FriendSearchTableViewCell` and add it to the _View_ group as shown below. Remember to first add a folder on the filesystem, then add that folder to Xcode. That way groups and folders stay in sync:
> ![image](friendsearchtableviewcell.png)

##Referencing Outlets and Button Callback

Then connect the new class to the Table View Cell.

> [action]
> Set the custom class of the TableViewCell to `FriendSearchTableViewCell`:
> ![image](friend_cell_custom_class.png)

Next, set up referencing outlets for the label and the button on the Table View Cell. Also add a callback for button taps:
> [action]
> Create the three code connections outlined below with the `FriendSearchTableViewCell`:
> ![image](code_connections.png)

##Table View Data Source

Now, set the Table View's Data Source to be the `FriendSearchViewController`.

> [action]
> Set the Table View Data Source as as shown below:
> ![image](setup_data_source.png)

##Referencing Outlets for FriendSearchViewController

Next, set up referencing outlets to the `FriendSearchViewController` from the Table View and the Search Bar.

> [action]
> Set up the following referencing outlets:
> ![image](referencing_outlet.png)

##Search Bar Delegate

Finally, set up the Search Bar Delegate to be the `FriendSearchViewController`.

> [action]
> Set up the Search Bar Delegate as shown below:
> ![image](search_bar_delegate.png)

#Adding the Friend Search Code

As discussed at the beginning of this step, we won't discuss the code in detail. The source code has comments in all the relevant places. We will provide you with the full source code that you need to each class - you should take time to read through it and make sure that you understand it. We'll then discuss a few interesting, high-level details about the solution.

##Addding Parse Requests

First we are going to add 5 different Parse request.

> [action]
> Add the following methods to the `ParseHelper` class:
>
     // MARK: Following
>
    /**
      Fetches all users that the provided user is following.
>
      :param: user The user who's followees you want to retrive
      :param: completionBlock The completion block that is called when the query completes
    */
    static func getFollowingUsersForUser(user: PFUser, completionBlock: PFArrayResultBlock) {
      let query = PFQuery(className: ParseFollowClass)
>
      query.whereKey(ParseFollowFromUser, equalTo:user)
      query.findObjectsInBackgroundWithBlock(completionBlock)
    }
>
    /**
      Establishes a follow relationship between two users.
>
      :param: user    The user that is following
      :param: toUser  The user that is being followed
    */
    static func addFollowRelationshipFromUser(user: PFUser, toUser: PFUser) {
      let followObject = PFObject(className: ParseFollowClass)
      followObject.setObject(user, forKey: ParseFollowFromUser)
      followObject.setObject(toUser, forKey: ParseFollowToUser)
>
      followObject.saveInBackgroundWithBlock(nil)
    }
>
    /**
      Deletes a follow relationship between two users.
>
      :param: user    The user that is following
      :param: toUser  The user that is being followed
    */
    static func removeFollowRelationshipFromUser(user: PFUser, toUser: PFUser) {
      let query = PFQuery(className: ParseFollowClass)
      query.whereKey(ParseFollowFromUser, equalTo:user)
      query.whereKey(ParseFollowToUser, equalTo: toUser)
>
      query.findObjectsInBackgroundWithBlock {
        (results: [AnyObject]?, error: NSError?) -> Void in
>
          let results = results as! [PFObject]
>
          for object in results {
            object.deleteInBackgroundWithBlock(nil)
          }
      }
    }
>
    // MARK: Users
>
    /**
      Fetch all users, except the one that's currently signed in.
      Limits the amount of users returned to 20.
>
      :param: completionBlock The completion block that is called when the query completes
>
      :returns: The generated PFQuery
    */
    static func allUsers(completionBlock:PFArrayResultBlock) -> PFQuery {
      let query = PFUser.query()!
      // exclude the current user
      query.whereKey(ParseHelper.ParseUserUsername,
        notEqualTo: PFUser.currentUser()!.username!)
      query.orderByAscending(ParseHelper.ParseUserUsername)
      query.limit = 20
>
      query.findObjectsInBackgroundWithBlock(completionBlock)
>
      return query
    }
>
    /**
    Fetch users who's username matches the provided search term.
>
    :param: searchText The text that should be used to search for users
    :param: completionBlock The completion block that is called when the query completes
>
    :returns: The generated PFQuery
    */
    static func searchUsers(searchText: String, completionBlock: PFArrayResultBlock)
      -> PFQuery {
      /*
        NOTE: We are using a Regex to allow for a case insensetive compare of usernames.
        Regex can be slow on large datasets. For large amount of data it's better to store
        lowercased username in a separate column and perform a regular string compare.
      */
      let query = PFUser.query()!.whereKey(ParseHelper.ParseUserUsername,
        matchesRegex: searchText, modifiers: "i")
>
      query.whereKey(ParseHelper.ParseUserUsername,
        notEqualTo: PFUser.currentUser()!.username!)
>
      query.orderByAscending(ParseHelper.ParseUserUsername)
      query.limit = 20
>
      query.findObjectsInBackgroundWithBlock(completionBlock)
>
      return query
    }

These are a total of 5 differnent queries.

##Implementing the FriendSerchTableViewCell

Code...

#Getting Additional Users into Makestagram

To test all of this new functionality we need multiple users with multiple posts stored on our server. There are two ways how you can accomplish this:

1. Create new users in the Parse Data Browser, then log in with these users and create posts.
2. [Download the data that we have prepared for you.](https://s3.amazonaws.com/mgwu-misc/SA2015/Parse_Exports.zip)

The downloaded data contains multiple users and a few posts. You can import them into your server through the Parse data browser.

> [action]
>
> 1. Unzip the Parse data that you downloaded
> 2. Use the _import_ functionality in the Parse Data Browser to select the two Parse _.json_ files and upload them:
> <video width="100%" height="400pt" controls>
  <source src="https://s3.amazonaws.com/mgwu-misc/SA2015/ParseImport_small.mov" type="video/mp4">
> 3. Make sure to import the *_User* file and the _Post_ file  
