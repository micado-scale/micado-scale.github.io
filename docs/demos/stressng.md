# stressng

This application deploys a single service - 
[`stressng`](https://wiki.ubuntu.com/Kernel/Reference/stress-ng)[^1] - 
which exercises a constant CPU load on the system.

A scaling policy is defined for this application that scales up/down both
nodes and the stressng service based on CPU consumption. Helper scripts have
been added to the directory to ease application handling.


[^1]: In the container [lorel/docker-stress-ng](https://github.com/Lorel/docker-stress-ng/).