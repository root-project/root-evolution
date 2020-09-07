# root-infrastructure

* Proposal: [RE-0003](0003-root-infrastructure.md)
* Authors: [Vassil Vassilev](https://github.com/vgvassilev)
* Review Manager: TBD
* Status: **Awaiting review**

Motivation
===

Last several decades show stable trend in costs of computers and developers time. While machine time is cheap, developers' time is expensive.

This proposal intends to describe a set of common problems of the ROOT's testing, integration and deployment from code developers' perspective. It lays out a possible solution to address the current issues with respect to reproducibility of build issues, stability of the nodes and better use of overall resources.


Workflows
===

ROOT pull request builds and nightly builds.


Pull requests
---

Each pull request is built only in a dedicated folder which is the same for all parallel builds. While this saves disk space, it makes it impossible to debug a pull request test failure.


Nightly builds
---

  No procedure for cleaning up the build nodes when they are short in space.


Possible implementation
===

Deployment logic is tightly coupled to the software development and even github/gitlab nowadays. Jenkins allows a single jenkinsfile to be part of the repository similarly to travis and appveyor. This file can drive the overall build and deployment process [1], [2]. The major benefit will be that each modification of those files will undergo the same procedure -- will be visible only in the relevant pull request and will get merged only once the results are admissible.

This logic should use the truly parallel jenkins pipelines. It should allow each pull request to be built in a separate subfolder and should allow a configurable slot size with LRU eviction policy. Likewise for the nightly builds.

The GitHub bot (aka phsft-bot) should also live in the repository (as it is one or two files) and should be cleaned up to match the new jenkins pipeline infrastructure.
None of the proposed changes require major infrastructure outages -- all can be done with a separate job and initial (simple) jenkinsfile in the ROOT repository.

ROOT Team Meeting 7-Sep-2020 Feedback
===

JB: Faster builds like 30 mins per PR.

EG: Improve reproducibility of a build node setup. We need to be able eaily add nodes and reproduce build failures outside. Maybe spawn docker, ansible, etc.

AN: Central repo for build node config would be great. Maybe conflicts with the long lived VMs.

AN: Share build logs from the outside -- not relying on cdash and put the logs in GH

OS: It is an issue.

PC: If builds are long we need information about its state -- is it in the queue or some other state.

VV: We should watch about the bot sending many emails (CMSSW)

VE: We should be able to see which are the current versions of the used dependent-for-a-build packages

VV: VE, would the config need to be in the config in the jenkins file? A: maybe.

EG: It would be great if we can move away from groovy. Maybe using declarative pipelines?

AN: We should be able to steer the CI infra from GH. We should also improve the cleanup procedures, eg. zombie incrementals. 

AN: We should have at least PR, incr, release configured the same way (documentation generation included), sharing the same system (use the same configuration).

AN: We should test different release branches. We should be able to backport the jenkinsfile to v618, including the new branchaes and the old branches which makes ... Works on linux, osx, ppc, arm.

VV: Implement PR coverage information -- if a PR increases/decreses ROOT testing coverage

ET: A given PR choses a particular flavor and it is fine -- then the same flavor on the nightlies fails.  

Xavi: why do not we want to use containers? A: we want build artifacts to debug.

SH: Address sanitizer -- it is expensive to run but we should have this. We should add this in a section where people see it.

JB: Do we use coverity? A: We have it (incl coverage build). Currently failing but somebody need to fix them.

JB/AN: We should have PR coverity like coverage.

Xavi: Are the build nodes working with the IT cloud infra. Are they puppetized, updated? A: No

JB: Puppet is one more thing to master.

VV: Consider using two jenkins service/nodes eg. like one in FNAL, or other place if we have outage or lack of resources.


References
===

[1] https://www.google.com/search?q=jenkinsfile+cpp+site%3Agithub.com&oq=jenkinsfile+cpp+site%3Agithub.com

[2] https://github.com/cloudogu/jenkinsfiles

[3] https://thoughts-on-coding.com/2019/03/27/introduction-into-build-automation-setup-with-jenkins-and-cmake/

