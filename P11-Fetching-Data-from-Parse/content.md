---
title: "Fetching Data from Parse"
slug: fetching-data-parse
---

We have spent the last few steps discussing how to store data in Parse. In this step we will look at how to fetch it!
Throughout this step we will discuss the basics of querying data, while starting to work on the important _Timeline_ feature of Makestagram.

As a first step if implementing the _Timeline_ feature, we will set up the basic UI for the `TimelineViewController`.

#Adding TableView to the TimelineViewController

Let's add a `UITableView` to the `TimelineViewController` - we will use that Table View to display the posts in a user's timeline.

The Table View should be a full-screen view. However, we need to leave space for the status bar on the top and the Tab Bar on the bottom.

<div class="action"></div>
Open _Main.storyboard_ and add a Table View to the _TimelineViewController_. The resulting scene hierarchy should look like this:

![image](add_tableview.png)

<div class="action"></div>
Next, set up the horizontal constraints. Make sure that the Table View is selected. Then open the constraints menu. Uncheck _Constrain to margins_. Then set the constraints for _left_ and _right_ to _0_. Finally, hit _Add Constraints_:

![image](horizontal_constraints.png)

<div class="action"></div>
Now, set up the top constraint. Hold the _Control_ key and, in the _Document Outline_, drag a line from the Table View to the _Top Layout Guide_:
![image](top_layout_guide1.png)
Select _Vertical Spacing_ in the popup that shows up after you created the connection:
![image](top_layout_guide2.png)



#Basics of quering in Parse

#Building the Timeline query

#Displaying the query results
