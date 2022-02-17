---
title: "Gradle"
date: 2022-02-16T10:58:10+08:00
draft: true
toc: true
tags: 
  - engineering

---

```
export GRADLE_PATH=/opt/gradle-7.4
export PATH=$PATH:$GRADLE_PATH/bin
```



```
~/.gradle touch gradle.properties
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
org.gradle.parallel=true
org.gradle.configureondemand=true
systemProp.file.encoding=UTF-8  
```

