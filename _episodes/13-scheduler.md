---
title: "Scheduling jobs"
teaching: 45
exercises: 30
questions:
- "What is a scheduler and why are they used?"
- "How do I launch a program to run on any one node in the cluster?"
- "How do I capture the output of a program that is run on a node in the cluster?"
objectives:
- "Run a simple Hello World style program on the cluster."
- "Submit a simple Hello World style script to the cluster."
- "Use the batch system command line tools to monitor the execution of your job."
- "Inspect the output and error files of your jobs."
keypoints:
- "The scheduler handles how compute resources are shared between users."
- "Everything you do should be run through the scheduler."
- "A job is just a shell script."
- "If in doubt, request more resources than you will need."
---

## Job scheduler
An HPC system might have thousands of nodes and thousands of users. How do we decide who gets what
and when? How do we ensure that a task is run with the resources it needs? This job is handled by a
special piece of software called the scheduler. On an HPC system, the scheduler manages which jobs
run where and when.

The following illustration compares these tasks of a job scheduler to a waiter in a restaurant.
If you can relate to an instance where you had to wait for a while in a queue to get in to a 
popular restaurant, then you may now understand why sometimes your job do not start instantly
as in your laptop.

{% include figure.html max-width="75%" file="/fig/restaurant_queue_manager.svg"
alt="Compare a job scheduler to a waiter in a restaurant" caption="" %}


The scheduler used in this lesson is {{ site.sched_name }}. Although {{ site.sched_name }} is not used everywhere,
running jobs is quite similar regardless of what software is being used. The exact syntax might change, but the
concepts remain the same.  You might come across HPC machines running Slurm or PBS elsewhere, the basic ideas are the same.

## Running a batch job

The most basic use of the scheduler is to run a command non-interactively. Any command (or series 
of commands) that you want to run on the cluster is called a *job*, and the process of using a
scheduler to run the job is called *batch job submission*.

In this case, the job we want to run is just a shell script. Let's create a demo shell script to 
run as a test.

> ## Creating our test job
> 
> Using your favorite text editor (nano or gedit), create the following script and run it. Does it run on the
> cluster or just our login node?
>
>```
> #!/bin/bash
> #$ -cwd -V
> #$ -l h_rt=00:05:00
>
> echo 'This script is running on:'
> hostname
> sleep 120
> ```
> {: .bash}
{: .challenge}

If you completed the previous challenge successfully, you probably realise that there is a
distinction between running the job through the scheduler and just "running it". To submit this job
to the scheduler, we use the `{{ site.sched_submit }}` command.

```
qsub example-job.sh
```
{: .bash}



And that's all we need to do to submit a job. Our work is done -- now the scheduler takes over and
tries to run the job for us. While the job is waiting to run, it goes into a list of jobs called 
the *queue*. To check on our job's status, we check the queue using the command
`qstat`.

```
qstat
```
{: .bash}


We can see all the details of our job, most importantly that it is in the "r" or "RUNNING" state.
Sometimes our jobs might need to wait in a queue ("qw") or have an error. The best way to check
our job's status is with `qstat` or `qstat -u "*"` if you want to see the status of all the users. 
Of course, running `qstat` repeatedly to check on things can be
a little tiresome. 

To see a real-time view of our jobs, we can use the `watch` command. `watch`
reruns a given command at 2-second intervals. This is too frequent, and will likely upset your system
administrator. You can change the interval to a more reasonable value, for example 60 seconds, with the
`-n 60` parameter. Let's try using it to monitor another job.

```
{{ site.host_prompt }} qsub example-job.sh
{{ site.host_prompt }} watch -n 60 qstat
```
{: .bash}

You should see an auto-updating display of your job's status. When it finishes, it will disappear
from the queue. Press `Ctrl-C` when you want to stop the `watch` command.

## Customising a job

The job we just ran used all of the scheduler's default options. In a real-world scenario, that's
probably not what we want. The default options represent a reasonable minimum. Chances are, we will
need more cores, more memory, more time, among other special considerations. To get access to these
resources we must customize our job script.

Comments in UNIX (denoted by `#`) are typically ignored. But there are exceptions. For instance the
special `#!` comment at the beginning of scripts specifies what program should be used to run it
(typically `/bin/bash`). Schedulers like {{ site.sched_name }} also have a special comment
used to denote special scheduler-specific options. Though these comments differ from scheduler to
scheduler, {{ site.sched_name }}'s special comment is `{{ site.sched_comment }}`.
Anything following the `{{ site.sched_comment }}` comment is interpreted as an
instruction to the scheduler.

Let's illustrate this by example. By default, a job's name is the name of the script, but the
`{{ site.sched_flag_name }}` option can be used to change the name of a job.

Submit the following job (`{{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh`):

```
#!/bin/bash
{{ site.sched_comment }} {{ site.sched_flag_name }} new_name

echo 'This script is running on:'
hostname
sleep 120
```

```
{{ site.host_prompt }} {{ site.sched_status }} {{ site.sched_flag_user }}
```
{: .bash}

Fantastic, we've successfully changed the name of our job!

### Resource requests

But what about more important changes, such as the number of cores and memory for our jobs? One 
thing that is absolutely critical when working on an HPC system is specifying the resources 
required to run a job. This allows the scheduler to find the right time and place to schedule our 
job. If you do not specify requirements (such as the amount of time you need), you will likely be
stuck with your site's default resources, which is probably not what we want.


Note that just *requesting* these resources does not make your job run faster! We'll talk more 
about how to make sure that you're using resources effectively in a later episode of this lesson.

> ## Submitting resource requests
>
> Submit a job that will use 1 full node and 5 minutes of walltime.
{: .challenge}


Resource requests are typically binding. If you exceed them, your job will be killed. Let's use
walltime as an example. We will request 30 seconds of walltime, and attempt to run a job for two
minutes.


Submit the job and wait for it to finish. Once it is has finished, check the log file.

```
{{ site.host_prompt }} {{ site.sched_submit }} {{ site.sched_submit_options }} example-job.sh
{{ site.host_prompt }} watch -n 60 {{ site.sched_status }} {{ site.sched_flag_user }}

```
{: .bash}


Our job was killed for exceeding the amount of resources it requested. Although this appears harsh,
this is actually a feature. Strict adherence to resource requests allows the scheduler to find the
best possible place for your jobs. Even more importantly, it ensures that another user cannot use
more resources than they've been given. If another user messes up and accidentally attempts to use
all of the cores or memory on a node, {{ site.sched_name }} will either restrain their job
to the requested resources or kill the job outright. Other jobs on the node will be unaffected.
This means that one user cannot  mess up the experience of others, the only jobs affected by a
mistake in scheduling will be their
own.

## Cancelling a job

Sometimes we'll make a mistake and need to cancel a job. This can be done with the `{{ site.sched_del }}`
command. Let's submit a job and then cancel it using its job number (remember to change the 
walltime so that it runs long enough for you to cancel it before it is killed!).

```
qstat

```
{: .bash}


Now cancel the job with it's job number. Absence of any job info indicates that the job has been
successfully cancelled.

```
qdel <jobid>

```
{: .bash}
... Note that it might take a minute for the job to disappear from the queue ...
qstat
```
{: .bash}

## Other types of jobs

Up to this point, we've focused on running jobs in batch mode. {{ site.sched_name }}
also provides the ability to start an interactive session.

There are very frequently tasks that need to be done interactively. Creating an entire job
script might be overkill, but the amount of resources required is too much for a login node to
handle. A good example of this might be building a genome index for alignment with a tool like
[HISAT2](https://ccb.jhu.edu/software/hisat2/index.shtml). Fortunately, we can run these types of
tasks as a one-off with `qsh`.



{% include links.md %}
