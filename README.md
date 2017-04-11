## Situation

Try to restart Tomcat when JVM clashed by "Out Of Memory" using ``` -XX:OnOutOfMemoryError ``` option,
but the script that I registred on OnOutOfMemoryError option did not make a tomcat alive.
The script is really simple. It looks like below one.
```bash restart-tomcat.sh
#!/bin/bash
catalina.sh stop
sleep 1
catalina.sh start
```

## Problem

What distrupt Tomcat starting was privous listen port. That port had not been closed event though Tomcat was stopped. 
Because, the listend port is inheited to child process that is restart script run by OnOutOfMemoryError option.
If you look up open ports by ``` netstat ```, you can see the port listen by sh process.
Actualy, I counld not find why Tomcat http port is not cleaned up.

## Solution

first, I did is finding that prevent inheit File Descriptors from jvm to sh process. I could not, unfortunatlly, find it.
Changed the approch, just tried to close socket that is from jvm in restart script. So, the script of restarting tomcat was changed as 
```bash restart-tomcat.sh
#!/bin/bash
catalina.sh stop
sleep 1
# close File Descriptors that associated with socket.
catalina.sh start
```
After close socket fourced, Tomcat was started successfully.

This is smaple the solution script that I made
```bash restart-tomcat.sh
#!/bin/bash
catalina.sh stop
sleep 1
FDS=`ls -al /proc/$BASHPID/fd | grep socket | grep -v grep | awk '{print $9}'`
for FD in $FDS; do
  eval "exec $FD>&-"
done
catalina.sh start
```
