---
title: "Decorating the Post Table View Cell"
slug: decorating-post-table-view-cell
---

Now we'll spend some time in Interface Builder and with Auto Layout to polish up our extremely simple UI. We'll also add some more assets to our app. That will allow us to improve the look & feel throughout the next chapters.

#Adding additional resources

This step is pretty simple; we're going to add some more resources to our Asset Catalog. You've already downloaded the Art Pack for this tutorial earlier on, in case you no longer know where you've stored it, you can get it again [here](https://www.dropbox.com/s/kt9w4m94z4y3m4f/Art.zip?dl=0) (I don't blame you; my file system is pretty messy!).

<div class="action"></div>
Add new image sets for each of these resources (if you no longer know how this works, go back to the step where we set up the Tab Bar). The result should look like this (note that you don't need to set up _camera_ and _AppIcon_):

![image](add_all_resources.png)

Make sure you have added all of the resources with the correct names! Now we have some hearts and buttons that can we use to turn our Table View Cell into a nice looking interface.

When we are entirely done, we want the Table View Cell to look like this:

![image](finished_cell.png)

We won't finish the entire set up in this step - but we'll get pretty far!

#Changing the PostTableViewCell height

First, we'll make the cell a little bit larger to make space for our additional UI elements.

TODO: Decide height later...

#Adding the like button

Now let's start designing one of the core experiences of our app - the _like_ button!

##Creating an image button

<div class="action"></div>
Drag a Button from the object library and add it to the Table View Cell (technically the _Content View_ of the Table View Cell.) Move it to the bottom-right corner of the cell.

Now we're going to configure the like button to show the heart image.

<div class="action"></div>
1. Select the Button that you just created.
2. Open the _Attributes Inspector_.
3. Set the Button Type to _Custom_. Whenever you want to create a button that just shows an image, you should choose the _Custom_ type. The _System_ type that is selected by default, tints all images in a bright blue.
4. Remove the _Title_ from the button
5. Set the image to the _Heart_ button
![image](like_button_setup2.png)

Ok, our basic button is set up. Now we need to take care of some Auto Layout!

##Defining size and layout

Before designing your Apps, you should at least glance over [Apple's Human Interface Guidelines for iOS](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/).

They contain many do's and don'ts for mobile developers and designers.  
One part of the documentation discusses sizes of buttons:

> Make it easy for people to interact with content and controls by giving each interactive element ample spacing. Give tappable controls a hit target of about 44 x 44 points.

Source: [Apple's Human Interface Guidelines - Layout and Appearance.](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/MobileHIG/LayoutandAppearance.html)

We should keep that in mind when creating our like button!

<div class="action"></div>
Set up the constraints as shown below; we are setting the button size to the minimum recommended size of _44x44_:

![image](like_button_constraints.png)

Don't forget that you need to manually update the frame after you have added your constraints!

<div class="action"></div>
![image](like_button_update_frame.png)

You can either use the top menu or the _⌥⌘=_ shortkey, just make sure the heart button is selected.

Now your cell with the like button should look more or less like this:

![image](like_button_cell.png)

I'm not a designer by trade, but to me the like button looks a little bit too big. Apple's guide stated that 44x44 should be the minimum size of our buttons - **what can we do?**

Luckily, the touch area of our button can be larger than the visual one! We can leave the button at 44x44 but make the heart image a little bit smaller. We can do that by defining _Insets_ for the image.

<div class="action"></div>
1. Select the like button.
2. Open the _Attribute Inspector_ Tab in the right panel
3. Set the all of the four inset values to _4_:

![image](button_insets.png)

The touch area remains suitable for a small mobile screen - but the button looks better with the smaller heart image! You can use this _"trick"_ for many buttons in your own app as well!

##Changing the selected image

Just like the _Instagram_ app, _Makestagram's_ like button will have two different states. If you haven't liked a post yet, you'll see the gray like button. Once you've liked a post, you'll see the a red heart instead.

`UIButtons` have a total of four different states:
1. Default
2. Highlighted
3. Selected
4. Disabled

Most of them are self-explanatory. The `Highlighted` state is activated as soon as you tap onto a button. The `Selected` state needs to be triggered manually by the developer. As soon as a user likes a post, we'll set the like button to have the `Selected` state.

Let's set up the image for the selected state real quick.

<div class="action"></div>
Set up the _Heart-selected_ image for the like button:
![image](selected_state.png)

Finally our like button is complete! But there's a lot more UI to be built. Feel free to take a short break, but promise to come back!

#Adding the Action Button
