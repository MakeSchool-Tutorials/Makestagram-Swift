---
title: "Uploading photos"
slug: uploading-photos
---     

It's time to implement our very first feature: uploading Photos to Parse! In our App we want to allow the users to upload or take a photo as soon as they tap the camera button in the middle of the Tab Bar.

Typically at Tab Bar View Controller only allows a user to switch between different View Controllers. We don't want to switch to a View Controller when the button is tapped, instead we want to show an Action Dialog that lets the user take or select a picture:

![](taking_photo.png)

Unforunately, using a Tab Bar View Controller, we cannot **easily** perform an arbitrary method when one of the Tab Bar Items is selected. 

However,there's a **workaround**.

#Using a Tab Bar Item like a Button
Essentially we want to use the Photo Tab Bar Item like a button. When it is tapped we want to call a method to present dialog shown above.

One of the ways to accomplish this is to make use of the `UITabBarControllerDelegate` protocol.

That protocol contains a method that is interesting for us:

    optional func tabBarController(_ tabBarController: UITabBarController, shouldSelectViewController viewController: UIViewController) -> Bool

You can read the full documentation of the method [here](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITabBarControllerDelegate_Protocol/#//apple_ref/occ/intfm/UITabBarControllerDelegate/tabBarController:shouldSelectViewController:). 

Using this method, the Tab Bar View Controller asks its delegate whether or not it should present the View Controller that belongs to the selected Tab Bar Item.

**Can you imagine how we could use this method to accomplish our goal?**

<div class="solution"></div>
One of our classes can become the `delegate` of the `UITabBarViewController` and implement the method above. We can implement the method such that whenever the `PhotoViewController` should be presented, we show the photo capture dialog instead!

Let's implement this solution! The `TimelineViewController` will be the main View Controller of our app. When a user opens the App, the timeline will be displayed first.

That means we can use the `TimelineViewController` as the delegate for the `UITabBarViewController`.

Implementing our solution involves two steps:

1. Set the `TimelineViewController` as the delegate of `UITabBarViewController`
2. Implement the `tabBarController(_ , shouldSelectViewController:)` delegate method

<div class="action"></div>
Replace the entire source code in `TimelineViewController.swift` with the following one:

    import UIKit

    class TimelineViewController: UIViewController {

      override func viewDidLoad() {
        super.viewDidLoad()
        
        self.tabBarController?.delegate = self
      }

    }

    // MARK: Tab Bar Delegate

    extension TimelineViewController: UITabBarControllerDelegate {
      
      func tabBarController(tabBarController: UITabBarController, shouldSelectViewController viewController: UIViewController) -> Bool {
        if (viewController is PhotoViewController) {
          println("Take Photo")
          return false
        } else {
          return true
        }
      }
      
    }
    
What are we doing here? In `viewDidLoad` we are setting `TimelineViewController` to be the `tabBarController`s delegate. Every `UIViewController` in iOS has the `tabBarController?` property; if the View Controller is presented from a `UITabBarViewController` (as it's the case in our app), this property will store a reference to it.

In the second part of this solution, after:

    // MARK: Tab Bar Delegate
    
we implement the relevant protocol method. Within the method we check which View Controller is about to be selected.