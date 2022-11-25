First Assignment, FSM - Experimental Robotics
================================
**A ROS-based assignment for the Experimental Robotics Laboratory course held at the University of Genoa.**

*Code documentation can be found [here](https://davidebruzzo.github.io/assignment_FiniteStateMachine/).*

Author: [*Davide Bruzzo (S4649449)*](mailto:davide.brzo@gmail.com?subject=[GitHub]%20Source%20Han%20Sans)

****************************
## Introduction

This repository contains my solution of the 1st assignment of Experimental Laboratory class which I made on ROS-based software to simulate a robot's behaviour.

The assignment required the realization of a Finite State Machine (FSM) that simulates a robot moving in an environment and taking decisions based on some variables on where to move next.

This FSM is made with [Smach](http://wiki.ros.org/smach) and bases its behaviour on the Ontology created on armor thanks to [armor_py_api](https://github.com/EmaroLab/armor_py_api).

The Robot (*robot1*) can use the ontology to get informations about the environment. It has also a controller and a planner for simulating its moving in the environment and a battery, to simulate discharging and recharging.


___
### Scenario 

The scenario involves a robot deployed in a indoor environment for surveillance purposes.

The robot’s objective is to visit the different locations and stay there for some times.


___
### The Environment

The environment is assumed to be a 2D environment composed of:
- **Rooms**: locations with one door; 
- **Corridors**: location with at least two doors.

The specific environment required by the assignment is made of *4 rooms* and *3 corridors* as below: 


<p align="center">
<img src="https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/images/env.png" width="300" />
<p>

Where:
- *E* is the starting location and the charging room;
- *R1, R2, R3* and *R4* are the rooms;
- *C1* and *C2* are the corridors.


___
 ### The Phases
 
- Phase 1:
  - The robot start in the E location;
  - It waits here waits until it receives the information to build the topological map (with the relations between corridors and rooms, adding doors)

- Phase 2:
  - The robot moves in a new location;
  - It waits for some times before to visit another location. This behavior is repeated in a infinite loop and simulated with planning and controlling;
  - When the robot’s battery is low, it goes in the E location and wait for recharging.
  
  
___
  ### Surveillance policy
  The robot should follow this surveillance policy:
  - It should mainly stay on corridors;
  - If a reachable room has not been visited for some times it should visit it.
  
  The policy I implemented for surveillance is based on randomness in order to make robot more random-based. In fact the node [my_helper](https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/scripts/my_helper.py) in the function ```DecideUrgentReachable()``` obtains a list of room *reachable* and a list of room *urgent*, then by seeing if there are some of them that belongs to both lists it picks a random item belonging to both lists.
  
  ___
 ## Software architecture
  
  Down below the component diagram: 
  <p align="center">
  <img src="https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/images/uml.drawio (1).png" width="500" />
  <p>

From this diagram is quite visible the centrality of the **Finite State Machine** node implemented in the [assignment_FSM.py](https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/scripts/assignment_FSM.py). This node is vital to simulate the robot's behaviour. There are also other nodes that simulates some external robot's behaviours, such as controlling and planning robot's path to the rooms.  

The [battery.py](https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/scripts/battery.py) node simulates the discharging and the recharging of the robot's battery with ```Bool.msg``` taken by the ```std_msgs/Bool.msg``` file when the robot has low battery is sent to the state machine.

In detail:
- **planner**: it's an action server that simulates the planning of the path. It computes the path and gives the result through *Plan.action*, that is made of a set of points which compose the resulting plan.
- **controller**: it's an action server that simulates the moving of the robot along the path. It uses the *Control.action* to give the result and the feedbacks updating the position of the robot passing the via_points.
- **armor service**: it is used to do all the interactions with the ontology, this is made through queries done by the FSM.

___
### ROS messages and parameters

To link all the nodes and make them communicating each other have been used some actions and messages:

- Actions:
	- ```Control.action``` --> The motion controlling interface:
		- **Action Service**: 
			- ```Goal```, *Point[] via_points*: the plan as a set of points to reach.
			-  ```Result```, *Point reached_point*: the reached point when the plan has been followed.
			- ```Feedback```, *Point reached_point*: the last `via_point` reached so far.
			
	- ```Plan.action``` --> The motion planning interface:
		- **Action Server**: 
			- ```Goal```, *Point target*: the point to be reached at the end of the plan.
			- ```Result```, *Point[] via_points*: the plan, i.e., the `via_points` to reach the `target` given the current position.
			- ```Feedback```, *Point[] via_points*: the set of `via_points` computed so far.
			
- Message:
	- ```Bool.msg```: published on the topic ```state/battery_low```, warn FSM when robot's battery low.
___
### Software architecture

The software that implements the *FSM* is made of 4 states:
- **WaitForOntology**;
- **Recharging**;
- **Plan_To_Urgent**;
- **Visit_Room**.

 <p align="center">
  <img src="https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/images/statemachine.jpg" width="600" />
  <p>
  
  The yellow arrows represent the recursive iteration, they are not written in order to simplify the graph.
  
  The equivalent graph can be obtained by writing these commands on the bash:
  
  ```bash
$ rosrun smach_viewer smach_viewer.py
```
and the result will be this:

<p align="center">
  <img src="https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/images/smach.png" width="300" />
  <p>
   
   Where you can see aswell the recursive ones.
   
   ### Temporal diagram
   
   Here it is the temporal diagram:
   
   <p align="center">
  <img src="https://github.com/davidebruzzo/assignment_FiniteStateMachine/blob/main/images/temporal.drawio.png" width="500" />
  <p>
  
   It shows all the nodes of the architecture, in particular it's colored in yellow the battery node. This is done on purpose because this node is always active and    runs on a separate thread. 
   It publishes a boolean value when the robot has *low battery*, it returns to FSM to recharge the robot. The other nodes are active aswell but they are based on    the interaction with FSM that makes them do something.
