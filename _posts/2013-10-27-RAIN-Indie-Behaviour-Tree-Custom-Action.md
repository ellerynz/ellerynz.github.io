---
layout: post
title: RAIN{Indie} Behaviour Tree Custom Action
category: Coding
tags: RAIN{Indie} Unity3D AI
year: 2013
month: 10
day: 27
summary: A tutorial on using custom actions in RAIN{Indie}
---

This has been sitting at 90% done for a while now. Rival Theory is making some changes so I thought I'd better hurry up with this before it becomes irrelevant. Like I mentioned in the [last post](http://blog.ellerynz.com/2013/07/how-i-created-ai-with-rainindie-and.html), I had to use RAIN{Indie} this year in a group project at university, and that involved making custom actions. Building off of the last tutorial, we now want the AI to patrol, and when they detect something, move towards it. Here's how I did it.

## Setting up waypoints

First we need some waypoints to get the AI to patrol between them. Then we'll spruce it up with custom actions.

### Create a waypoint collection for the sphere

<figure class="image-right image-long">
  <img alt="Waypoint inspector with Rain indie" src="/assets/images/2013-10-27/1-waypoint.png" />
  <figcaption class="image-sub-text">Waypoint inspector</figcaption>
</figure>

Select the Sphere's Path Manager.
- Choose Create a New Waypoint Collection. This will generate a new waypoint collection called Waypoints 1.
- Select Add a Waypoint. This generates a randomly named waypoint, in my case (Waypoint64585), under the Waypoints 1 collection.

Find the generated waypoint in Waypoints 1 in the hierarchy. Select Add a Connected Waypoint. This will generate another waypoint in the same waypoints collection. If you drag the waypoints apart you'll see that they are connected.

So you now have two connected waypoints in the Waypoints 1 collection. Position these waypoints to where you want the AI to patrol. For this example, set one waypoint to pos(5, 0, 15) and the other to pos(15, 0, 15).
Now to get the sphere to follow the waypoints.

<figure class="image-middle">
  <img alt="Connected waypoint with Rain indie" src="/assets/images/2013-10-27/2-waypoint-connected.png" />
  <figcaption class="image-sub-text">Two connected waypoints</figcaption>
</figure>

##Setup the behaviour tree

We want the Sphere to patrol between waypoints and keep an eye out for the cylinder at the same time. If the Cylinder is detected, the Sphere should stop patrolling, switch to the navgrid (to avoid obstacles), and move towards the cylinder.

Breaking this behaviour into a behaviour tree, we need a patrol node which has nodes for detecting and moving. We need a node for moving towards a cylinder. Finally, we need a variable for changing between patrolling and chasing.

<figure class="image-middle">
  <img alt="Behaviour tree with variable in Rain indie" src="/assets/images/2013-10-27/3-BTree-variable.png" />
  <figcaption class="image-sub-text">Full Behaviour Tree showing variable setting</figcaption>
</figure>

### What type of nodes are needed?

Off the root node, there is a choice between patrolling and chasing. We only want to do one or the other. A [select node](http://support.rivaltheory.com/rainindie/wiki/doku.php?id=behavior:behaviortrees:nodes:selector) will select the first node that returns success and will only fail if all child nodes fail. With the root as a selector we can place the patrol node first. If patrolling succeeds, the root selector node will only choose patrolling. If it fails, the root selector node will move on to the next node which will be chasing/moving towards the cylinder.

Patrolling has two parts, detecting and moving. These both need to be run at the same time, which we use a [parallel node](http://support.rivaltheory.com/rainindie/wiki/doku.php?id=behavior:behaviortrees:nodes:parallel) for.

The detecting node also has two parts, one [detect](http://support.rivaltheory.com/rainindie/wiki/doku.php?id=behavior:behaviortrees:nodes:detect) action and an [assign](http://support.rivaltheory.com/rainindie/wiki/doku.php?id=behavior:behaviortrees:nodes:assign) action. The detect action is the same as the previous tutorial where the Sphere's Sensor is used to detect the Cylinder's Aspect. The assign action is used to change the variable for chasing and patrolling. We want these nodes to be run in order (so we only change from patrolling if we've detected something), which is what the [sequencer](http://support.rivaltheory.com/rainindie/wiki/doku.php?id=behavior:behaviortrees:nodes:sequencer) node does.

Both the moving between waypoints and chasing the cylinder are custom actions which I'll go over next. Since we don't need a special node here, we'll just use a sequencer to call the custom action.
### Behaviour tree variable and preconditions

To switch between patrolling and chasing, we need to declare a variable. You can do this in the BT Editor by selecting the BT node and adding a variable to the list. For this example, I called the variable canSeeCylinder and set it to 0. 0 means nothing has been detected, 1 means something has been detected.

In the patrol node, set the precondition to canSeeCylinder == 0. This means the patrol node will only execute if canSeeCylinder is 0. For the assign action which I've called CanSeeCylinder, you want to set the canSeeVariable to 1. That is, set the Expression to canSeeCylinder = 1. Once the assign node is executed, it will change the value of the canSeeCylinder variable.
Finally, add a precondition to the ChaseCylinder node. Precondition canSeeCylinder == 1.

<figure class="image-middle">
  <img alt="Behaviour tree with precondition in Rain indie" src="/assets/images/2013-10-27/4-BTree-precondition.png" />
  <figcaption class="image-sub-text">Behaviour Tree with a precondition</figcaption>
</figure>

##Rain indie custom actions

We have two custom actions; MoveToWaypoint and MoveToCylinder.
### Generating custom actions

For custom actions, generate ActionScript in Javascript (it's quite similar in C#). Create a custom action, give it a class name in the class field, and then right click the custom action and generate the action script. If you try to save the file and there's a warning about line endings, just click convert. Ensure that the Class option in the custom action node is referring to the class you've created. Each custom action class starts with four functions: newclass, Start, Execute, Stop. You can read more about these functions [here](http://support.rivaltheory.com/rainindie/api/class_r_a_i_n_1_1_action_1_1_action.html). A quick rundown of three interesting ones would be:

- Start: called once when the node is first used
- Execute: does the logic for the action
- Stop: called as a cleanup when executing has finished

Each of these functions return ActionResult.SUCCESS|RUNNING|FAILURE

- Success: action completed successfully
- Running: more to be done
- Failure: action failed

### MoveToWaypoint

I created a file called AIPatrol to get the AI to patrol between waypoints. See the gist [here](https://gist.github.com/ellerynz/7188841).
In the start method, the wayPath is setup. This gives access to the [PathManager](http://support.rivaltheory.com/rainindie/wiki/doku.php?id=unitycomponents:rainpathmanager) and all of its methods. From here, we can ensure the waypoint collection is being used. We are also setting up the wayPathSize (how many waypoints in the waypoint collection) so we can loop through all the waypoints.

The nextWaypoint() function is for looping through the waypoint collection. Will simply go back to the first waypoint when the last waypoint has been reached and then calls getWaypointLocation(). That method will find the transform.position of the next waypoint in the collection. It seems a little messy but the currentPointName is getting the name of the waypoint. The split is needed because the name will be 'waypoint123 (UnityEngine.Transform)' and we just want the waypoint123 part. We can then find the waypoint position using the name we found. Looping through the waypoints this way means you can add and drop extra waypoints to this collection without having to rename them all.

The execute method is straightforward. Determine how close the AI is to the goal waypoint, if it's not close enough: keep moving. Otherwise, switch to the next waypoint. Note that this could have been done with a [MoveTo](http://support.rivaltheory.com/rainindie/api/class_r_a_i_n_1_1_core_1_1_agent.html#a6e3126a757900be5f116d178d8bc8126) like so:

(gist)

This will move the AI to a point and exit out once they're there. However, this never quite reached one of the waypoints for some reason, so I went with the 'how close' way of moving. Also note that the execute method is returning RUNNING. This is because we want this method to keep going, much like the update method in a regular unity script.

<figure class="image-middle">
  <img alt="Patrolling with Rain indie" src="/assets/images/2013-10-27/5-patrolling.png" />
  <figcaption class="image-sub-text">Two waypoints to patrol between</figcaption>
</figure>

#### MoveToCylinder

[Here](https://gist.github.com/ellerynz/7189202) I created the AIMoveToCylinder class. This has a similar start method to AIPatrol, except we're ensuring that the navgrid is used, and we need the cylinder object and the AI's sensor. The cylinder object is found by getting the variable from the detect action. The detected variable is found by getting the [context item](http://support.rivaltheory.com/rainindie/api/class_r_a_i_n_1_1_action_1_1_action_context.html) using the variable name.

In the execute method, the AI will move towards the cylinder as long as the cylinder has been detected and the ray sensor can currently see it. Otherwise, the AI will move towards the point that the cylinder was last seen using [VectorTarget](http://support.rivaltheory.com/rainindie/api/class_r_a_i_n_1_1_path_1_1_move_look_target.html#a6981df6e540892a15d1b52d9f1a789d3). Also, the canSeeCylinder is set back to 0 as the AI can no longer see the cylinder.

<figure class="image-middle">
  <img alt="AI detection with Rain indie" src="/assets/images/2013-10-27/6-detect-cylinder.png" />
  <figcaption class="image-sub-text">Sphere moving towards the cylinder</figcaption>
</figure>

##Outro

So that's a fairly basic introduction to custom actions. I hope it has given you a nice introduction to what can be accomplished with the custom actions. I may cover some newer stuff to do with Rain in the future.
