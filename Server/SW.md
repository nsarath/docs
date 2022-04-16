
### makefile

"makefile" is general Linux based utility to compile the source code into Binary. ( used on c, c++, go etc)
Similar to how "dockerConfig" file used

This should be executed relative to the location which to be used to create binary..

````
[root@cn2-aio1-node1 cn2]# make -C build/ agent_push^C

[root@cn2-aio1-node1 cn2]# git branch

  cn2-5349

* master

[root@cn2-aio1-node1 cn2]# docker images | grep agent

svl-artifactory.juniper.net/blr-data-plane/contrail-vrouter-agent/ravsingh-logsize/contrail-vrouter-agent         master              822e969e95b9   4 hours ago     374MB

10.204.88.241:5000/atom-docker/cn2/bazel-build/dev/contrail-vrouter-agent                                         master              822e969e95b9   4 hours ago     374MB

[root@cn2-aio1-node1 cn2]# docker tag 822e969e95b9 svl-artifactory.juniper.net/blr-data-plane/contrail-vrouter-agent/ravsingh-logsize/contrail-vrouter-agent:master ^C

[root@cn2-aio1-node1 cn2]# docker push svl-artifactory.juniper.net/blr-data-plane/contrail-vrouter-agent/ravsingh-logsize/contrail-vrouter-agent:master^C
````


### docker build 

````buildoutcfg

root@pxeocp:~/runT/atom/naas/test# docker build -f docker/Dockerfile . -t svl-artifactory.juniper.net/contrail-nightly/cn2-test:cn2ocp
Sending build context to Docker daemon    204MB
Step 1/27 : FROM svl-artifactory.juniper.net/atom-docker-remote/ubuntu:focal
---> ff0fea8310f3
Step 2/27 : RUN apt -q update
---> Using cache
---> ff6c2c27c843
::
::
::
Removing intermediate container bfc311fa7b25
---> a1d856ed6b97
Successfully built a1d856ed6b97
Successfully tagged svl-artifactory.juniper.net/contrail-nightly/cn2-test:cn2ocp
root@pxeocp:~/runT/atom/naas/test# docker images
REPOSITORY                                              TAG           IMAGE ID       CREATED         SIZE
svl-artifactory.juniper.net/contrail-nightly/cn2-test   cn2ocp        a1d856ed6b97   6 seconds ago   1.61GB
svl-artifactory.juniper.net/atom-docker-remote/ubuntu   focal         ff0fea8310f3   7 days ago      72.8MB
kindest/node                                            <none>        32b8b755dee8   10 months ago   1.12GB
hub.juniper.net/cn2-early/contrail-k8s-deployer         master-2708   40b008abcccc   52 years ago    79.8MB
root@pxeocp:~/runT/atom/naas/test# docker push svl-artifactory.juniper.net/contrail-nightly/cn2-test:cn2ocp
The push refers to repository [svl-artifactory.juniper.net/contrail-nightly/cn2-test]
364d0da89c45: Pushed
f0ceeb800088: Pushed
c30908bfd017: Pushed
aaee00efc93d: Pushed
c38f052c2127: Pushed
cd8a0e4db9ae: Pushed
ff89acd19dac: Pushed
794ee477dfac: Pushed
d92505debb55: Pushed
0c22881b5c6e: Pushed
516c2194a9b3: Pushed
e19cbe5c984c: Pushed
f732a7dea466: Pushed
3c946ed1e8d7: Pushed
2c15fe5d7ff0: Pushed
fffcd4e60ad4: Pushed
776d490c2a91: Pushed
f86f89526d1f: Pushed
e9dcf11a821f: Pushed
04d5fa485ab5: Pushed
c04b6c1bd298: Pushed
ea0e2049f672: Pushed
401f75b542e3: Pushed
867d0767a47c: Layer already exists
cn2ocp: digest: sha256:7f2011d85e3c539fbcc87d91078e1c1842722af2897f70c815fe9ab485f0aa15 size: 5368

````

### git patch

````buildoutcfg

first to create patch file "xxxxx.patch" from the Pycharm git-->createPatch

Then, take this file to the repository home and from there apply using "git apply xxxxx.patch"

````

