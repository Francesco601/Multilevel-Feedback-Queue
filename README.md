# Multilevel-Feedback-Queue

A Multilevel Feeback Queue (MLFQ) CPU scheduling algoritm is one of the most well-known approaches to CPU scheduling in operating systems. It was first described by 
Corbato et al in 1962 in a system knownn as the Compatible Time-Sharing System and was later incorporated into Multics, the forerunner of Unix. The schedular has been
subsequently developed and refined thoughout the years into the implementations that you might encounter in some modern systems.

The fundmamental problem that MLFQ tries to address is two-fold. First, it needs to optimize ***turnaround time***, which is usually done most successsfuly by 
running shorter jobs first. Unfortunaately, the OS doesn't generally know how long a job or process will run for, exactly the knowledge that algorithms like SJF
(or STCF) require. Second of all, MLFQ needs to make a system feeel respomsive to interactive users (i..e users sitting and staring at the screen, waiting for a 
process to finish) and thus minimize ***response time***; unfortunately algorithms like Round Robin reduce response time but are horrible for turnaround time. So 
the problem is: given that we in general do not know anything about a process, how can we build a CPU scheduler to acheive these goals? How can the scheduler learn.
as the system is running, the characteristics of the jobs it is running, and theefore make better scheduling decisions? In short, how can we design a scheduler that
both minimizes response time (i.e T(first run) - T(arrival)) for interactive jobs while also minimizing turnaround time (T(completion)-T(arrival)) without any
***apriori*** knowledge of job lengths?

<H1> Basic Rules of MLFQ </H1>
The MLFQ has a number of distinct **queues**, each assigned a different **priority level**. At any given timne, a job that is ready to run is on a single queue. 
MLFQ used priorities to decide which job should run at a given time. A job with a higher priority (i.e, a job on a higher queue) is chosen to run. Of course, more
than one job may be on a given queue, and this have the  **same*** priority. In this case, we will just use round-robin scheduling among those jobs. The key to MLFQ
scheduling lies in how the scheduler establishes priorities. Rather than giving  fixed priority  to each job, MLFQ ***varies*** the priority of a job based on its
observed behavior. If, for example, a job repeatedly relinquishes the CPU while waiting for input from the keyboard, MLFQ will keep its priority high, as this is how
an interactive process might behave. If, instead, a job uses the CPU intensively for long periods of time, MLFQ will reduce its priority. In this way, the algorithm
will try to lean about processes as they run, and thus use the ****history*** of the job to predict its future behavior.
   The two basic rules of the for MLFQ are:
   - If Priority(A) > Priority(B), A runs and B doesn't.
   - If Priority(A) = Priority(B), A and B run in RR (round robin)

What we need to understand is how job priority changes over time. Here is a first attempt at a priority-adjustment algorithm:
  - When a job enters the system, it is placed at the highest priority level (the topmmost queue).
  - If a job uses up an entire time slice or quantum while running, itspriority is ***reduced*** (i.e, it moves down one queue).
  - If a job gives up the CPU before the time slice is up, it stays at the same priority level.

Let's think about a somewhat co mplicated example to see how MLFQ tries to approximate SJF. In this example, there are two jobs: A, which is a long-running CPU intensive
job, and B, which is a short-running interactive job. Assume A has been running for some time, and then B arrives. What will happen? A is running along in the lowest
prioirty queue (as would any long-running CPU-intensive jobs); B arrives, say, at time T=100, and thus is inserted into the highest queue. Since its run time is short
(only say 20ms), B completes before reaching the bottom queue, in two time slices; then A resumes running (at low priority).
From this example, you can hopefully understand one of the major goals of the algorithm: since it doesn't ***know*** whether a job will be a short or long running 
job, it first ***assumes*** it will be a short job and gives it high priority. If it is actually a short job, it will run quickly and complete; if is is not a short job,
it will slowly move down the queues, and thus prove itself  to be a long-running more batch-like process. In this manner, MLFQ approximates Shortest Job First.

We can thus have a basic MLFQ algorithm. It seems to do a fairly good job, sharing the CPU fairly between long-running jobs, and letting short or I/O-intensive 
interactive jobs run quickly. Unfortunately, this this approach has serious defects. First, there is the problem of **starvation**: if there are "too many" 
interactive jobs in the system, they will combine to consume ***all*** CPU time, and thus long-running jobs will never receieve any CPU time (they starve). We
would like to make some progress on these jobs even in this scenario. Another problem is that a smart user could rewrite their program to **game the scheduler**. 
Gaming the scheduler generally refers to the idea of doing something sneaky to trick the scheduler into giving you more than your fair share of the resource. The
algorithm so far described is vulnerable to the following attack: before the time slice (quantum) is over, issue an I/O operation to some file you don't cate about
and thus relinquish the CPU. Doing so allows you to remain in the same queue and thus gain a higher percentage of CPU time. When done right (for example, by running
for 99% of a time slice before giving up the CPU), a job could nearly monoplize the CPU.And, finally, a program may change its behavior over time


