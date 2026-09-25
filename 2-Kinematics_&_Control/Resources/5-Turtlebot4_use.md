# Using the TurtleBot 4

## Robot Assignments

When lab is over, you must power off the TurtleBot and return it to the storage cabinet. Be mindful that this robot is shared with the other section. Regardless of how much you completed, record your progress. For robot assignments, partial credit may be given for partially working solutions on the robot.

## 0. Get your assigned TurtleBot 4

1. Check the lab group sign-up spreadsheet (under Lab 0 on Brightspace) to get the robot ID for you and your partner.
2. The robot ID determines the robot's **ROS domain ID**, **Discovery Server ID**, and **IP address**. For Robot `XX`, the IP address is `192.168.1.1XX`. This ID number is printed on the robot.

## 1. Turn on TurtleBot 4

1. Place the TurtleBot 4 on the charging station.
2. When the robot gets power, the light ring will turn on (white).
3. Wait 1–2 minutes for it to boot. It will chime when it is ready.
   - If the light ring turns a different color, it could indicate a low battery, failure to connect to Wi-Fi, or another issue. Refer here: [Create3 Buttons and Light Ring Docs](https://iroboteducation.github.io/create3_docs/hw/face/)
   - If it repeatedly undocks from the docking station, press the dock button (1 dot) on top of the TurtleBot. It will attempt to autonomously dock. You must wait for it to finish. It might help to hold the docking station in place. The robot might get distracted by other docking stations. The undock button (2 dots) will undock the robot.

## 2. Connect to the TurtleBot 4 via the ME597_TB4 network

1. Connect your PC to `ME597_TB4`. The password is `turtlebot4`.
   - If you are using a **Virtual Machine**, follow the [VM Network Instructions](#vm-network-instructions) below.
   - If you are using **WSL**, follow the [WSL Network Instructions](#wsl-network-instructions) below.
2. The robot will automatically connect to `ME597_TB4` on startup.
3. Verify that your PC can reach the TurtleBot 4:

   ```bash
   ping 192.168.1.1XX
   ```

   where `XX` is your robot ID number.

   - It should say something similar to:
     ```
     64 bytes from 192.168.1.1XX: icmp_seq=1 ttl=64 time=33.3 ms
     ```
   - If it says `destination host unreachable`, your PC cannot reach the robot via Wi-Fi.
   - Be patient. The TurtleBot 4 must be fully booted. If it has not connected after a couple of minutes, try restarting the TurtleBot 4 and moving it closer to the router.

# ROS 2 Configuration

Recall from your Lab 2 reading on [TurtleBot 4 Networking](https://turtlebot.github.io/turtlebot4-user-manual/setup/networking.html) that connecting multiple devices in ROS 2 has two primary configurations: Simple Discovery and Discovery Server. Below are instructions for Discovery Server.

## 3. Communicate with TurtleBot 4 via ROS 2: Discovery Server

1. **Before configuring the Discovery Server, make sure your network is configured correctly:**
   - **Virtual Machine:** Change your VM network to **Bridged** mode. See the [VM Network Instructions](#vm-network-instructions) below.
   - **WSL:** Configure WSL to use **mirrored networking**. See the [WSL Network Instructions](#wsl-network-instructions) below.

2. In `~/.bashrc`, comment out any manually configured `ROS_DOMAIN_ID`:

   ```bash
   # export ROS_DOMAIN_ID=XX
   ```

   and set:

   ```bash
   export ROS_LOCALHOST_ONLY=0
   ```

   The Discovery Server setup script will configure `ROS_DOMAIN_ID` automatically.

3. Complete the `User PC Setup` here: [Setup Discovery Server](https://turtlebot.github.io/turtlebot4-user-manual/setup/discovery_server.html#user-pc).

   After calling the setup script, input the following response values for their corresponding settings, where `XX` is your robot ID number:

   ```
   ROS_DOMAIN_ID: XX
   Discovery Server ID: XX
   Discovery Server IP: 192.168.1.1XX
   Discovery Server Port: [Enter]
   done (d)
   ```

4. Source your updated configuration:

   ```bash
   source ~/.bashrc
   ```

5. Test the connection:

   ```bash
   ros2 topic list
   ```

   TurtleBot 4 robot topics should now be available.

> ### `ros2 topic list` not showing the robot topics?
> 
> Try the following:
> 
> - Make sure you sourced your `.bashrc`:
>   ```bash
>   source ~/.bashrc
>   ```
> 
> - Restart the ROS 2 daemon:
>   ```bash
>   ros2 daemon stop; ros2 daemon start
>   ```
> 
> - Try `ros2 topic list` again.
>   
> - Verify that you can still ping your assigned robot:
>   ```bash
>   ping 192.168.1.1XX
>   ```
>   
> - Check your computer's IP address
>   ```bash
>   hostname -I
>   ```
>   If it does not match the robot's local network `192.168.1.XXX`, revisit [VM Network Instructions](#vm-network-instructions) below. **After changing network settings, rerun the TurtleBot 4 Discovery Server setup script** using your assigned robot ID and IP address.
> 
> - Check your ROS configuration:
>   ```bash
>   printenv | grep -i ros
>   ```
>   Make sure `ROS_LOCALHOST_ONLY=0` when communicating with the physical TurtleBot 4.
>   Double-check that the `ROS_DOMAIN_ID` and Discovery Server IP correspond to your **current robot**.
> - Power cycle the robot.
>
> **Can ping the robot but still can't see ROS 2 topics? Check the Windows firewall.**
> Temporarily disabling the firewall is an important troubleshooting step because Windows Firewall may block ROS 2/DDS traffic even though `ping` works.
> To temporarily disable it on Windows 11:
> **Settings → Privacy & security → Windows Security → Firewall & network protection**
> Select the network profile marked **(active)** and temporarily turn **Microsoft Defender Firewall** off. Then return to the terminal and try:
> ```bash
> ros2 daemon stop; ros2 daemon start
> ros2 topic list
> ```
> If the topics appear, the firewall was blocking the ROS 2/DDS communication. **Turn the firewall back on after troubleshooting** and consult a TA if needed.

6. **Changing robots:** If you switch to a different TurtleBot 4, **rerun the Discovery Server configuration script** using the new robot's ID and IP address. The script updates `/etc/turtlebot4_discovery/setup.bash`.

   After rerunning the script:

   ```bash
   source ~/.bashrc
   ros2 daemon stop; ros2 daemon start
   ```

7. When you are **done using the physical robot**, you must deactivate the Discovery Server settings for ROS 2 to work with the simulator. See the instructions below to switch configurations.

## Switching Between Simulator and Robot (Discovery Server)

Your environment needs two different configurations for using the simulator and for connecting to the physical robot. To change between these, modify your `~/.bashrc`.

### 1. To use Simulator

```bash
source /opt/ros/humble/setup.bash
export ROS_LOCALHOST_ONLY=1       # Disables communication with other devices on the network
export ROS_DOMAIN_ID=XX           # Your student domain ID here

# Comment out this line if you have it:
# source /etc/turtlebot4_discovery/setup.bash
```

### 2. To connect to Robot (Discovery Server)

```bash
source /opt/ros/humble/setup.bash
export ROS_LOCALHOST_ONLY=0       # Enables communication with other devices on the network
source /etc/turtlebot4_discovery/setup.bash

# Comment out this line if you have it:
# export ROS_DOMAIN_ID=XX
```

Remember to change the namespaces and to open a new terminal each time you switch, or run:

```bash
source ~/.bashrc
```

You may also have to restart the ROS 2 daemon:

```bash
ros2 daemon stop; ros2 daemon start
```

# Network Instructions

## VM Network Instructions

Getting an internet connection on a VM should be handled automatically by the VM. However, for ROS communication to work between your VM and another device, the VM Network Adapter must be in **Bridged** mode.

In Bridged mode, the VM's network adapter is connected directly to the physical network, as if it were a separate physical machine. The VM gets its own IP address on the local network, similar to other devices.

The default mode may be NAT (Network Address Translation). In NAT mode, VMware places the VM on a private virtual network and translates its traffic through the host. This is convenient for internet access but can interfere with ROS 2/DDS communication with physical devices on the external network.

To change this:

1. Shut down your VM safely if it is running.
2. Click **Edit virtual machine settings → Network Adapter → Bridged: Connected directly to the physical network**.
   - If available, select **Replicate physical network connection state**.
3. Start the VM and check whether it has access to the network. University Wi-Fi (`eduroam` and `PAL3.0`) likely will not work for communicating with the robots. Use the `ME597_TB4` network.
4. Check the VM's IP address:

   ```bash
   hostname -I
   ```

   When connected to `ME597_TB4` in Bridged mode, you should see an IP address on the same network as the robots, such as:

   ```
   192.168.1.xxx
   ```

   If you only see an address on a private VM network, such as `192.168.100.xxx`, check your Bridged network configuration.

5. You can follow the same steps to change back to NAT when you are no longer using the physical robot.

If Bridged mode does not work, follow these steps from [this tutorial](https://onlinecomputertips.com/support-categories/pc-troubleshooting/vmware-workstation-bridged-connection-fix/):

1. Shut down your VM safely.
2. Click **Edit → Virtual Network Editor...**
3. Click **Change Settings → Yes** to allow administrator privileges.
4. Change `VMnet0` to **Bridged**, then click **Automatic Settings...** and deselect all adapters except your Wi-Fi adapter.
   - If you are unsure which adapter is your Wi-Fi adapter, go to your PC's Wi-Fi settings and check its hardware properties.
5. Repeat the original Bridged network steps above.

## WSL Network Instructions

By default, WSL2 may use a private virtual network. This can result in an IP address such as `172.x.x.x`, even when the Windows host is connected directly to the `192.168.1.x` TurtleBot network. ROS 2/DDS communication may not work correctly in this configuration.

For WSL2, use **mirrored networking mode** so that WSL can communicate directly with devices on the same network as the Windows host.

1. On **Windows**, create or edit the following file:

   ```
   C:\Users\<your-Windows-username>\.wslconfig
   ```

   > **Note:** `.wslconfig` belongs in your **Windows user folder**, not inside the Ubuntu/WSL filesystem.

2. Add the following to `.wslconfig`:

   ```ini
   [wsl2]
   networkingMode=mirrored
   ```

3. Save the file.

4. Open **Windows PowerShell** and completely shut down WSL:

   ```powershell
   wsl --shutdown
   ```

5. Reopen your Ubuntu/WSL terminal.

6. Check your network configuration:

   ```bash
   hostname -I
   ```

   When Windows is connected to `ME597_TB4`, WSL should now be able to use the same local network as the TurtleBots rather than only showing a private `172.x.x.x` WSL network.

7. Verify that you can reach your robot:

   ```bash
   ping 192.168.1.1XX
   ```

## Debugging the TurtleBot 4 via SSH (For TA Use Only)

As a last resort, if you have verified the Wi-Fi connection, pinged your robot, double-checked your configuration, power cycled the robot, and suspect an issue with the robot, you may use the following steps to debug:

1. SSH into the robot:

   ```bash
   ssh ubuntu@192.168.1.1XX
   ```

   The password is `turtlebot4`.

   > **DO NOT create or modify any files on the TurtleBot 4.**  
   > **DO NOT upgrade or reinstall any TurtleBot 4 packages.**  
   > **DO NOT change the ROS domain or Discovery Server configuration on the TurtleBot 4.**

   Remember, for assignments, you will use ROS 2 on your PC and communicate via ROS 2 topics and services with the TurtleBot 4.

2. Check topics and nodes:

   ```bash
   ros2 topic list
   ros2 node list
   ```

3. Run:

   ```bash
   turtlebot4-setup
   ```

   - **DO NOT apply any new settings.** If you think the configuration is wrong, consult a TA.
   - You may check the status of the TurtleBot 4 upstart job and restart it if necessary.
   - You may view the current settings.
