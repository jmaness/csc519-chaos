# Overview
The following shows the results of running several experiments described
in the CSC 519 Chaos workshop from https://github.com/CSC-DevOps/Chaos.

Most of the lessons learned are recorded in the screencast below, but here
are also some screenshots and highlights.

# Screencast
https://drive.google.com/open?id=1JLYVBfm1kkxVDSdl7tGf_gxSGBsJmgO1

# Experiments
## 1. CPU Burn
When CPUs are fully utilized by other processes on the host, latency of processes generally increases.

![CPU Burn - CPU](/images/cpu_burn_01.png)
![CPU Burn - Latency](/images/cpu_burn_02.png)

## 2. Network traffic corruption
Latency was not necessarily high when network traffic corruption occurred potentially because the
transactions (e.g. HTTP requests fail when the network traffic). This resulted in spikey CPU and latency.

![Network traffic corruption - Spikey CPU](/images/network_corruption_01.png)
![Network traffic corruptions - Comparison of corrupted network traffic vs uncorrupted](/images/network_corruption_02.png)

## 3. Killing/Starting containers

When two of the three container are stopped, the only running container on the green canary VM is overwhelmed and
its health turns red. There are significantly more failed transactions.

![Killing containers - The only running green container is unhealthy as it processes more requests than it can handle.](/images/kill_01.png)
![Killing containers - app1 and app2 are stopped. Availability is much lower, and number of failed transactions is much higher.](/images/kill_02.png)

## 4. Restricting CPU and Memory
I encountered a problem running the `app1` and `app2` containers with less than 16 MB of memory, but I was able to
reduce the CPUs to 0.25 which demonstrated significant latency and poor health of the containers as they failed
to process all requests.

![Restricting CPU and Memory - Latency is higher, health is red](/images/restrict_01.png)

## 5. Fill Disk 

This experiment showed a few interesting points:

1. The latency of the green canary was higher while the script to fill the 
disk was running
2. Because of the particular implementation of the containers running on the
green canary, the containers were able to continue running even without 
disk space. 
3. Most importantly, because there were no configured disk limits on the green
canary containers, the overlay file system filled up the underlying host file sysystem. This prevented operations such as `docker exec` operations on running
containers.

![Fill Disk - All file system is used but able to run app3 container](/images/disk_01.png)
![Fill Disk - Unable to exec a command in a container because no disk space available](/images/disk_03.png)
![Fill Disk - All host file system is used](/images/disk_04.png)


# Reflection

Q. How could you extend this workshop to collect more measures and devise an automated experiment to understand which event/failure causes the most problems?

A.

1. Define the list of measures (latency, CPU, transaction rate, number of successful transactions) and 
how to collect each. For example, the metrics can be stored in a database associating the metric, the VM, container, etc., scenario.
1. Define the list of scenarios, and how to execute the steps for each scenario to configure the systems under test. 
1. Execute each scenario: 
    1. Run an Ansible playbook to configure each system for the initial state of the scenario.
    1. Run the scenario
    1. Collect the metrics, and store them in the database.
1. Analyze the results, and select the event/failure that maximizes (or minimize where appropriate) the metrics.


