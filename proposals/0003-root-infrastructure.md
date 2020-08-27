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

References
===

[1] https://www.google.com/search?q=jenkinsfile+cpp+site%3Agithub.com&oq=jenkinsfile+cpp+site%3Agithub.com

[2] https://github.com/cloudogu/jenkinsfiles

[3] https://thoughts-on-coding.com/2019/03/27/introduction-into-build-automation-setup-with-jenkins-and-cmake/

