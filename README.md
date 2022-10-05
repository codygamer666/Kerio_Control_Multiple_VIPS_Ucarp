# Kerio_Control_Multiple_VIPS_Ucarp
## A Kerio Control (UTM Firewall) mod to allow multiple vips in ha mode

## Prerequisites:
  - Two running instances of kerio control 
  - HA enabled and at least 1 interface already with a virtual IP
  
## Step by step guide

1. Connect to kerio control with ssh

   Example with powershell --> ssh 0.0.0.0 -l root

2. Go to root dir
  ```
   / $
  ```

3. Make os r/w
  ```
  mount -o rw,remount /
  ```

4. Create script folder
  ```
  mkdir ha-pub
  ```

5. Open dir
  ```
  cd ha-pub
  ```

6. Create ucarp configuration files

  nano pub-vip-up.sh
  ```
  #!/bin/sh
  /sbin/ip addr add x.x.x.1/x dev ethx
  /sbin/ip addr add x.x.x.2/x dev ethx
  start-stop-daemon --start --pidfile /ha-pub/pids/ucarp.0.0.0.1 --make-pidfile --background --exec /usr/sbin/arping -- -q -b -I ethx -U 0.0.0.1
  start-stop-daemon --start --pidfile /ha-pub/pids/ucarp.0.0.0.2 --make-pidfile --background --exec /usr/sbin/arping -- -q -b -I ethx -U 0.0.0.2
  ```

  nano pub-vip-down.sh
  ```
  #!/bin/sh
  /sbin/ip addr del x.x.x.1/x dev ethx
  /sbin/ip addr del x.x.x.2/x dev ethx
  start-stop-daemon --stop --pidfile /ha-pub/pids/ucarp.0.0.0.1 --exec /usr/sbin/arping
  start-stop-daemon --stop --pidfile /ha-pub/pids/ucarp.0.0.0.2 --exec /usr/sbin/arping
  rm /ha-pub/pids/ucarp.0.0.0.1
  rm /ha-pub/pids/ucarp.0.0.0.2
  ```


7. Make the files executable
   ```
   chmod +x /ha-pub/pub-vip-up.sh /ha-pub/pub-vip-down.sh
   ```
  
8. Create pid dir
   ```
   mkdir /ha-pu/pids
   ```
  
9. Run the script
   ```
   ./pub-vip-up.sh
   ```
   
9b. Stop the script
     ```
     ./pub-vip-down.sh
     ```

9c. CRON
    The script will not run at startup with the current configuration... we need a cron job!
  
    - Appoint nano as the default editor:
     ```
     export EDITOR=nano
     ```
    - Open the crontab:
     ```
     crontab -e
     ```
    - Add the job and save:
     ```
     @reboot /ha-pub/pub-vip-up.sh
     ```
  
10. Repeat the same process with the second node

11. Create the first firewall rule

12. Turn off the master node and check the active connections section of the slave node to see if the ha works. When the master node is online again all the new connections will go to the master node.


## Real world example:

Subnet x.x.60.0/24, Firewall A x.x.60.1, Firewall B x.x.60.5, VIPS x.x.60.2 and x.x.60.3

In this situation I want to expose a service running inside the x.x.91.0/24 subnet, but I have other services to setup later on.

In order not to waste other ip addresses I follow this guide above with my parameters and my two VIPS are up and running.

After this I need to create a new set of firewall rules to map my service:

  
![new_rules](https://user-images.githubusercontent.com/96527590/187072908-b6c456cc-eb87-4e13-a968-697cd21262f8.jpg)



## Final Notes

This is an unofficial mod. It was tested with Kerio Control 9.3.6 patch 1 build 5808 and may stop working with future releases.

If you encounter any issues feel free to open a new issue.
