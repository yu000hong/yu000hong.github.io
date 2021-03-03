---
layout: post
title: YARN如何配置cgroup
date:  2021-02-20 17:00:00 +0800
tags: [YARN, cgroup]
---


### yarn-site.xml

需要配置如下选项：

- yarn.nodemanager.container-executor.class
- yarn.nodemanager.linux-container-executor.path
- yarn.nodemanager.linux-container-executor.nonsecure-mode.limit-users
- yarn.nodemanager.linux-container-executor.group
- yarn.nodemanager.linux-container-executor.resources-handler.class
- yarn.nodemanager.linux-container-executor.cgroups.mount
- yarn.nodemanager.linux-container-executor.cgroups.mount-path
- yarn.nodemanager.linux-container-executor.cgroups.hierarchy
- yarn.nodemanager.linux-container-executor.cgroups.strict-resource-usage
- yarn.nodemanager.resource.percentage-physical-cpu-limit

#### yarn.nodemanager.container-executor.class

```xml
<property>
  <name>yarn.nodemanager.container-executor.class</name>
  <value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>
</property>
```

这里需要配置为`LinuxContainerExecutor`才能使用Linux cgroups功能。

#### yarn.nodemanager.linux-container-executor.path

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.path</name>
  <value>/data0/container-executor/bin/container-executor</value>
</property>
```

配置`container-executor`二进制可执行文件的具体路径，默认值为: "$HADOOP_HOME/bin/container-executor"。如果我们集群是以root用户启动的，那么这里可以不用配置，否则就需要进行配置，因为`container-executor`要求其本身以及配置文件所在的目录树的所有者都必须为root。

这里假设我们集群是以用户`yu000hong`执行的，那么需要如下几个步骤：

```bash
[yu000hong@hadoop0:~] $ sudo mkdir /data0/container-executor 
[yu000hong@hadoop0:~] $ sudo mkdir /data0/container-executor/bin
[yu000hong@hadoop0:~] $ sudo mkdir /data0/container-executor/etc/hadoop
[yu000hong@hadoop0:~] $ sudo cp $HADOOP_HOME/bin/container-executor /data0/container-executor/bin/
[yu000hong@hadoop0:~] $ sudo cp $HADOOP_HOME/etc/hadoop/container-executor.cfg /data0/container-executor/etc/hadoop/
[yu000hong@hadoop0:~] $ sudo chown -R root:root /data0/container-executor
[yu000hong@hadoop0:~] $ sudo chown root:yu000hong /data0/container-executor/bin/container-executor
[yu000hong@hadoop0:~] $ sudo chmod u+s /data0/container-executor/bin/container-executor
[yu000hong@hadoop0:~] $ sudo chmod o-rwx /data0/container-executor/bin/container-executor

[yu000hong@hadoop0:~] $ sudo cat /data0/container-executor/etc/hadoop/container-executor.cfg
yarn.nodemanager.linux-container-executor.group=yu000hong
banned.users=              #comma separated list of users who can not run applications
allowed.system.users=      #comma separated list of system users who CAN run applications
min.user.id=1000           #Prevent other super-users
feature.tc.enabled=false
```

#### yarn.nodemanager.linux-container-executor.nonsecure-mode.limit-users

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.nonsecure-mode.limit-users</name>
  <value>false</value>
</property>
```

如果不设置这个值为false，那么会报错：

```text
Application application_1613802414517_0002 failed 2 times due to AM Container for appattempt_1613802414517_0002_000002 exited with exitCode: -1000
Failing this attempt.Diagnostics: [2021-02-20 14:55:42.836]Application application_1613802414517_0002 initialization failed (exitCode=255) with output: main : command provided 0
main : run as user is nobody
main : requested yarn user is yu000hong
Requested user nobody is not whitelisted and has id 99,which is below the minimum allowed 1000
For more detailed output, check the application tracking page: http://gpu139.cd.weibonode.com:8088/cluster/app/application_1613802414517_0002 Then click on links to logs of each attempt.
. Failing the application.
```

#### yarn.nodemanager.linux-container-executor.group

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.group</name>
  <value>yu000hong</value>
</property>
```

设置`container-executor`的运行组，这里的值必须和二进制文件本身的所有者组一致。

#### yarn.nodemanager.linux-container-executor.resources-handler.class

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.resources-handler.class</name>
  <value>org.apache.hadoop.yarn.server.nodemanager.util.CgroupsLCEResourcesHandler</value>
</property>
```

这里必须设置为：`CgroupsLCEResourcesHandler`。

#### yarn.nodemanager.linux-container-executor.cgroups.mount

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.cgroups.mount</name>
  <value>false</value>
</property>
```

是否由YARN在启动的时候去挂载`cgroup VFS`，这里设置为"false"，一般系统都是已经挂载了cgroup的。

#### yarn.nodemanager.linux-container-executor.cgroups.mount-path

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.cgroups.mount-path</name>
  <value>/sys/fs/cgroup</value>
</property>
```

大部分系统挂载点都是**/sys/fs/cgroup**，也有**/cgroup**。

#### yarn.nodemanager.linux-container-executor.cgroups.hierarchy

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.cgroups.hierarchy</name>
  <value>/yarn</value>
</property>
```

这里我们配置为**/yarn**，那么我们就必须在cgroup controllers下面创建对应的目录，并且设置相应的属性：

```bash
[yu000hong@hadoop0:~] $ sudo mkdir /sys/fs/cgroup/cpu/yarn
[yu000hong@hadoop0:~] $ sudo chown -R yu000hong:yu000hong /sys/fs/cgroup/cpu/yarn
[yu000hong@hadoop0:~] $ sudo mkdir /sys/fs/cgroup/memory/yarn
[yu000hong@hadoop0:~] $ sudo chown -R yu000hong:yu000hong /sys/fs/cgroup/memory/yarn
```

目前YARN只用到了`cpu`/`memory`两类Controller，因此只需要在cpu和memory子目录下创建yarn目录即可，注意必须通过chown来修改权限以使YARN用户可以设置cgroup。

#### yarn.nodemanager.linux-container-executor.cgroups.strict-resource-usage

```xml
<property>
  <name>yarn.nodemanager.linux-container-executor.cgroups.strict-resource-usage</name>
  <value>true</value>
</property>
```

#### yarn.nodemanager.resource.percentage-physical-cpu-limit

```xml
<property>
  <name>yarn.nodemanager.resource.percentage-physical-cpu-limit</name>
  <value>90</value>
</property>
```

值范围为：(0, 100]。这里90可以严格控制YARN的CPU使用率不超过90%。


### capacity-scheduler.xml

```xml
<property>
  <name>yarn.scheduler.capacity.resource-calculator</name>
  <value>org.apache.hadoop.yarn.util.resource.DominantResourceCalculator</value>
</property>
```

默认值是`DefaulttResourceCalculator`，它只会考虑内存因素，不会考虑CPU，所以需要换成`DominantResourceCalculator`。


### 问题总结

#### container-executor.cfg must be owned by root

```text
2021-02-20 10:23:44,404 WARN org.apache.hadoop.yarn.server.nodemanager.containermanager.linux.privileged.PrivilegedOperationExecutor: Shell execution returned exit code: 24. Privileged Execution Operation
 Stderr:
File /data0/hadoop/hadoop-3.2.1/etc/hadoop/container-executor.cfg must be owned by root, but is owned by 1079

Stdout:
Full command array for failed execution:
[/data0/hadoop/hadoop-3.2.1/bin/container-executor, --checksetup]
```

`container-executor`可执行文件的配置文件的位置为：**../etc/hadoop/container-executor.cfg**。使用的是相对位置，是写死在C代码里的，没法配置。

`container-executor`可执行文件有一些安全要求，配置文件及其所有父目录的所有者都必须是root用户，如果采用默认配置，`container-executor`在**$HADOOP_HOME/bin** 目录下，因此要求其配置文件在**$HADOOP_HOME/etc/hadoop/container-executor.cfg**，那么**$HADOOP_HOME** 及其所有父级目录的所有者都必须是root。为了我们可以使用非root用户启动集群，那么就必须移动可执行文件的位置，同时也必须移动其配置文件的位置，操作如下：

```bash
[yu000hong@hadoop0:~] $ sudo mkdir /data0/container-executor 
[yu000hong@hadoop0:~] $ sudo mkdir /data0/container-executor/bin
[yu000hong@hadoop0:~] $ sudo mkdir /data0/container-executor/etc/hadoop
[yu000hong@hadoop0:~] $ sudo cp $HADOOP_HOME/bin/container-executor /data0/container-executor/bin/
[yu000hong@hadoop0:~] $ sudo cp $HADOOP_HOME/etc/hadoop/container-executor.cfg /data0/container-executor/etc/hadoop/
[yu000hong@hadoop0:~] $ sudo chown -R root:root /data0/container-executor
[yu000hong@hadoop0:~] $ sudo chown root:yu000hong /data0/container-executor/bin/container-executor
[yu000hong@hadoop0:~] $ sudo chmod u+s /data0/container-executor/bin/container-executor
[yu000hong@hadoop0:~] $ sudo chmod o-rwx /data0/container-executor/bin/container-executor
```


#### The container-executor binary should be set setuid

```text
2021-02-20 12:35:02,611 WARN org.apache.hadoop.yarn.server.nodemanager.containermanager.linux.privileged.PrivilegedOperationExecutor: Shell execution returned exit code: 22. Privileged Execution Operation
 Stderr:
Invalid permissions on container-executor binary.

Stdout: The container-executor binary should be set setuid.

Full command array for failed execution:
[/data0/hadoop/container-executor/bin/container-executor, --checksetup]
```

通过如下命令`chmod u+s /data0/container-executor/bin/container-executor`即可解决。

#### Requested user nobody is not whitelisted and has id 99,which is below the minimum allowed 1000

```text
Application application_1613802414517_0002 failed 2 times due to AM Container for appattempt_1613802414517_0002_000002 exited with exitCode: -1000
Failing this attempt.Diagnostics: [2021-02-20 14:55:42.836]Application application_1613802414517_0002 initialization failed (exitCode=255) with output: main : command provided 0
main : run as user is nobody
main : requested yarn user is yuhong4
Requested user nobody is not whitelisted and has id 99,which is below the minimum allowed 1000
For more detailed output, check the application tracking page: http://gpu139.cd.weibonode.com:8088/cluster/app/application_1613802414517_0002 Then click on links to logs of each attempt.
. Failing the application.
```

设置`yarn.nodemanager.linux-container-executor.nonsecure-mode.limit-users`为false即可解决。



### 参考资源

[Using CGroups with YARN](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/NodeManagerCgroups.html)

[Enable Cgroups](https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.5/data-operating-system/content/enabling_cgroups.html)

[Managing CPU Resources in your Hadoop YARN Clusters](https://blog.cloudera.com/managing-cpu-resources-in-your-hadoop-yarn-clusters/)

[Apache Hadoop YARN in HDP 2.2: Isolation of CPU resources in your Hadoop YARN clusters](https://blog.cloudera.com/apache-hadoop-yarn-in-hdp-2-2-isolation-of-cpu-resources-in-your-hadoop-yarn-clusters/)