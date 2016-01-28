---
layout: post
title: Linux SWAP 和 MySQL 内存（译）
categories: [MySQL, Linux]
tags: [MySQL, Swap, Linux]
---

LINUX SWAP AND MYSQL MEMORY
After mysql installation, To make mysql server production ready we have to tune default mysql variables considering the hardware, production load, performance, durability etc.

But What about the optimizing  Linux OS settings  with mysql ?

mysql-linux1

Here it is, i encountered  an issue once when i was checking TOP output for mysqld process and found that the mysqld  memory continuously increasing. Also there was one more thing  that it was using swap memory even if there was enough RAM memory available. we have monitored for a week and this was causing an issue like slow query processing, performance and slowness on DB server.

To solve this problem we have to set correct swappiness

      To check vm.swappiness
                    cat /proc/sys/vm/swappiness    (Default value : 60 (Range 0 to 100))

   To set a new non-persistent value :
                     sysctl -w vm.swappiness=0

   To set a new persistent value :
                     add vm.swappiness=0 in the /etc/sysctl.conf file

All Set !!

About SWAP space:

The swappiness parameter controls the tendency of the kernel to move processes out of physical memory and onto the swap disk. Because disks are much slower than RAM, this can lead to slower response times for system and applications if processes are too aggressively moved out of memory.

swappiness can have a value of between 0 and 100
swappiness=0 tells the kernel to avoid swapping processes out of physical memory for as long as possible
swappiness=100 tells the kernel to aggressively swap processes out of physical memory and move them to swap cache
Following are basic instructions for checking swappiness, emptying your swap and changing the swappiness to 0:

To check memory usage status :

          free

To check the swappiness value:

         cat /proc/sys/vm/swappiness

To turn swapoff :
This will empty your swap and transfer all the swap back into memory. First make sure you have enough memory available by viewing the resources tab of gnome-system-monitor, your free memory should be greater than your used swap. This process may take a while, use gnome-system-monitor to monitor and verify the progress.

sudo swapoff -a

To set the new value to 0:

echo 0 | sudo tee /proc/sys/vm/swappiness

To turn swap back on:

sudo swapon -a