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

### Activiti
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
