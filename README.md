# Kerio_Control_Multiple_VIPS_Ucarp
## A Kerio Control (UTM Firewall) mod to allow multiple vips in ha mode

## Prerequisites:
  - Two running instances of kerio control 
  - HA enabled and at least 1 interface already with a virtual IP
  
## Step by step guide

1) Connect to kerio control with ssh

   Example with powershell --> ssh 0.0.0.0 -l root

2) Go to root dir
  ```
   / $
  ```

3) Make os r/w
  ```
  mount -o rw,remount /
  ```

4) Create script folder
  ```
  mkdir ha-pub
  ```

5) Open dir
  ```
  cd ha-pub
  ```

6) Create ucarp configuration files

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


7) Make the files executable
  ```
  chmod +x /ha-pub/pub-vip-up.sh /ha-pub/pub-vip-down.sh
  ```
  
8) Create pid dir
  ```
  mkdir /ha-pu/pids
  ```
  
9) Run the script
  ./pub-vip-up.sh

9.1) Stop the script
  ```
  ./pub-vip-down.sh
  ```
  
10) Repeat the same process with the second node
