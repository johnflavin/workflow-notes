# 2018-03-07

Started keeping workflow notes here.

Today I am reviewing all my old bookmarks. I want to collect the links, read through them all, take notes. Also adding additional links I had found over time and collected elsewhere.

The leading contenders to be considered are CWL (which is a workflow language preference, but we would need to find an appropriate engine) and Activiti (which is an engine preference, but with a non-prefered workflow language).

## Languages / Engines

### CWL
http://www.commonwl.org/
Common Workflow Language

A YAML spec for writing workflows and tool descriptors, as well as a spec for the workflow engine to execute them. They provide a command-line reference implementation for the spec, but other implementations support the spec.

The project lead, Michael Crusoe, is aware of our project and its on-again-off-again flirtation with workflow engines. He has popped up on our wiki to offer his assistance if we want to integrate CWL.

This is a leading contender, though I don't know what engine we would use.

### WDL / Cromwell
* [WDL](https://software.broadinstitute.org/wdl/) - "workflow description language". I have no real thoughts. See below.
* [Cromwell](https://github.com/bw2/cromwell) - Engine to execute WDL workflows. Blurb from the WDL [Execute!](https://software.broadinstitute.org/wdl/documentation/execution) page:

> Cromwell is an open source (BSD 3-clause) execution engine written in Java that supports running WDL on three types of platform: local machine (e.g. your laptop), a local cluster/compute farm accessed via a job scheduler (e.g. GridEngine) or a cloud platform (e.g. Google Cloud or Amazon AWS).

Seems like Cromwell and WDL are separating, however; this Jan. 2, 2018 blog post says [Multiple workflow languages coming to Cromwell, starting with CWL](https://software.broadinstitute.org/wdl/blog?id=11109). "...in order to maximize their usefulness and satisfy the needs of a wide community, Cromwell and WDL need to be decoupled so that they can both evolve more freely."

### Activiti (link)
https://www.activiti.org/

This is a workflow engine library/API that we could embed into our java webapp. Workflows are defined in XML using [Business Procces Model and Notation (BPMN) 2.0](http://www.bpmn.org/).

This is a leading contender.

### Pinball
https://github.com/pinterest/pinball

"Pinball is a scalable workflow manager". Runs as a server, has a master node that controls available workflows and worker nodes that run jobs. Does not appear to have a remote API, just a command line.

Probably not appropriate to our needs.

### Luigi
https://github.com/spotify/luigi

> Luigi is a Python (2.7, 3.3, 3.4, 3.5, 3.6) package that helps you build complex pipelines of batch jobs. It handles dependency resolution, workflow management, visualization, handling failures, command line integration, and much more.

That sounds good.

> Everything in Luigi is in Python. Instead of XML configuration or similar external data files, the dependency graph is specified *within Python*.

That sounds not good. We can't have user-generated and -supplied workflow definitions if they have to send us python code into our Java app. Doesn't work.

## Papers

* Malcolm Atkinson, Sandra Gesing, Johan Montagnat, Ian Taylor, Scientific workflows: Past, present and future. *Future Generation Computer Systems* **75**, 216–227 (2017). https://doi.org/10.1016/j.future.2017.05.041
    Introductory article to a special issue of *Future Generation Computer Systems* about workflows. I haven't read it because apparently WashU doesn't have access to this journal.
* Tristan Glatard, *et al.*, Software architectures to integrate workflow engines in science gateways. *Future Generation Computer Systems* **75**, 239–255 (2017). https://doi.org/10.1016/j.future.2017.01.005
    The one article I have read from the special issue mentioned above. (It's open access.) Discusses a few different models of how front-end applications integrate with workflow engines, the pros and cons of each, etc.

# 2018-03-09

Reiterating the two viable options I found before:

* **Activiti**: Easy for us to code, as they have an engine we can deploy into our application however we like, and an API for us to code against. But they use a workflow format that no one in the scientific community uses, as far as I know. That means our users will have to put in the work to learn a new format (which is a big ask and puts the onus on us for training) or we have to build a UI that makes it easy to construct workflows. Either way, this is a big burden on the users. This makes me think the project would be unlikely to have a big impact.
* **CWL**: Good for the users. Many of them are already familiar with / using CWL workflows. The workflow format is fairly simple to write. But I haven't as yet identified a candidate engine / API we can drop in. Unless I find one, that means we would be on the hook for the huge amount of work it would take to implement and maintain a CWL engine.

My next task is to confirm or refute the hypothesis that there are no CWL engines we could drop in to XNAT. I will review each of the CWL engines on the CWL site.

After writing these summaries, I am coming to understand more of what we need. Some parts of this will serve one purpose, but may not serve another. We need...

* Front-end, which sits inside XNAT. This will allow users to manage their workflows, launch jobs, see job progress, etc. Think HCP's pipeline control panel. We will have to write everything in here.
* Middle. This could be inside XNAT, but maybe should be outside. I think this should be a server, which will sit in between XNAT on the front and the job scheduling / execution on the back. XNAT can interact through an API. This layer will store the user workflows, launch jobs, keep track of and report on job progress, possibly more. We could write this layer, but I would hope we don't have to. Even if we do write it we should use some readily available library like [Xenon](#xenon).
* Back. This is where the jobs get executed. The question I have is whether we pass the jobs from the front to the back as CWL, and use some native CWL tool on the back end to execute them as a whole, or whether the middle layer is taking care of running the CWL job but sends the individual steps of that job to the back end to execute "natively". I think the latter makes more sense. So this back end could be raw docker or docker swarm, but possibly with some scheduler in front. Or it could be kubernetes or something more exotic like that.

So now I feel like I need to re-review these tools with that split in mind. Some of them could function on any one or multiple of these three layers.

## CWL Tool review

| Software                                                       | Description                                                                                                                                    | Platform support                                                       | Short Notes                                                                                                                                                                                                                           |
|:---------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [cwltool](https://github.com/common-workflow-language/cwltool) | Reference implementation of CWL                                                                                                                | Linux, OS X, Windows, local execution only                             | Command-line only. Not appropriate to our needs.                                                                                                                                                                                      |
| [Arvados](https://arvados.org/)                                | Distributed computing platform for data analysis on massive data sets. [Using CWL on Arvados](http://doc.arvados.org/user/cwl/cwl-runner.html) | AWS, GCP, Azure, Slurm                                                 |                                                                                                                                                                                                                                       |
| [Toil](http://toil.ucsc-cgl.org/)                              | Toil is a workflow engine entirely written in Python.                                                                                          | AWS, Azure, GCP, Grid Engine, LSF, Mesos, OpenStack, Slurm, PBS/Torque | Vanilla Toil runs jobs written in python code. Additional command-line tool can run CWL. Not what we need.                                                                                                                            |
| [Rabix Bunny](rabix.io)                                        | An open-source, Java-based implementation of Common Workflow Language with support for multiple drafts/versions.                               | Linux, OS X, GA4GH TES (experimental)                                  |                                                                                                                                                                                                                                       |
| [Galaxy](https://galaxyproject.org/)                           | Web-based platform for data intensive biomedical research.                                                                                     |                                                                        | See [section below](#galaxy)                                                                                                                                                                                                          |
| [cwl-tes](https://github.com/common-workflow-language/cwl-tes) | CWL engine backended by the [GA4GH Task Execution API](https://github.com/ga4gh/task-execution-schemas)                                        | Local, GCP, AWS, HTCondor, Grid Engine, PBS/Torque, Slurm              |                                                                                                                                                                                                                                       |
| [CWL-Airflow](https://github.com/Barski-lab/cwl-airflow)       | Package to run CWL workflows in Apache-Airflow (supported by BioWardrobe Team, CCHMC)                                                          | Linux, OS X                                                            |                                                                                                                                                                                                                                       |
| [Cromwell](https://github.com/broadinstitute/cromwell)         | Cromwell workflow engine                                                                                                                       | local, HPC, Google, HtCondor                                           |                                                                                                                                                                                                                                       |
| [Xenon](http://nlesc.github.io/Xenon/)                         | Run CWL workflows using Xenon                                                                                                                  | any Xenon backend: local, ssh, SLURM, Torque, Grid Engine              | See [section below](#xenon)                                                                                                                                                                                                           |
| [Consonance](https://github.com/Consonance/consonance)         | orchestration tool for running SeqWare workflows and CWL tools                                                                                 | AWS, OpenStack, Azure                                                  | [README](https://github.com/Consonance/consonance): "Consonance is a cloud orchestration tool for running Docker-based tools and CWL/WDL workflows available at Dockstore on fleets of VMs running in clouds." Seems not appropriate. |
| [Apache Taverna](http://taverna.incubator.apache.org/)         | Domain-independent Workflow Management System                                                                                                  |                                                                        |                                                                                                                                                                                                                                       |
| [AWE](https://github.com/MG-RAST/AWE)                          | Workflow and resource management system for bioinformatics data analysis.                                                                      |                                                                        | See [section below](#awe)                                                                                                                                                                                                             |
| [yacle](https://github.com/otiai10/yacle)                      | Yet Another CWL Engine                                                                                                                         |                                                                        | Command-line tool. Not what we need.                                                                                                                                                                                                  |
| [REANA](https://reana.readthedocs.io/en/latest/index.html)     | RE usable ANAlyses                                                                                                                             | Kubernetes, CERN OpenStack (OpenStack Magnum)                          |                                                                                                                                                                                                                                       |


Table copied from [commonwl.org](commonwl.org).


### Galaxy

This is a web platform, which allows you to import genomic data and build custom processing workflows. It looks... well, it looks like it has good functionality. But I think the workflows are too specific to genomics/bioinformatics. I don't know if we can get our own workflows in there.

### Xenon

This is pretty interesting. From the README on the github ([https://github.com/NLeSC/Xenon](https://github.com/NLeSC/Xenon)):

> Xenon is an abstraction layer that sits between your application and the remote resource it uses. Xenon is written in Java...

This is a library that abstracts file handling and job submission to various schedulers.

They are listed on the CWL page, but I do not see anything in their docs about CWL. All their examples of submitting jobs use java code to define the job. So we will have to investigate some more. But this looks promising!

Still, this is a library that works more on the submission side. On the execution side, we would need to use some engine (like `cwltool` perhaps) to actually run the jobs. And then we would need to figure out if we get monitoring/control capabilities and such.

### Awe

I'm not entirely sure what this is. It looks like this is a particular server instance that manages workflows, in concert with another server called Shock that stores genomic data. So... I think we can't use this.

# 2018-03-12

Things you need a workflow system to do.

1. Provide a way to define workflows.
2. Launch jobs using the workflows.

A *task* is a unit of stuff to be run. This could be a script, an executable, a docker container, whatever. This is the unit of action in a workflow.
A *workflow* is a directed graph of tasks.

The Pipeline Control Panel (PCP) is not a workflow system. It is a task system. You define all the tasks that can be run on the data within the PCP (where they are referred to as "pipelines"). There are interfaces to launch tasks, to see which tasks can be run, have been run, and failed to run. But, crucially, there is no workflow definition, no larger flow between tasks. There are event triggers, which are used to launch tasks under various circumstances. But at best those define individual edges of particular graphs; nowhere is the whole graph, the whole workflow, defined.

As such, the PCP does not meet the needs for a workflow management system. It it focused on tasks, not the workflow made of multiple tasks.

It is not even a general-purpose tool for task management and execution. The burden for defining a new task is fairly high. The approach the PCP takes to task management and execution is very tightly coupled into the tasks/workflows of the HCP/CCF, which does not change very often and does not need to service user-created workflows.

## What does a workflow engine do?

The fundamental thing we need to do is this:

1. On one end, the front end, a user or event initiates a request to launch a workflow.
2. On the other end, the back end, tasks are executed on compute nodes.

In the middle something needs to turn the workflow into its constituent tasks and execute them. This is the role of the *workflow engine*. It reads the workflow definition and executes all the tasks contained within.

There is a lot of variation on how that middle bit can happen. Some questions you could ask about the architecture of a workflow system:

* Where does the workflow engine live? Front end / back end / middle?
* Where does the workflow definition live? Front end / back end / middle?
* How do job schedulers fit in?
* Are all the tasks of one entire workflow job run on one compute node, or are workflow job tasks sent to compute nodes indivudually and a higher layer combines their results and keeps track of workflow state?
* How does the front end receive updated information about workflow state? Task state?

For an example, let's look at what we have currently: the XNAT Pipeline engine. Its workflow definitions are the pipeline XML files. "Pipelines" are made of "steps", each of which executes a "resource". A "step" + "resource" in pipeline terms are what I would call a task. The "resource" is itself defined in an XML file which is separate from the pipeline XML, and is essentially a re-usable part of the task definition. With that background, let's answer the above questions.

* The workflow engine, what we would call the "pipeline engine" or the `XnatPipelineLauncher`, lives on all the back-end compute nodes. It also usually lives on the front-end server along with XNAT, but it is not used there in multi-node setups.
* The workflow definition lives in the pipeline XML file. This file must be present on the front-end XNAT node as well as all back-end compute nodes. (The XML file only needs to be present on the XNAT node long enough for XNAT to read the file at the time it is defined in the site, then could be removed after. It is never used there again.)
* A job scheduler can fit between the XNAT front end and the compute node back end. We provide our pipeline users with the `schedule` script as an extension point to customize job scheduling. The `PipelineJobSubmitter` is used internally in our `schedule` scripts for job submission to SGE.
* The entire workflow is sent as one job from the front end to a single compute node.
* The workflow engine sends workflow state back to the front end. As the tasks are not broken out into separate processes, no task-level state is available other than that in the workflow state itself.

Now let's answer all the same questions, but this time about the PCP.

* The workflow engine doesn't really exist as a thing. Part of the workflow is controlled by the Event infrastructure within XNAT, and part is controlled by a user clicking in the PCP interface to launch tasks.
* Similar to above answer. The workflow is defined implicitly by the event connections and also in each user's head.
* I don't know this answer. I think that the PBS system on the CHPC is used for scheduling, but perhaps also the SGE on the NRG infrastructure depending on the job.
* Tasks are the unit of work in the PCP; individual task jobs are scheduled.
* Task state is stored in XNAT, and updated when certain actions are performed (user launches a task job) or when conditions are met (periodically-run validator detects proper output files are present).
