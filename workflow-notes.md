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

| Software | Description | Platform support | Short Notes |
|:--|:--|:--|:--|
| [cwltool](https://github.com/common-workflow-language/cwltool) | Reference implementation of CWL | Linux, OS X, Windows, local execution only | Command-line only. Not appropriate to our needs. |
| [Arvados](https://arvados.org/) |	Distributed computing platform for data analysis on massive data sets. [Using CWL on Arvados](http://doc.arvados.org/user/cwl/cwl-runner.html) | AWS, GCP, Azure, Slurm | See [section below](#arvados) |
| [Toil](http://toil.ucsc-cgl.org/) | Toil is a workflow engine entirely written in Python. | AWS, Azure, GCP, Grid Engine, LSF, Mesos, OpenStack, Slurm, PBS/Torque | Vanilla Toil runs jobs written in python code. Additional command-line tool can run CWL. Not what we need. |
| [Rabix Bunny](rabix.io) | An open-source, Java-based implementation of Common Workflow Language with support for multiple drafts/versions. | Linux, OS X, GA4GH TES (experimental) | See [section below](#rabix-bunny) |
| [Galaxy](https://galaxyproject.org/) | Web-based platform for data intensive biomedical research. | | See [section below](#galaxy) |
| [cwl-tes](https://github.com/common-workflow-language/cwl-tes) | CWL engine backended by the [GA4GH Task Execution API](https://github.com/ga4gh/task-execution-schemas) | Local, GCP, AWS, HTCondor, Grid Engine, PBS/Torque, Slurm |  |
| [CWL-Airflow](https://github.com/Barski-lab/cwl-airflow) | Package to run CWL workflows in Apache-Airflow (supported by BioWardrobe Team, CCHMC)| Linux, OS X |  |
| [Cromwell](https://github.com/broadinstitute/cromwell) | Cromwell workflow engine | local, HPC, Google, HtCondor |  |
| [Xenon](http://nlesc.github.io/Xenon/) | Run CWL workflows using Xenon | any Xenon backend: local, ssh, SLURM, Torque, Grid Engine | See [section below](#xenon) |
| [Consonance](https://github.com/Consonance/consonance) | orchestration tool for running SeqWare workflows and CWL tools | AWS, OpenStack, Azure | [README](https://github.com/Consonance/consonance): "Consonance is a cloud orchestration tool for running Docker-based tools and CWL/WDL workflows available at Dockstore on fleets of VMs running in clouds." Seems not appropriate. |
| [Apache Taverna](http://taverna.incubator.apache.org/) | Domain-independent Workflow Management System | | See [section below](#taverna) |
| [AWE](https://github.com/MG-RAST/AWE) | Workflow and resource management system for bioinformatics data analysis. | | See [section below](#awe) |
| [yacle](https://github.com/otiai10/yacle) | Yet Another CWL Engine | | Command-line tool. Not what we need. |
| [REANA](https://reana.readthedocs.io/en/latest/index.html) | RE usable ANAlyses | Kubernetes, CERN OpenStack (OpenStack Magnum) |  |

Table copied from [commonwl.org](commonwl.org).

### Arvados


### Rabix Bunny


### Galaxy

This is a web platform, which allows you to import genomic data and build custom processing workflows. It looks... well, it looks like it has good functionality. But I think the workflows are too specific to genomics/bioinformatics. I don't know if we can get our own workflows in there.

### Xenon

This is pretty interesting. From the README on the github ([https://github.com/NLeSC/Xenon](https://github.com/NLeSC/Xenon)):

> Xenon is an abstraction layer that sits between your application and the remote resource it uses. Xenon is written in Java...

This is a library that abstracts file handling and job submission to various schedulers.

They are listed on the CWL page, but I do not see anything in their docs about CWL. All their examples of submitting jobs use java code to define the job. So we will have to investigate some more. But this looks promising!

Still, this is a library that works more on the submission side. On the execution side, we would need to use some engine (like `cwltool` perhaps) to actually run the jobs. And then we would need to figure out if we get monitoring/control capabilities and such.

### Taverna


### Awe

I'm not entirely sure what this is. It looks like this is a particular server instance that manages workflows, in concert with another server called Shock that stores genomic data. So... I think we can't use this.

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
