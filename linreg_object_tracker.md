[Back to portfolio](index.md)

## LinReg Object Tracker
Code can be found [here](https://github.com/csulpizi/linreg_object_tracker/blob/master/linreg_track_objects.py).<br>
Readme can be found [here](https://github.com/csulpizi/linreg_object_tracker/blob/master/README.md).<br>

One project I worked on was developing a camera that detected, tracked, and counted the number of bicycles going through a video. In order to perform the tracking, I developed the following function. The function takes 2-dimensional coordinates and their associated time stamp and figures out which items (a single data point) belong to which objects (a series of data points that corresponds to a single object moving through the scene).

**The algorithm uses RANSAC to find new objects, uses linear regression to find the path of motion of each object, and calculates the distance between each point in each frame to the predicted location of each object in that frame to

### The Algorithm
The algorithm works frame by frame to progressively detect new objects, and determine whether or not points belong to each existing object.

There are 2 sets of objects:
1. ```obj_list```. This list contains all of the objects and the indices of each item belonging to it
2. ```s_act_obj```. This list contains the indices of objects in obj_list that are "active". A non-active object is an object that is off-screen or too old that the algorithm doesn't need to consider when determining whether or not points belong to each object. The active objects, on the other hand, are ones that are on-screen that the algorithm will consider when looking at each new frame. 

Each frame, the following actions are performed:
1. Determine whether or not any of the active objects should be changed to inactive. If the object's predicted location is off-screen, or if too much time has passed since the object was assigned a new point, that object is removed from the active objects list.
2. Calculate the predicted location of each active object.
3. Calculate the distance between each of the items in this frame and the predicted locations of each active object.
4. Find the item and the object with the minimum distance. If that distance is less than ```bound_tight``` then add that item to that object. 
5. Repeat step 4 with all of the remaining items and objects until there are no items, no objects, or until the minimum distance exceeds ```bound_tight```.
6. Use linear regression to recalculate the trajectory of each of the objects. The trajectory is calculated using the ```m``` most recent items in each object. 

After performing the above actions, if there are still items left over, the algorithm determines whether or not those points should be considered new objects using the following actions:
1. For each combination of unused items in this frame and unused items in the previous frame, find the parameters of the line passing through those points. Call these combinations the "pairs"
2. For each of the lines, find the line's predicted location for the next 5 time frames.
3. Calculate the distance between those predicted locations and all of the items in the next 5 time frames.
4. Calculate the "score" of each of the lines. For each distance calculated above, add 2 to the score for every distance that's smaller than the ```bound_tight```, and add 1 to the score for every distance that's between ```bound_tight``` and ```2 * bound_tight```.
5. Find the pair with the highest score that has at least one distance less than the ```bound_tight```, and create a new object with that pair of items in it. 
6. Repeat steps 1 through 5 until there are no remaining pairs with at least 1 point within the ```bound_tight```. 

The algorithm is demonstrated visually in the example section below.

### Example: Tracking Cars through a Scene.
This example uses points that were found by using background detection on a video of cars moving down a street. Each of the points represents a car that was detected. 

The plot below shows 4 frames of data that were 
<img src="https://github.com/csulpizi/linreg_object_tracker/blob/master/images/example_1.jpg">

The blue object demonstrates how the items are assigned to existing objects. The blue "x" points are points belonging to the blue object, the blue line is the calculated trajectory of that object, the end of the line represents the predicted location of that object in this time frame, and the grey circle shows the ```bound_tight``` distance from the predicted location. The black "x" points represent items that have not yet been classified. You can see that in each frame there is an item within the grey circle, and so each of those items is added to the blue object. 

The green object demonstrates how new objects are created. At t=551 you can see an item that is not assigned to any object. At t=552, you can see a new object appear near the previous point (the previous point is shown in grey). The algorithm decides that there are enough items in the trajectory of this potential object, and so the green object is created. You can see in t=553 that new items are within the ```bound_tight``` distance and are therefore added to the green object. 

The animation below shows the object tracking results superimposed on the source video. 
<img src="https://github.com/csulpizi/linreg_object_tracker/blob/master/images/example_2.gif">

As you can see the algorithm works well even when there are multiple objects on screen at the same time, and even when the motion of the object is non-linear. 
