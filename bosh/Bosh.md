# Bosh

## What is bosh

Target at cloud software

- release engineering
- deployment
- lifecycle management

## What problem does bosh solve

Do Versioning, packaging, and deploying software reproducibly as a whole. 

Provide a method to maintaining consistency between multiple environments.

### Compare with k8s+docker

**To be done**

### Four principles of mordern Release Engineering

- **Identifiability**

  > Being able to identify all of the source, tools, environment, and other components that make up a particular release.

  BOSH has a concept of a software release which packages up all related source code, binary assets, configuration etc. This allows users to easily track contents of a particular release. In addition to releases BOSH provides a way to capture all Operating System dependencies as one image.

- **Reproducibility**

  > The ability to integrate source, third party components, data, and deployment externals of a software system in order to guarantee operational stability.

  与我理解的reproducibility有些不同：

  > Reproducibility:This is about the ability to generate an exact copy of (i.e. reproduce) a release

  Release engineering里的Reproducibility倾向于便捷地整合各种资源的能力。

  BOSH tool chain provides a centralized server that manages software releases, Operating System images, persistent data, and system configuration. It provides a clear and simple way of operating a deployed system.

- **Consistency**

  > The mission to provide a stable framework for development, deployment, audit, and accountability for software components.

  这里的一致性与常规定义里的一致性也有些不同，Release engineering里的Reproducibility倾向于能够为软件开发的workflow提供一个好用的框架。

  BOSH software releases workflows are used throughout the development of the software and when the system needs to be deployed. BOSH centralized server allows users to see and track changes made to the deployed system.

- **Agility**

  > The ongoing research into what are the repercussions of modern software engineering practices on the productivity in the software cycle, i.e. continuous integration.

  其实就是可持续集成的能力

  BOSH tool chain integrates well with current best practices of software engineering (including Continuous Delivery) by providing ways to easily create software releases in an automated way and to update complex deployed systems with simple commands

## Bosh Concepts

### Stemcell

A typical stemcell contains a bare minimum OS skeleton with a few common utilities pre-installed, a BOSH Agent, and a few configuration files to securely configure the OS by default.

其实就是一个操作系统镜像，预装了一些便于bosh进行管理的工具（bosh agent）。除此之外，stemcell不包含任何其他信息。既没有该镜像实例化之后应该后续安装的软件信息，也没有敏感的credential，这使得stemcell能够方便地分发出去。

Cloud foundry 官方负责维护适用于各大IaaS平台的Stemcell.

By introducing the concept of a stemcell, the following problems have been solved:

- Capturing a base Operating System image
- Versioning changes to the Operating System image
- Reusing base Operating System images across VMs of different types
- Reusing base Operating System images across different IaaS

### Release

A release is a versioned collection of configuration properties, configuration templates, start up scripts, source code, binary artifacts, and anything else required to build and deploy software in a **reproducible** way.

就是通常意义下的Release，不过这里的**reproducible** 似乎又变成了它本来的意思。

Release位于Stemcell的上层，包含了应该安装的软件的信息。

By allowing layering of stemcells and releases, BOSH is able to solve problems such as “how does one make sure that the compiled version of the software is reliably available throughout the deploy”, or “how to version and roll out updated software to the whole cluster, VM-by-VM”, that other orchestration software is not able to solve.

By introducing the concept of a release, the following concerns are addressed:

- Capturing all needed configuration options and scripts for deployment of the software
- Recording and keeping track of all dependencies for the software
- Versioning and keeping track of software releases
- Creating releases that can be IaaS agnostic
- Creating releases that are self-contained and do not require internet access for deployment

### Deployment

A deployment is a collection of VMs, built from a [stemcell](http://bosh.io/docs/stemcell.html), that has been populated with specific [releases](http://bosh.io/docs/release.html) and disks that keep persistent data. These resources are created in the IaaS based on a deployment manifest and managed by the [Director](http://bosh.io/docs/terminology.html#director), a centralized management server.

即运行` bosh deploy`后创建出的集群。用户可通过manifest来管理集群，该manifest 是 IaaS independent的，故修改manifest的工作量非常小。（针对各IaaS平台的具体信息存放于`cloud-config`中