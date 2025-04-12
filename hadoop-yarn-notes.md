### Differnce beteen haddop 1 and hadoop 2?
In hadoop 1 we can only run Map reduce related application ~ Hive, uzi etc
**Job tracker**-- Reseoucmenanager  scheduling and tracking all the application running on the cluster -- single point of failure
**Task tracker**-- runs on the worker node- needs to manage progress of the jobs and sends the progress back to the Job tracker
slot --> Are fixed # of slots on Cluster-- slots are computation units of MR1
            1. slots are fixed on map or reduce
            2. We may not be able to utilize the cluster completely
            3. can be reused by the once the application is finished
            
In hadoop 2 we can run application like spark and ML related work
 Resource manager
 Node manager
 Application master
 Job history server
 containers

In hadoop 2.0 we have hdfs; on top hdfs we have yarn(resource manager layer) can run --spark, hive on tez

| Application State | Description                                             |
|-------------------|---------------------------------------------------------|
| NEW               | Application which was just created.                     |
| NEW_SAVING        | Application which is being saved.                       |
| SUBMITTED           | Application which has been submitted.                     |
| ACCEPTED            | Application has been accepted by the scheduler.          |
| RUNNING             | Application which is currently running.                 |
| FINISHED            | Application which finished successfully.                |
| FAILED              | Application failed.                                     |
| KILLED              | Application was terminated by a user or admin.          |

[Source](https://hadoop.apache.org/docs/r2.4.1/api/src-html/org/apache/hadoop/yarn/api/records/YarnApplicationState.html#line.34)

**
**What is a container of yarn?
1. Fixed amount memory+cpu on any task yarn.
2. There is no fixed number of 
3. you cannot run the same application gain in the container

**In which scenarios the containers are killed in hadoop 2.x?**
1. Out of Memory:If a container uses more memory than it was allocated, it will be killed. This is like if you 
tried to pour too much water into a glass - it overflows and you have to stop.
2. Exceeding Time Limit:If a container runs for longer than it's supposed to, it gets killed.
3. Node Failure:If the machine (node) running the container crashes or loses connection, all its containers are killed.
4. Application Finish:When your Hadoop job is done, all its containers are killed. This is like cleaning up after finishing a project
5. User Request:If a user or administrator manually stops a job, its containers are killed.
6. Resource Contention:In some cases, if the cluster needs resources for more important jobs, it might kill containers from less important jobs
7.Container Failure:If the process inside the container crashes due to a bug or error, the container is killed
8.Preemption: Higher priority applications may cause lower priority containers to be killed to free up resource
9.Application Master Failure: If the Application Master (AM) fails, all associated containers are typically terminated

## Anatomy of yarn application:
1. Application client -- submits the yarn application request to RM
2. RM will create the application ID and talk with the Node manager to launch a container
3. Application master is launched on the first container 
4. Application master will request the resources to the resource manager
5. Based upon the request of Application manager to Resource manager, AM will request the NM to laucnh containers
6. AM will continuously ask for the resources until the application is completely
7. The clients will continuously poll the status of the job
8. After the job is complete the the client will exit 

**Concise explanation of the anatomy of a YARN application run:**
Key Components
**Resource Manager (RM):** One per cluster, manages resources across the cluster.
**Node Managers (NMs):** Run on all cluster nodes, launch and monitor containers.
**Containers:** Execute application-specific processes with constrained resources.
**Application Master (AM):** Manages the application's lifecycle and resource needs.
Application Run Process
**Client Submission:**
Client contacts the Resource Manager
Requests to run an Application Master process
**Application Master Launch:**
RM finds a Node Manager to launch the AM in a container
**Resource Negotiation:**
AM requests resources (containers) from the RM
RM allocates containers based on availability and constraints
**Container Launch:**
AM communicates with Node Managers to launch containers
Containers execute application-specific tasks
**Application Execution:**
Containers perform the actual work
They maintain contact with the AM
**Progress Monitoring:**
AM monitors the progress of the application
Client can communicate with AM for status updates
**Completion:**
When the application finishes, AM deregisters with RM
AM shuts down, allowing its container to be repurposed
**Key Points to Remember**
YARN separates resource management (RM) from application management (AM)
AMs can request resources dynamically during runtime
Containers can run various processes, not limited to Java applications
Resource allocation is late-binding, allowing flexible use of resources
YARN supports different application lifespans, from short-lived to long-running shared applications
Understanding this process demonstrates YARN's flexibility in managing diverse distributed applications efficiently13.


## Scheduler in YARN
1. What is the purpose of the scheduler of yarn?
    weather resources are available on the que
2. How do yarn know the utilization of the cluster?
    Node manager sends heartbeat to the Resource manager about the availability of the resources, like how much memory is available (can be seen in the log)
    
    ### FIFO scheduler:
    2. The job will take resources on first come first serve basis
    1. other job will have to wait
    
 ## YARN FIFO Scheduler: Benefits and Drawbacks

| **Benefits**                                     | **Drawbacks**                                      |
|--------------------------------------------------|--------------------------------------------------|
| 1. Simple to understand and implement           | 1. Can lead to resource underutilization         |
| 2. Predictable job execution order              | 2. Long-running jobs can block shorter jobs      |
| 3. Works well for similar-sized jobs            | 3. Not suitable for mixed workloads              |
| 4. Low scheduling overhead                      | 4. Lacks fairness in resource allocation         |
| 5. Good for batch processing workflows          | 5. No priority-based scheduling                  |
| 6. Easy to reason about job completion times    | 6. Can't handle varying resource requirements efficiently |
| 7. Suitable for single-user clusters            | 7. Poor for multi-tenant environments            |


### Capacity scheduler
1. Can be divided into different organizations 
2. What is the drawback to the capacity scheduler?
YOU CAN NOT be able to utilize whole resources to the cluster 
Y
Fair scheduler draw back
1. If preemption is aggressive 
2. Fair scheduler has ques

There is no write answer to choose the best scheduler?

What is the use case of Node labels?
1. when you want to launch the AM to core nodes?
2. what is a ---node label partition

YARN FEATURES
1.  queues
2.  Node labels
3.  Delay Scheduling
4.  Dominant Resource Fairness DRF

## Delay Scheduling
Does YARN on EMR support Data locality? YEs
What does it mean to data locality?-- making the computation near the data
Delay scheduling ?-- wait for a few sec where the data resides -- will be much faster
Available in capacity and fair scheduling?
How does yarn know where to launch the map task?
How does yarn know where the data is present--> by node manager
Does Data locality come into picture when the data is in S3--> No -- the delay scheduling is not needed
delay scheduling, and it is supported by both the Capacity Scheduler and the Fair Scheduler.

Dominant Resource Fairness DRF
By default yarn will look for memory
With DFS yarn will use Memory + CPU
how does yarn calculate faireness of resources
1:1 for faireness only for the dominant resource

Imagine you're sharing a pizza and soda with friends. Some friends are hungrier (want more pizza), while others are thirstier (want more soda). DRF is like making sure everyone gets their fair 
share of what they want most.
Here's how it works in YARN:
Resources: YARN manages different resources like CPU and memory.
Dominant Resource: For each application, YARN figures out which resource it needs most. This is the "dominant resource."
Fair Sharing: YARN tries to give each application an equal percentage of its dominant resource.
Balancing Act: If one app needs more CPU and another needs more memory, YARN gives the CPU-hungry app more CPU and the memory-hungry app more memory.
Calculation: YARN looks at what percentage of the total cluster resources each app is using for its dominant resource. It tries to make these percentages equal across all apps.
Example:
App A needs 2 CPUs and 5 GB memory
App B needs 1 CPU and 10 GB memory
In a cluster with 10 CPUs and 100 GB memory:
For App A, CPU is dominant (20% of CPUs vs 5% of memory)
For App B, memory is dominant (10% of memory vs 10% of CPUs)
YARN would try to give App A 20% of CPUs and App B 20% of memory
This way, each app gets a fair share of what it needs most, making efficient use of the cluster's resources.
Related

## Log aggregation
Where are the container logs initially stored?
1.initally it will store yarn will store it in the local node and then it will be aggregated to the HDFS by the help of nodemanagr
2. By default EMR has log aggregation enabled
important setting for log aggregation
1.to enable log agggregation-- yarn-log-aggregation-enabled true-false
2.Where to store container logs --yarn.nodemanagr.log-dir-->yarn/log/haddoop-yarn/containers
3. Where to aggregate the logs too yarn.nodemanagr.remote-app-log-dir--->/var/log/hadoop-yarn/apps

if the yarn application is still running where will the client get the logs from?
Local directory -- logs are not pushed from the HDFS
If the job is completed where would the it search the logs?
it will check the local directoy and then it will check the HDFS or the log aggreation time will be passed:

** we can configure the log aggregation location to S3 but the yarn will not fetch the logs from s3 **

You have a very long running application do you think your persistent application will open or not?
yes it will show the logs and collect from the local node for both spark and yarn timeline server

What is the use of yarn timeline server?
if the RM is killed then it will pick logs will be timeline server

How will log aggregation work when the log aggregation is kept to s3?
log will still be kept in local hence log4j properties will be expected 

Higher availability 

1. 1 active RM and 2 standby RM
RM restart are of 2 kind
    1. Non work preserving restart i
    it will start from the beginig of the application
    2. Work preserving restart
    All the is failed or restartred -- will start from where the it left the job

RM state store

RM stores all the state to zookeeper 
Active RM allways write the store to Embeded zookeper

Multi master Failover:

1. Automatic Failover
If the RM of Restart due to some issue -- it can failover to the another RM
2. RM failoverProvider
Change the state of the active RM to standby RM and then the state of the standby RM to active

```
yarn.clinet.failover-proxy-provider = org.apache.hadoop.yarn.clinet.configuredRMFailoverProxyProvider
```
## Fault Tolerance

IMPORTANT

1. Task failure
AM retriggered the TASK attempt
Comfigured max task failure fails the entier application
    default attempt at 4 if the task is killed 4 times then the application is klilled 
2. AM failure
RM retrigers another attempt
Max configured failure fails the entire application
3.Node Manager Failure
RM exclude the node from the cluster
Application master kills the container on the node
-- Impact will be the containers will be impacted they will be killed
-- if the Node manager keeps on failing then the node will be keep in the exclude list 
4. ResourceManager failover
High availability enable


When the Resource Manager fails, the currently running job will be interrupted. The application will lose its coordination point.
Application Master (AM) Disconnect:
The Application Master, which manages the job's execution, will lose connection to the Resource Manager.
No New Resource Allocation:
During the Resource Manager downtime, no new resources (containers) can be allocated to the job.
Existing Tasks Continue:
Tasks that were already running on NodeManagers will continue to execute, but they won't be able to report their status or request new resources.
**Recovery Process:**
If EMR is configured for high availability (HA) with multiple master nodes, a standby Resource Manager will take over.
Without HA, the Resource Manager will need to be restarted manually or automatically (depending on configuration).
**Job Fate:**
With HA: The job can often recover and continue from where it left off, with minimal data loss.
Without HA: The job will likely fail and need to be restarted from the beginning, unless it has built-in checkpointing mechanisms.
**Application Resubmission:**
In many cases, especially without HA, the application will need to be resubmitted to the cluster once the Resource Manager is back up.
Data Consistency:
There might be some data inconsistency or duplication, especially for jobs that were in the middle of writing output when the failure occurred.

In summary, a Resource Manager failure can cause significant disruption, but the exact impact depends on the EMR cluster's configuration (especially HA setup) and the application's ability to handle such failures. With proper HA configuration, the impact can be minimized, allowing jobs to continue with little interruption.

**What is the difference between a killed attempt and failed attempt?
Can you tell me a killed attempt example? what do you mean by a killed attempt**

**AM is killed by the user, will it start the job from beginning or resume the job where it started?**
AM killed by the user is not called an attempt and it will be resume where it left the job.

**Difference between a killed Application Master (AM) and a failed job** 

Killed Application Master (AM):
This is when the AM process is terminated, either by a user or the system.
The AM is responsible for managing the job's execution.
Killing the AM doesn't necessarily mean the job has failed.
**Failed Job:**
This occurs when the actual work of the job cannot be completed successfully.
It could be due to errors in the code, data issues, or resource problems.
**Attempt:**
An "attempt" is a try at running the job or the AM.
YARN allows multiple attempts to run a job if previous attempts fail.
Now, to address your specific question:
If the AM is killed by the user, what happens next depends on the configuration and the state of the job:
**Resume the job:**
In many cases, YARN will start a new AM attempt.
This new AM will try to recover the job's state and resume from where it left off.
This is especially true for jobs that support checkpointing or have built-in recovery mechanisms.
**Start from the beginning:**
If the job doesn't support recovery or if the configuration doesn't allow for multiple attempts, the job may start from the beginning.
This is more likely to happen if the AM is killed very early in the job's execution.
**Key factors that influence this behavior:**
``yarn.resourcemanager.am.max-attempts:`` This setting determines how many times YARN will try to restart the AM.
**Application's recovery capabilities:** Some applications (like Spark) have better recovery mechanisms than others.
**Job progress:** Jobs that have made significant progress are more likely to be resumed rather than restarted.
In AWS EMR, these behaviors are generally the same, but EMR provides additional tools and configurations to manage job recovery and AM restarts.
Remember, the exact behavior can vary based on the specific application (e.g., Spark, MapReduce) and how it's configured to handle failures and restarts.

**Common issue with EMR and how to deal with it**

1. Understand the IC jointStatusMAP for YARN
2. Disk full how will it happen
    1. local disk is full user data user cache
    2. yarn and HDFS uses the same disk 
    3. Long running application
3. What will happen if the disk is full to the yarn?
YARN will be unhealthy
4. if the node is full for 45 min then node will be terminated

5. Yarn Negative core scenarios
6. RM resrart core scenarios 
7. RM restarts take very long time
8. Do you think keeping the termination protection on will also prevent the termination of task node?
9.When can you expect the yarn negative core scenarios?


The YARN negative core scenario occurs when the Capacity Scheduler is configured to use only memory as a resource (ignoring CPU),
which can lead to an unusual situation where the number of virtual cores (vCores) appears negative in the YARN Resource Manager UI.

Here's how it happens:
Default configuration:
YARN's Capacity Scheduler by default uses a **DefaultResourceCalculator** that only considers memory, not CPU.
**Resource allocation:**
YARN allocates containers based on available memory, not considering vCores.
**Oversubscription:**
This can lead to more containers being allocated than there are physical cores available.
**Negative vCore display:**
The YARN UI calculates available vCores by subtracting used from total.
When more containers are running than physical cores, this subtraction results in a negative number.
**Example scenario:**
Node has 16 vCores and 122 GB memory
Many small containers (e.g., 1.5 GB each) are allocated
The number of containers exceeds 16, leading to negative vCore display
**Impact:**
This doesn't necessarily mean the system is overloaded
It indicates that CPU resources are not being considered in scheduling decisions
**Solution:**
Configure YARN to use DominantResourceCalculator instead, which considers both memory and CPU
This scenario highlights the importance of properly configuring resource management in YARN, especially in clusters with diverse workloads or resource-intensive applications.
Related

How does the Capacity Scheduler contribute to negative vCores
What are the implications of negative vCores on YARN performance
How can I prevent negative vCores from occurring in my YARN cluster
What role does memory allocation play in YARN's vCore management
How does the DefaultResourceCalculator affect vCore allocation

**RM restart is takes very long time?**
Because there are many applicaiton runing for long time
hence the state store is full 

**Resource Manager (RM) restart taking a long time due to a large state store from long-running applications.**
  the solution:
**State Store Issue:**
The RM state store keeps track of running applications and their states.
With many long-running applications, this store can grow large over time.

During restart, RM needs to reload all this information, which can be time-consuming.

**Clearing the State Store:**
There's no direct command to clear the state store while the cluster is running.
Clearing it improperly can lead to data loss or inconsistencies.
*Recommended Approach:*
Instead of clearing the state store, consider these options:
*a. Graceful Cleanup:*
Identify and complete or terminate long-running applications that are no longer needed.
This naturally reduces the state store size over time.
*b. Rolling Restart:*
Perform a rolling restart of the Resource Managers if you have a high-availability setup.
This minimizes downtime and allows for a smoother transition.
*c. Maintenance Window:*
Schedule a maintenance window for a full cluster restart.
Before restarting, ensure all applications are properly shut down.
*d. State Store Optimization:*
Review and optimize your state store configuration.
Consider using a more efficient state store implementation if available.
EMR-Specific Considerations:
For AWS EMR, you typically don't directly manage the state store.
Instead, focus on proper application management and cluster maintenance.
Last Resort (Use with Caution):
**If you absolutely must clear the state store:**
1.Stop all YARN services
2.Locate the state store directory (usually configured in yarn-site.xml)
3.Backup the current state store
4.Remove or rename the state store directory
5.Restart YARN services

*WARNING: This approach can lead to data loss and should only be used in extreme cases with full understanding of the consequences.
Remember, regularly managing and closing completed applications is the best way to keep the state store manageable. If restart times are consistently problematic, consider reviewing your application lifecycle management practices or consulting with AWS support for EMR-specific optimizations.*


```
yarn.nodemanager.disk-health-checker.interval-ms = 120000
```
The frequency (in seconds) that the disk health checker runs.

https://hadoop.apache.org/docs/r3.3.4/hadoop-project-dist/hadoop-common/CommandsManual.html#daemonlog
```
hadoop daemonlog -getlevel
hadoop daemonlog -setlevel
```
**Set the debug in Haddop**
```
$ bin/hadoop daemonlog -setlevel 127.0.0.1:9870 org.apache.hadoop.hdfs.server.namenode.NameNode DEBUG
$ bin/hadoop daemonlog -getlevel 127.0.0.1:9871 org.apache.hadoop.hdfs.server.namenode.NameNode DEBUG -protocol https
```

**What could be the reason for AMlineveless monitoir not interactiing with the Application master Service?**
**What is the lost node in YARN?**
[Check the cloudera link](https://community.cloudera.com/t5/Community-Articles/Deep-dive-into-YARN-Log-Aggregation-Deep-dive-into-managing/ta-p/331399)
```
yarn.nodemanager.log-aggregation.roll-monitoring-interval-seconds
to aggregate logs while the application is running
```
[Check the cloudera link](https://community.cloudera.com/t5/Community-Articles/Deep-dive-into-YARN-Log-Aggregation-Deep-dive-into-managing/ta-p/331399)



## Yarn Timeline Server

[Check the offical doc](https://hadoop.apache.org/docs/stable/hadoop-yarn/hadoop-yarn-site/TimelineServiceV2.html)

Customer has run EMR step “s-XXXXXXXXX on EMR cluster “j-123456789”. 
The application completed correctly but the logs are not available in Resource Manager Web UI.

Why would this happen? What can you recommend to the customer?
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Statement1",
      "Effect": "Deny",
      "Action": "s3:Get*",
      "Resource": "arn:aws:s3:::ishan-staging/yarnlogs/*"
    }
  ]
  ```
  log4j.logger.com.amazon.ws.emr.hadoop.fs.shaded.com.amazonaws.request=DEBUG on /etc/hadoop/conf/log4j.properties and restart nodemanager.
  
Launch a following cluster:
EMR 7.2.0
Primary: m5.xlarge x1  Core: m5.xlarge x1
Create two queues for Capacity scheduler. Each queue is allocated 50% of the cluster capacity.
Limit the maximum resources that a single user can consume to 50% of a queue capacity.

**Important Questions:**
1. How do you confirm the queues are configured correctly?
2. Submit the following application. What do you observe?
```
$ hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient-3.3.6-amzn-4-tests.jar sleep -Dmapred.job.queue.name=[queue 1] -Dyarn.app.mapreduce.am.resource.memory-mb=1024 -Dmapreduce.map.resource.memory-mb=1024 -m 10 -mt 600000
```
Try implementing preemption and check the RM logs..



## Commiters
### FileOutputCommiter-- 
1. Used for HDFS
2. use a temporary location for keep the interims files then moves the fle to final destination
 CON
1. Not suitable for Writing to S3 (does not support multipart upload and S3 does not support rename )
2. Does not handle failure gracefull and lead to inconsitent output
### S3A commiters
Use a staging area in S3 then commits files
spark.hadoop.fs.s3a.commiter.name in sparkConf basically you can use -- spark -config or spark conf 
Con
Does not work with ORC format

EMRFSS3 optimized commiter EMR version 5.19 and later
extentionof hadoop file system to S3 provides.

S3 client side and server side encryption and Amazon S3 Authorization
reducing the S3 ceventual consistency.

EMRFS direct write commiter 6.1.0
It directly writes to S3 can help avoid data locality issues network staffic increase the overall  job performace
### Sample spark job
```
 sudo systemctl stop hadoop-yarn-resourcemanager.service
 spark-submit --conf spark.yarn.maxAppAttempts=<number>
 spark-submit --executor-memory 1g --driver-memory 1g --class org.apache.spark.examples.SparkPi /usr/lib/spark/examples/jars/spark-examples.jar 100000
```