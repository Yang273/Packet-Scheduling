Packet-Scheduling
===========================
## Description
### Background
Packet scheduling can easily become the network system bottleneck because of the large number of buffered packets. Thus, to optimize network performance, packet scheduling is a good and important component.

### Use Case
Deploying the software package schedulers in the kernel module as qdisc (queueing discipline). To evaluate the traffic shaping effect, we use two AWS EC2 instances as the client and server, and use neper to generate a large number of flows between them. We measure the CPU overhead of different schedulers using `dstat`.

## Progress
### Research on Neper and BESS
- Neper (a load generator for linux kernel use case)
- BESS (a software switch for user space use case)

### Research on algorithm
- Priority Queue (cFFS)
- PIFO model extension  
-  Details about the above researches could be found in the [Midterm report](https://docs.google.com/document/d/1EVX2nIhreSNcIEetghqFkokzMUnvDQfx9hfE8U013dM/edit?ts=5e619bad#heading=h.lx1fcchkkaqo).
### **Current development (deploy trials)**
- Since the author didn’t explain the algorithm in detail, we decided to run one of the use cases first and see what could be found. We have found the linux-kernel implementation written by the author and want to deploy it on AWS.
- We follow the instructions [here](https://saeed.github.io/eiffel/).
  1. We tried both `working_ffs-based_qdisc` branch and `v4.10-gq` branch. And tried on m4.xlarge and m4.16xlarge. 
  2. In the step 3 of the tutorial, we follow the instructions [here](https://www.freecodecamp.org/news/building-and-installing-the-latest-linux-kernel-from-source-6d8df5345980/) to compile and install the designed version of linux kernel.
  3. To compile `working_ffs-based_qdisc` branch on m4.xlarge, we need another patch [here](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/diff/?id=474c90156c8dcc2fa815e6716cc9394d7930cb9c).
  4. To run the new algorithm, we should still use `fq` instead of `gq` because the author simply built the new implementation on top of `fq` as we noticed in his code.
- Result
  1. Fail to install on m4.16xlarge.
  2. Success to install on m4.xlarge and we got **results** here.
    ![image](https://github.com/14-760-S20-Team7/Packet-Scheduling/blob/master/image/FQ_Eiffel_CPU_Utilization.png)  
We perform the experiments on m4.xlarge, m4.2xlarge and m4.4xlarge on AWS. And here is the result of the experiment on m4.4xlarge. In this experiment, we started 1000 tcp flow with 4 threads using nepper. And we use `dstat` to collect the system cpu utilization every second. The result shown above supports the [author’s claim](https://www.usenix.org/system/files/nsdi19-saeed.pdf) that Eiffel reduces CPU overhead compared to FQ.

## Issues
We have met a lot of problems during the deployment of Eiffel, here is the problems and our solutions:
- **Fail to compile the Linux kernel**  
Solution: apply patch to the kernel.

- **Fail to deploy the custom queue as qdisc (Eiffel)**  
We followed the author’s instruction and tried to deploy the Eiffel queue as qdisc with command `tc qdisc add dev ens3 root gq`. But we find that the custom qdisc is not deployed by using `tc qdisc show`. Then we confirmed this by using `strace` and figured out that the system cannot find the target `.so` file and fails silently. Finally, we checked the scheduler Makefile and found out that the custom gq is actually deployed as `fq` instead of the `gq`.
Solution: Use `tc qdisc add dev ens3 root fq` to deploy instead.

- **Not fixed: fail to compile the Linux kernel on m4.16xlarge**  
We created an image of the provided Linux kernel on m4.large, and fail to use this image on m4.16xlarge. We also tried to compile this image on m4.16xlarge directly, but still fail after we reboot the machine. So far, we have no idea what cause this problem, and thus we decide to do the experiments on smaller machines.

- **Not fixed: fail to start 20k tcp flow and set the max pacing rate**  
We get an error when we try to start more than 1000 tcp flows with nepper. We need to do more research on nepper to fix this issue.



## Links:
[Eiffel](https://www.usenix.org/conference/nsdi19/presentation/saeed)  
[Qdisc implementation](https://github.com/saeed/eiffel_linux)  

