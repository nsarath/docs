# loading Private image without re-deploying the setup
    
We cannot rely on the procedure like ‘stop/modifycontainer/start’ on any k8s based setups
 (no matter ocp or vanilla k8s). This is because all containers are managed by k8s and in case of container is stopped
  k8s decided to recreate it..

So, please find below the detailed procedure how to do this for example on Vrouter private image given,


   1) ssh to a node with agent
   2) find container id: crictl ps | grep agent
   3) on the host 
       cp new binary into /var/lib/contrail/ - this folder is bind to container: cp <newbin>/var/lib/contrail
   4) exec to container: crictl exec -it <id> bash
   5) replace /usr/bin/contrail-vrouter-agent with new binary
   6) reload agent w/o terminating container: kill -HUP 1


```buildoutcfg
(vrouter-agent)[root@master-2 /]$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  14208  4148 ?        Ss   Apr15   0:00 bash -c #!/bin/bash set -ex  function wait_file() {   local src=$1
root      159451  9.3  2.3 789652 554900 ?       Sl   Apr15  71:15 /usr/bin/contrail-vrouter-agent
root      256553  0.0  0.0  13440  3464 pts/2    Ss   08:52   0:00 bash
root      257230  0.0  0.0  53348  3896 pts/2    R+   08:54   0:00 ps aux

(vrouter-agent)[root@master-2 /]$ kill -HUP 1

(vrouter-agent)[root@master-2 /]$ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0  14208  4148 ?        Ss   Apr15   0:00 bash -c #!/bin/bash set -ex  function wait_file() {   local src=$1   
root      256553  0.0  0.0  13440  3464 pts/2    Ss   08:52   0:00 bash
root      257383 26.0  0.5 494092 137136 ?       Sl   08:54   0:00 /usr/bin/contrail-vrouter-agent


```

Note: main process in container is bash that handles HUP signal to reload agent (this is for handling 
config changes w/o containers recreation)

##### The above is the procedure which we been using during Classic Contrail days given by Development team (Credit to Alexey Morlang)