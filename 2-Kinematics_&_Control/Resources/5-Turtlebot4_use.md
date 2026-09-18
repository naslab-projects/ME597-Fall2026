# Using the Turtlebot4

## Robot Assignments
When lab is over, you must power off the turtlebot and return it to the storage cabinet. Be mindful that this robot is shared with the other section. Regardless of how much you completed, record your progress. For robot assignments, partial credit may be given for partially working solutions on robot.

## 0. Get your assigned Turtlebot4
1. Check the lab group sign-up spreadsheet (under Lab 0 on Brightspace) to get your robot ID for you and your partner.
2. The robot ID determines the robot's **ROS domain ID**, **Discovery Server ID**, and **IP address**. For Robot XX, the IP address is 192.168.1.1XX. This ID number is printed on the robot.

## 1. Turn on Turtlebot4
1. Place the Turtlebot4 on the charging station.
2. When the robot gets power, the light ring will turn on (white).
3. Wait 1-2 minutes for it to boot. It will chime when it is ready.
    - If the light ring turns a different color, it could be low battery, failed to connect wifi, or some other issue. Refer here: [Create3 Buttons and Light Ring Docs](https://iroboteducation.github.io/create3_docs/hw/face/)
    - If it repeatedly undocks from the docking station, press the dock button (1 dot) on top of the Turtlebot. It will attempt to autonomously dock. You must wait for it to finish. It might help to hold the docking station in place. The robot might get distracted by other docking stations. The undock button (2 dots) will undock the robot.

## 2. Connect to the Turtlebot4 via the ME597_TB4 network
1. First, connect your pc to ME597_TB4. The password is `turtlebot4`. If you are using a Virtual Machine, follow the VM network instructions at the bottom of the page.
2. The robot will automatically connect to ME597_TB4 on startup. 
3. Verify that your PC can reach the TurtleBot 4: `ping 192.168.1.1XX` (XX is your robot ID number)
    - It should say something like `64 bytes from 192.168.1.1XX: icmp_seq=1 ttl=64 time=33.3 ms`
    - If it says `destination host unreachable`, your PC cannot reach the robot via wifi.
    - Be patient. The TurtleBot4 must be fully booted. If it has not connected after a couple minutes, try restarting the TurtleBot4 and moving it closer to the router.

# ROS2 configuration
Recall from your lab 2 reading of [Turtlebot4 Networking](https://turtlebot.github.io/turtlebot4-user-manual/setup/networking.html), that connecting multiple devices in ROS2 has two primary configurations: Simple Discovery and Discovery Server. Below are instructions for Discovery Server.

## 3. Communicate with TurtleBot4 via ROS2: Discovery Server
1. Change your VM Network to **Bridged** (See VM Network Instructions below)
2. In `~/.bashrc`, comment out any manually configured `ROS_DOMAIN_ID` (# export ROS_DOMAIN_ID=XX) and set `export ROS_LOCALHOST_ONLY=0`. The Discovery Server setup script will configure `ROS_DOMAIN_ID` automatically.
3. Do `User PC Setup` here: [Setup Discovery Server](https://turtlebot.github.io/turtlebot4-user-manual/setup/discovery_server.html#user-pc). After calling the setup script, input the following response values for their corresponding settings (XX is your robot ID number):
```
ROS_DOMAIN_ID: XX
Discovery Server ID: XX
Discovery Server IP: 192.168.1.1XX
Discovery Server Port: [Enter]
done (d)
```
4. To test, do `ros2 topic list` in your PC. TurtleBot4 robot topics should be available.

    Is ros2 topic list not getting the robot topics?
    - Make sure you sourced your .bashrc file: `source ~/.bashrc`
    - Restart the ros2 daemon: `ros2 daemon stop; ros2 daemon start`
    - Do `ros2 topic list` twice.
    - Check again your connection to the TurtleBot4 using previous steps.
    - Check your PC configuration with `printenv | grep -i ros`
    - Make sure ROS_LOCALHOST_ONLY=0 (Only when communicating with the TurtleBot4!)
    - Power cycle the robot
    - Refer to debugging steps below
 
5. Changing robots: If you switch to a different TurtleBot 4, rerun the Discovery Server configuration script using the new robot's ID and IP address. The script updates /etc/turtlebot4_discovery/setup.bash.
6. When you are done using the physical robot, you must deactivate the discovery server settings for ROS2 to work with the simulator. See the instructions below to switch configurations.

## Switching between Simulator and Robot (Discovery Server)
Your environment needs two different configurations for using the simulator and for connecting to the physical robot. To change between these, you need to modify your `~/.bashrc`. Below are snippets of what your `~/.bashrc` should look like:

### 1. To use Simulator:
```
source /opt/ros/humble/setup.bash
export ROS_LOCALHOST_ONLY=1       # Disables communication with other devices on the network
export ROS_DOMAIN_ID=XX           # Your student domain id here

### Comment out this line if you have it:
# source /etc/turtlebot4_discovery/setup.bash
```

### 2. To connect to Robot (Discovery Server):
```
source /opt/ros/humble/setup.bash
export ROS_LOCALHOST_ONLY=0       # Enables communication with other devices on the network
source /etc/turtlebot4_discovery/setup.bash  # Applies configuration specified in step 3B.1

### Comment out this line if you have it:
# export ROS_DOMAIN_ID=XX
```

Remember to open a new terminal each time you switch. You may have to do `ros2 daemon stop; ros2 daemon start`

## VM Network Instructions
Getting internet connection on a VM should be handled automatically by the VM, however, for ROS communication to work between your VM and another device, the VM Network Adapter must be in "Bridged" mode. In Bridged mode, the VM’s network adapter is connected directly to the physical network, as if it were a separate physical machine. The VM gets its own IP address on the local network, similar to other devices (like your physical computer, printers, etc.). 

For context, the default mode is probably NAT (Network Address Translation). In NAT mode, the VM is "hidden" behind the host machine’s IP address. The VM shares the host machine’s network connection and IP address, but it is assigned a private IP address within a virtual network managed by the hypervisor (e.g., VirtualBox, VMware). This causes issues with ROS communications because the VM is not directly accessible from the outside network.

To change this, do the following:
1. Shut down your VM safely, if it is running.
2. Click "Edit virtual machine settings" --> Network Adapter --> Bridged: Connected directly to the physical network --> OK 
![vmware-bridged-network-adapter.png](images/vmware-bridged-network-adapter.png)
3. Start VM and check if your VM has access to the network (ping google.com or open a browser). University wifi (eduroam and PAL3.0) likely won't work. Use the ME597_TB4 network.
![images/vmware-bridged-network-adapter-working.png](images/vmware-bridged-network-adapter-working.png)
4. You can follow these same steps to change back to NAT settings.

If this did not work, follow these steps from [this tutorial](https://onlinecomputertips.com/support-categories/pc-troubleshooting/vmware-workstation-bridged-connection-fix/):
1. Shut down your VM safely
2. Click "Edit" --> Virtual Network Editor...
3. Click "Change Settings"--> Yes to allow admin privileges
4. Change VMnet0 to Bridged, then click "Automatic Settings..." and de-select all of the adapters except your wifi adapter. (If you are not sure which one, you can go to your PC's wifi settings and click hardware properties)

![images/vmware-bridged-network-adapter-debug.png](images/vmware-bridged-network-adapter-debug.png)

5. Last, re-do steps 2 and 3 of the original instructions above ("Edit virtual machine settings" ...)

## Debugging the TurtleBot4 via ssh
As a last resort: If you have verified connection to wifi, pinged your robot, double checked your configuration, power cycled the robot, and you suspect an issue with the robot, you may use the following to debug:
1. SSH into the robot: `ssh ubuntu@192.168.1.1XX`. The password is `turtlebot4`

    ```
    DO NOT create or modify any files in the TurtleBot4.
    DO NOT upgrade or re-install any TurtleBot4 packages. 
    ```` 

    Remember, for assignments, you will use ROS2 on your PC and communicate via ROS2 topics and services with the TurtleBot4.
1. Check topics and nodes with `ros2 topic list` and `ros2 node list`
1. Do `turtlebot4-setup` 
    - DO NOT apply any new settings - if you think the configuration is wrong, consult a TA.
    - You may check the status of the TurtleBot4 upstart job and restart it if necessary
    - You may view the current settings
