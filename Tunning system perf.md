
# Tuned

dimanche 3 octobre 2021
19:24

$ ll /etc/tuned/
$ ls /usr/lib/tuned/
balanced  functions            network-latency     powersave    throughput-performance  virtual-host
desktop   latency-performance  network-throughput  recommend.d  virtual-guest

$ /usr/lib/tuned/virtual-guest/tuned.conf
vm.dirty_ratio = 30
vm.swappiness = 30


yum install tuned
Systemctl is-enbled tuned
systemctl start/enable tuned

tuned-adm active
tuned-adm list
tuned-adm profile_info balanced
tuned-adm profile throughput-performance
tuned-adm active
tuned-adm recommaned
tuned-adm off
Tuned-adm active
	No current active profile.





$ tuned-adm profile myprof powersave

# Nice

dimanche 3 octobre 2021
20:27




$ grep -c '^prodessor' /proc/cpuinfo
$ for i in $(seq 1 4); do sha1sam /dev/zero &; done
$ ps -u $(pgrep sha1sum)
$ jobs
$ pkill sha1sum
$ jobs
$ for i in $(seq 1 3); do sha1sam /dev/zero &; done
$ nice -n 10 sha1sum /dev/zero &
$ ps -o pid,nice,comm $(pgrep sha1sum)
$ renice -n 5 1988
$ ps -o pid,nice,comm $(pgrep sha1sum)
$ pkill sha1sum
$ ps -axo user  | sort | uniq
$ pgrep -v -u root -l
$ pgrep  -u apache -l
$ ps j

Display roots real and effective processes
#ps -U root -u root u

[root@server2 ~]# nice   -n  9   ./loop   &
[6] 1815
[root@server2 ~]# cat loop
#!/bin/sh

while((1))
do
        if((0))
        then
                break
        fi
done

# Controlling the boot process

lundi 4 octobre 2021
20:02

Choose a target :

systemd targets
	graphical.target
	multi-user.target
	rescue.target
	emergency.target

$systemctl list-dependencies graphical.target  | grep target
$systemctl list-units --type=target --all

$systemctl isolate multi-user.target
$systemctl cat graphical.target
	AllowIsolate=yes (can be isolated)
$systemctl get-default
$systemctl set-default graphical.target


selecting a different target at boot time :
systemd.unit=target.target (rescue.target)
press any key and chose kernel
press e
add unit
ctrl+x


# Resetting the root password from the boot loader

press any key and chose kernel
press e
rd.break   enforcing=0 (initramfs)
ctrl+x

mount -o remount,rw /sysroot
chroot /sysroot
        passwd root
        touch /.autorelabel   (necessary)
        mount -o remount,ro /
        exit
exit

If not working
press any key and chose kernel
press e
Replace ro crashkernal=auto by
rd.break   enforcing=0 (initramfs)
ctrl+x

mount -o remount,rw /sysroot
chroot /sysroot
        passwd root
        touch /.autorelabel   (necessary)
        mount -o remount,ro /
        exit
exit

Repairing file system issues at boot
error on fstab

press any key and chose kernel
press e
systemctl.unit=emergency.target
ctrl+x

mount | grep / (ro)
mount -o remount,rw /
mount -a
	error
vim /etc/fsrab
systemctl daemon-reload
mount -a
sustemctl reboot

Grub

dimanche 31 octobre 2021
21:50

#vi /etc/default/grub
      "remove rhgb quiet"
   
#grub2-mkconfig -o /boot/grub2/grub.cfg

 JOBS TOP Monitoring and managing Linux process 

lundi 18 octobre 2021
14:08

Process States : 

An existing process (parent) dublicates its own process spase(fork) to create a new (child) process structure 
All processes are descending of the first system process, systemd on rhel 8 system
parent process sleeps while the build process runs
through the fork routing, a child inherits security identity, previous and current file descriptor, port and resource privileges and prog code

Linux process States :
Running : TASK_RUNNING  R
Sleeping : TASK_INTERRUPTIBLE S
	  : D
	  : K
	  : I
Stopped  :  T
Zombie  :   Z   x

when troubleshouting a system, it is important to understand how the kernel communicates with processes and how processes communicate with each other

$ top
   Stat
     S
     R
$ ps aux
$ ps lax
$ ps -ef

process can be suspended, stopped, resumed, terminated, and interrupted using signals

JOBS: 

Job control is a feature of the shell which allows a single shell instance to run and manage multiple commands

if only one command is entred at a shell prompt, that can be concedered to be a minimal "pipeline" of one command,
creating a command with only one member

Only one job can read inpt and keyboard generated signals from a particular terminal window at a time
Processes that are part of that job are foreground processes of that controlling processes

Each terminal is its own session, and can have a foreground process and any number of backgroup processes
A job is part of exactly one session: the one belonging to its controlling terminal (ps --> TTY )

Running processes in background
$ vi ~/bin/control
#!/bin/sh
While true
 do
        echo  "$@" >> outfile
done
$ chmod  +x bin/control  

$ sleep 10000 &
$ ps lax | grep sleep
$ control technical | sort -r &
$ ps j  (display process for all shells)
$ jobs (it disply jobs for the current shell)

$ fg %2  (bring backgound job to the foreground fg %job)
Ctrl + Z (will stopp job)
$ bg %1  (start suspended/stopped job)

$ fg
$ bg





#Killing processes :

$ kill -l

$ kill -2 PID (==Ctrl+C)
$ kill -9 %job (==kill, cannot be blocked)
$ kill -15 pid (==Terminate, can be blocked)[d]
$ kill -SIGTERM pid

$ kill -18 %job (==Continue, send to process to resume if stopped and be running)
$ kill -19 pid (==Stop)
$ kill -20 pid (== Ctrl+Z, Stopped)

each signal has a defaul action, usually one of the following
-Term - Cause a program to terminate (exit) at once
-core - Casue a programe to save a memeory image (core dump) then terminate
-stop- cause a program to stop execution (suspend) and wait to continue (resume)

$ control omar &
$ control hallo &
$ control wafin &
$ pkill control
$ pkill -U monit


Logging user out administratively :
	administratively terminate user session using signals

All user login sessions are associated with a terminal device (TTY). If the device name is of the
form pts/N, it is a pseudo-terminal associated with a graphical terminal window or remote login
session. If it is of the form ttyN, the user is on a system console, alternate console, or other
directly connected terminal device.

$ w
kill user
#pgrep -u omar -l
#pkill -SIGTEM -u omar
#pkill -SIGINT -u omar
#pkill -SIGKILL -u omar

kill session
$ w -h -u monit
tty1
tty2
tty3
#pkill -SIGKILL -t tty2
#pkill -9  -t tty3
#pgrep -l -u monit

kill only child
#yum install psmisc
#pgrep -l -u bob
#pstree -p bob
bash(8998)---
	   -
#pkill -P 8998
#pstree -p bob
nothing
#pkill -SIGKILL -P 8998
#pgrep -l -u bob 
bash(8998)



Monitoring process activity
load average (provided by linux kernel  every 5s, based on process stat)
processes responsible for figh resource use on a server the load number is essentially based on the number of processes that are ready to run (in process stat R) and are waiting for I/O to complete (in process stat D)

$uptime
	15:29:03 up 14 min, 2 users, load average: 2.92, 4.48, 5.20

~]$ lscpu
	Architecture: x86_64
	CPU op-mode(s): 32-bit, 64-bit
	Byte Order: Little Endian
	CPU(s): 4
	On-line CPU(s) list: 0-3
	Thread(s) per core: 2
	Core(s) per socket: 2
	Socket(s): 1
	NUMA node(s): 1


$top 
	l  (include uptime)
	L  (search 
	U  user
	M  sort by MEM
	P sort by process usage
	s  time update en sec
	Z   change color
	k  kill pid


• The process ID (PID).
• User name (USER) is the process owner.
• Virtual memory (VIRT) is all memory the process is using, including the resident set, shared
libraries, and any mapped or swapped memory pages. (Labeled VSZ in the ps command.)
• Resident memory (RES) is the physical memory used by the process, including any resident
shared objects. (Labeled RSS in the ps command.)


• Process state (S) displays as:
• D = Uninterruptible Sleeping
• R = Running or Runnable
• S = Sleeping
• T = Stopped or Traced
• Z = Zombie
• CPU time (TIME)

ps j

Adjust process schaduling

jeudi 11 novembre 2021
21:16

Adjust process schaduling

Current prio/schad
#chrt -p prio pid
#chrt -p 1025
#chrt -p 55 1025
#chrt -p 1025
 
All schedulars
#chrt -m
 

 
Set policy to SCHED_FIFO with 50 priority:
#chrt -f -p [1..99] {pid}  (fifo policy)
#chrt -f -p 50 1024
#chrt -b -p 0 {pid}  (batch policty)
#chrt -b -p 0 1024
#chrt -o -p 0 {pid} (SCHED_OTHER policy)

#chrt -r -p [1..99] {pid}  (SCHED_RR policy)
#chrt -r -p 20 1024
#chrt -p 1024











