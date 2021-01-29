# LaparoscopySim
Utilizing ur3 with end effectors to set up laparoscopic training simulator in gazebo

![](https://github.com/Darius0852/LaparoscopySim/blob/master/lapsimu.gif)
![](https://github.com/Darius0852/LaparoscopySim/blob/master/lapreal.gif)

--------------------------------------------------------------------------------------------------------------------------------------------

This readme explains how to launch and control the UR3 robots and laparsocopy "sleeve-pole" game in the simulation with ROS Kinetic/ROS industrial on Ubuntu 16.04 with gazebo. Much of the detail on how this works is documented in my report. After a new controller for the system was implemented, the Touch devices, which required Ubuntu 18.04 this catkin_ws was saved as a backup but was no longer updated. The simulation environment was unable to be created on Ubuntu 18.04 as ROS industrial was unavailable and therefore the UR3 robot models were unavailable.


--------------------------------------------------------------------------------------------------------------------------------------------

Make sure you have installed:

ROS Kinetic full
ROS industrial
gazebo
pygame
(There may be others that I have forgotten off the top of my head but if you google any error messages it should tell you what you need to download)

(Check TROUBLESHOOTING section at bottom for help)


--------------------------------------------------------------------------------------------------------------------------------------------

Once extracted, make sure the catkin_ws folder is in the home directory.

Any terminal used make sure 'setup.bash' is sourced once the catkin devel folder is created.

1. Open Terminal and open the catkin_ws folder

	cd catkin_ws

2. Use catkin_make to create two folders, build and devel (may take a while as it builds the entire project)

	catkin_make

3. Run the following command. This should launch the gazebo environment with the UR3's and tool end effectors.

	roslaunch move_ur3 Simulation.launch

4. Open new terminal and source setup.bash

	source catkin_ws/devel/setup.bash

5. Run the following command. This should launch code to move the UR3's into position and launches a "pygame" window to use keyboard controls. teleopmode changes the controller used to control the robots controllers other than keyboard have not been tested for some time now see catkin_ws/src/move_ur3/src/teleop_ur3.py for more details.

	roslaunch move_ur3 keyboard_teleop.launch teleopmode:=-k

Note: For some reason when the UR3's move into position they occasionaly 'jump' out of position slightly. To correct this you need to repeat the last command. click on the pygame window and press the Escape key to exit pygame which stops the code running also. Repeat previous command and the UR3's should move into the correct position.

6.1 From the prior note, we still need to launch the Laparoscopy training box into the world which we couldn't because of the potential for the UR3's to move out of position. Now they are in position use these commands to spawn the laparoscopy training box in a new terminal. Note, the file path needs to be explicit /home/*Username*/etc... so change "stewart" to your username.

	source catkin_ws/devel/setup.bash
 
	rosrun gazebo_ros spawn_model -file /home/user/catkin_ws/src/move_ur3/models/LaparoscopyBox.sdf -sdf -model Laparoscopy_Box

6.2 We should now launch the laparoscopy game (you can do this from the same terminal).

	rosrun gazebo_ros spawn_model -file /home/user/catkin_ws/src/move_ur3/models/TrainingGame.sdf -sdf -model Training Game

6.3 We can also now launch the sleeve for the game in the same terminal

	rosrun gazebo_ros spawn_model -file /home/user/catkin_ws/src/move_ur3/models/sleeve.sdf -sdf -model sleeve

You can change the sleeve spawn position and properties in its SDF file.

7. Now everything should be setup correctly. You can control the ur3's with the keyboard while the pygame window is clicked on.

![](https://github.com/Darius0852/LaparoscopySim/blob/master/Screenshot%202020-12-17%20at%2001.38.36.png)

The 6 ARM commands are:
UP DOWN LEFT RIGHT IN OUT  -  One of my reports will detail how this works. Essentially UP DOWN LEFT RIGHT are rotations around the training box hole (such that the tool does not move further into the box) and IN OUT exclusively moves the tool in and out of the training box.

The 4 GRIPPER commands are:
OPEN CLOSE ROTATE-LEFT ROTATE-RIGHT
The first two gripper commands open and close both fingers of the gripper whilst the second two rotate the gripper left and right about its axis.

----------------------------

For left ur3: 

ARM:

W    S    A    D    E    Q

Gripper:

V    C    R    F


----------------------------

For right ur3:

ARM:

I    K    J    L    U    O


Gripper:

N    B    Y    H

----------------------------

The robots may appear to move too slowly, open catkin_ws/src/jog_arm/jog_arm/config/jog_settings.yaml and change the max linear, rotational and joint speeds but be careful with this as it may be hard to get the balance right. keep a note of the original speeds.


--------------------------------------------------------------------------------------------------------------------------------------------

TROUBLESHOOTING:


----------------------------

Make sure file permissions are setup for the downloaded project by navigating to the catkin_ws/src/move_ur3/src folder and using the "chmod +x 'filename'" command, replacing the 'filename' with the files which will not launch. A quick way to do this is to enable permissions for the whole folder by using the "chmod 777 -R 'folder name'".

----------------------------

You may have to use the 'sudo pip install pygame' to bypass administrator and install pygame. 

----------------------------

A common error when running new python scripts is an 'indentation error'. There are two ways to solve this, paste the code into a new visual studio code document, highlight and format line spacing. The second way is to manually remove white-spaces at the points which the error code indicate. 

----------------------------

These are some commands you can send in the terminal to check the ROS controllers are working. These rospub commands will change the joint positions in gazebo (you can specify the 'positions' values yourself to test different positions. You can change 'robot1' to 'robot2' and vice versa to change which arm you send the commands to. The first two arm commands set the robot to the starting position.

Command to send from terminal:

ARM:

rostopic pub -1 /robot2/arm_controller/command trajectory_msgs/JointTrajectory "{joint_names:['elbow_joint', 'shoulder_lift_joint', 'shoulder_pan_joint','wrist_1_joint', 'wrist_2_joint', 'wrist_3_joint'], points:[{time_from_start:{secs: 1}, positions: [-1, 0.8, 2.9, 3.4, -1.3, 3.55]} ]}"

rostopic pub -1 /robot1/arm_controller/command trajectory_msgs/JointTrajectory "{joint_names:['elbow_joint', 'shoulder_lift_joint', 'shoulder_pan_joint','wrist_1_joint', 'wrist_2_joint', 'wrist_3_joint'], points:[{time_from_start:{secs: 1}, positions: [1, -0.8, -2.9, 2.6, -5, -6.3]} ]}"

GRIPPER:

rostopic pub -1 /robot1/gripper_controller/command trajectory_msgs/JointTrajectory "{joint_names:['swivel_joint', 'left_gripper_joint','right_gripper_joint'], points:[{time_from_start:{secs: 1}, positions: [0, 0, 0]} ]}"

rostopic pub -1 /robot2/gripper_controller/command trajectory_msgs/JointTrajectory "{joint_names:['swivel_joint', 'left_gripper_joint','right_gripper_joint'], points:[{time_from_start:{secs: 1}, positions: [0, 0, 0]} ]}"

----------------------------------
OPEN AND CLOSE GIPPER:

rostopic pub -1 /robot1/gripper_controller/command trajectory_msgs/JointTrajectory "{joint_names:['swivel_joint', 'left_gripper_joint','right_gripper_joint'], points:[{time_from_start:{secs: 1}, positions: [0, 0, 3]} ]}"


rostopic pub -1 /robot1/gripper_controller/command trajectory_msgs/JointTrajectory "{joint_names:['swivel_joint', 'left_gripper_joint','right_gripper_joint'], points:[{time_from_start:{secs: 1}, positions: [0, 0, 0]} ]}"

--------------------------------------
ROTATE GRIPPER:

rostopic pub -1 /robot1/gripper_controller/command trajectory_msgs/JointTrajectory "{joint_names:['swivel_joint', 'left_gripper_joint','right_gripper_joint'], points:[{time_from_start:{secs: 1}, positions: [2, 0, 0]} ]}"

rostopic pub -1 /robot1/gripper_controller/command trajectory_msgs/JointTrajectory "{joint_names:['swivel_joint', 'left_gripper_joint','right_gripper_joint'], points:[{time_from_start:{secs: 1}, positions: [0, 0, 0]} ]}"
