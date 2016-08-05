---
layout: posts
title: "Jenkins job export and import"
description: "Jenkins job export and import"
category: jenkins
tags: [jenkins]
---

# Jenkins job export and import

1 Download jenkins-cli.jar from jenkins server via [http://jenkinsserver/jnlpJars/jenkins-cli.jar](http://jenkinsserver/jnlpJars/jenkins-cli.jar)

2 export job

```cmd
java -jar jenkins-cli.jar -s http://jenkinsserver get-job myJob > myJob.xml
```

3 import job

```cmd
java -jar jenkins-cli.jar -s http://jenkinsserver create-job myJob < myJob.xml
```

## References
* [Jenkins CLI](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI)