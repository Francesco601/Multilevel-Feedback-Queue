# Multilevel-Feedback-Queue

A Multilevel Feeback Queue (MLFQ) CPU scheduling algoritm is one of the most well-known approaches to CPU scheduling in operating systems. It was first described by 
Corbato et al in 1962 in a system known as the Compatible Time-Sharing System and was later incorporated into Multics, the forerunner of Unix. The scheduler has been
subsequently developed and refined thoughout the years into the implementations that you might encounter in some modern systems.

The fundmamental problem that MLFQ tries to address is two-fold. First, it needs to optimize <em>turnaround time</em>, which is usually done most successsfuly by 
running shorter jobs first. Unfortunaately, the OS doesn't generally know how long a job or process will run for, exactly the knowledge that algorithms like SJF
(or STCF) require. Second of all, MLFQ needs to make a system feel responsive to interactive users (i.e. users sitting and staring at the screen, waiting for a 
process to finish) and thus minimize <em>response time</em>; unfortunately algorithms like Round Robin reduce response time but are horrible for turnaround time. So 
the problem is: given that we in general do not know anything about a process, how can we build a CPU scheduler to acheive these goals? How can the scheduler learn,
as the system is running, the characteristics of the jobs it is running, and theefore make better scheduling decisions? In short, how can we design a scheduler that
both minimizes response time (i.e. T(first run) - T(arrival)) for interactive jobs while also minimizing turnaround time (T(completion)-T(arrival)) without any
<em>apriori</em> knowledge of job lengths?

## Basic Rules of MLFQ 
The MLFQ has a number of distinct **queues**, each assigned a different **priority level**. At any given timne, a job that is ready to run is on a single queue. 
MLFQ uses priorities to decide which job should run at a given time. A job with a higher priority (i.e, a job on a higher queue) is chosen to run. Of course, more
than one job may be on a given queue, and thus have the  **same** priority. In this case, we will just use round-robin scheduling among those jobs. The key to MLFQ
scheduling lies in how the scheduler establishes priorities. Rather than giving fixed priority to each job, MLFQ ***varies*** the priority of a job based on its
observed behavior. If, for example, a job repeatedly relinquishes the CPU while waiting for input from the keyboard, MLFQ will keep its priority high, as this is how
an interactive process might behave. If, instead, a job uses the CPU intensively for long periods of time, MLFQ will reduce its priority. In this way, the algorithm
will try to learn about processes as they run, and thus use the ***history*** of the job to predict its future behavior.
   The two basic rules of the MLFQ are:
   - If Priority(A) > Priority(B), A runs and B doesn't.
   - If Priority(A) = Priority(B), A and B run in RR (round robin)

What we need to understand is how job priority changes over time. Here is a first attempt at a priority-adjustment algorithm:
  - When a job enters the system, it is placed at the highest priority level (the topmmost queue).
  - If a job uses up an entire time slice or quantum while running, itspriority is ***reduced*** (i.e, it moves down one queue).
  - If a job gives up the CPU before the time slice is up, it stays at the same priority level.

Let's think about a somewhat complicated example to see how MLFQ tries to approximate SJF. In this example, there are two jobs: A, which is a long-running CPU intensive
job, and B, which is a short-running interactive job. Assume A has been running for some time, and then B arrives. What will happen? A is running along in the lowest
prioirty queue (as would any long-running CPU-intensive jobs); B arrives, say, at time T=100, and thus is inserted into the highest queue. Since its run time is short
(only say 20ms), B completes before reaching the bottom queue, in two time slices; then A resumes running (at low priority).
From this example, you can hopefully understand one of the major goals of the algorithm: since it doesn't ***know*** whether a job will be a short or long running 
job, it first ***assumes*** it will be a short job and gives it high priority. If it is actually a short job, it will run quickly and complete; if is is not a short job,
it will slowly move down the queues, and thus prove itself  to be a long-running more batch-like process. In this manner, MLFQ approximates Shortest Job First.

We can thus have a basic MLFQ algorithm. It seems to do a fairly good job, sharing the CPU fairly between long-running jobs, and letting short or I/O-intensive 
interactive jobs run quickly. Unfortunately, this approach has serious defects. First, there is the problem of **starvation**: if there are "too many" 
interactive jobs in the system, they will combine to consume ***all*** CPU time, and thus long-running jobs will never receieve any CPU time (they starve). We
would like to make some progress on these jobs even in this scenario. Another problem is that a smart user could rewrite their program to **game the scheduler**. 
Gaming the scheduler generally refers to the idea of doing something sneaky to trick the scheduler into giving you more than your fair share of the resource. The
algorithm so far described is vulnerable to the following attack: before the time slice (quantum) is over, issue an I/O operation to some file you don't cate about
and thus relinquish the CPU. Doing so allows you to remain in the same queue and thus gain a higher percentage of CPU time. When done right (for example, by running
for 99% of a time slice before giving up the CPU), a job could nearly monoplize the CPU.And, finally, a program may change its behavior over time: what was CPU-bound may transition to a phase of interactivity. With our current approach, such a job would be out of luck and not be treated like the other interactive jobs in the system,

We can change the rules to avoid the problem of starvation. What can be done in order to assure that CPU-bound jobs
will make some progress (even if it is not much?). The simple idea is to periodically boost the priority of all the
jobs in the system. There are many ways to acheive this but the simplest is probably to throw them all into the topmost
queue.
  - After some period of time S. move all the jobs in the system to the topmost queue.

This new rule solves two problems at once. First, processes are assured not the starve: by sitting in the top queue, a
job will share the CPU with other high-prioirity jobs in a round-robin fashion, and eventually receive service. Second.
if a CPU-bounded job has become interactive, the scheduler treats it properly once it has received the priority boost.

Now there is one more problem to solve: how to prvent gaming of the scheduler. The real culprit here are the rules which let a job retain its priority by giving up the CPU before the time slice expires. What can be done? The solution
here is to perform better **accounting** of CPU time at each level of the MLFQ. Instead of forgetting how much of a 
time slice a process used at a given level, the scheduler should keep track; once a process has used its allotment, it
is demoted to the next priority queue. Whether it uses the time slice in one long burst or many small ones does not
matter. We can adopt the new rule:c
   - Once a job uses up its time allottment at a given level (regardless of how many times is has given up the CPU), its priority is reduced (i.e it moved down one queue).

 Whereas the multilevel queue algorithm keeps processes permanently assigned to their initial queue assignments, the
 Mutlilevel Feedback Queue shifts processes between queues. The shift is dependent on the CPU bursts of prior 
 time-slices
    - If a process uses too much CPU time, it will be moved to a lower-priority queue.
    - If a process is I/O bound or an interactive process, it will be moved to a higher-priority queue.
    - If a process is waiting too long in a low-priority queue and starving, it will be aged to a higher-priority queue.

 <H2>Algorithm</H2>
    
    Multiple FIFO queues are used and the operation is as follows:
     1) a new process is inserted at the tail of the top-level FIFO queue.
     2) at some stage the process reaches the head of the queue and is assigned the CPU.
     3) if the process is completed within the time-slice of the given queue, it leaves the system.
     4) if the process voluntarily relinquishes control of the CPU, it leaves the queueing network, and when the 
      process becomes ready again it is inserted at the tail of the same queue which is gave up earlier-
     5) If the process uses all the quantum time, it is pre-empted and inserted at the end of the next lower
     level queue. This next lower level queue will have a time quantum which is more than that of the previous
     higher-level queue.
     6) This scheme will continue until the process completes or it reaches the base level queue.
      - At the base level queue the processes circulate in round-robin fashion until they complete and
      leave the system. Processes in the base level queue can also be scheduled on a FCFS basis.
      - Optionally, if a process blocks for I/O, it is "promoted" one level, and placed at the end of the next
      higher-level queue. This allows I/O bound processes to be favored by the scheduler and allows processes to
      "escape" the base level queue.
 
#### Sources
<a href="https://os.ecci.ucr.ac.cr/slides/Abraham-Silberschatz-Operating-System-Concepts-10th-2018.pdf"> Operating System Concepts </a> Silberschatz, Galvin and Gagne
<a href="https://drdineshsharma.com/Operating%20Systems.pdf"> Three Easy Pieces </a> Renzi Arpaci-Dusseaa and Andrea Arpaci-Dusseau
<a href="https://en.wikipedia.org/wiki/Multilevel_feedback_queue"> Multilevel Feedback Queue </a> Wikipedia
     
 
 



