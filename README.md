First Assignment, FSM - Experimental Robotics
================================
**A ROS-based assignment for the Experimental Robotics Laboratory course held at the University of Genoa.**

*Code documentation can be found [here](https://davidebruzzo.github.io/assignment_FiniteStateMachine/).*

Author: [*Davide Bruzzo*](mailto:davide.brzo@gmail.com?subject=[GitHub]%20Source%20Han%20Sans)

****************************
## Introduction

This repository contains my solution of the 1st assignment of Experimental Laboratory class which I made on ROS-based software to simulate a robot's behaviour.

The assignment required the realization of a Finite State Machine (FSM) that simulates a robot moving in an environment and taking decisions based on some variables on where to move next.

This FSM is made with [Smach](http://wiki.ros.org/smach) and bases its behaviour on the Ontology created on armor thanks to [armor_py_api](https://github.com/EmaroLab/armor_py_api).

The Robot (*robot1*) can use the ontology to get informations about the environment. It has also a controller and a planner for simulating its moving in the environment and a battery, to simulate discharging and recharging.

### Scenario 

The scenario involves a robot deployed in a indoor environment for surveillance purposes.

The robot’s objective is to visit the different locations and stay there for some times.

### The Environment

The environment is assumed to be a 2D environment composed of:
- **Rooms**: locations with one door; 
- **Corridors**: location with at least two doors.

The specific environment required by the assignment is made of *4 rooms* and *3 corridors* as here below: 


<p align="center">
<img src="https://github.com/davidebruzzo/ResTrack/blob/main/Flowchart.drawio.png" width="900" />
<p>
