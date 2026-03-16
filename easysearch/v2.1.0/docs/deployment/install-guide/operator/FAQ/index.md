---
title: "FAQ"
date: 0001-01-01
summary: "FAQ #    Easysearch 版本降级会报错 #    cannot downgrade a node from version [1.7.0] to version [1.6.1]
[2024-01-28T09:42:34,314][ERROR][o.e.b.EasysearchUncaughtExceptionHandler] [onenode-masters-0] uncaught exception in thread [main] org.easysearch.bootstrap.StartupException: java.lang.IllegalStateException: cannot downgrade a node from version [1.7.0] to version [1.6.1] at org.easysearch.bootstrap.easysearch.init(Easysearch.java:173) ~[easysearch-1.6.1.jar:1.6.1] at org.easysearch.bootstrap.easysearch.execute(Easysearch.java:160) ~[easysearch-1.6.1.jar:1.6.1] at org.easysearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:71) ~[easysearch-1.6.1.jar:1.6.1] at org.easysearch.cli.Command.mainWithoutErrorHandling(Command.java:112) ~[easysearch-cli-1.6.1.jar:1.6.1] at org.easysearch.cli.Command.main(Command.java:75) ~[easysearch-cli-1.6.1.jar:1.6.1] at org.easysearch.bootstrap.easysearch.main(Easysearch.java:125) ~[easysearch-1.6.1.jar:1.6.1] at org.easysearch.bootstrap.easysearch.main(Easysearch.java:67) ~[easysearch-1.6.1.jar:1.6.1] Caused by: java.lang.IllegalStateException: cannot downgrade a node from version [1."
---


# FAQ

- ### Easysearch 版本降级会报错

cannot downgrade a node from version [1.7.0] to version [1.6.1]

```text
[2024-01-28T09:42:34,314][ERROR][o.e.b.EasysearchUncaughtExceptionHandler] [onenode-masters-0] uncaught exception in thread [main]
org.easysearch.bootstrap.StartupException: java.lang.IllegalStateException: cannot downgrade a node from version [1.7.0] to version [1.6.1]
at org.easysearch.bootstrap.easysearch.init(Easysearch.java:173) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.easysearch.execute(Easysearch.java:160) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:71) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.cli.Command.mainWithoutErrorHandling(Command.java:112) ~[easysearch-cli-1.6.1.jar:1.6.1]
at org.easysearch.cli.Command.main(Command.java:75) ~[easysearch-cli-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.easysearch.main(Easysearch.java:125) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.easysearch.main(Easysearch.java:67) ~[easysearch-1.6.1.jar:1.6.1]
Caused by: java.lang.IllegalStateException: cannot downgrade a node from version [1.7.0] to version [1.6.1]
at org.easysearch.env.NodeMetadata.upgradeToCurrentVersion(NodeMetadata.java:79) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.env.NodeEnvironment.loadNodeMetadata(NodeEnvironment.java:418) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.env.NodeEnvironment.<init>(NodeEnvironment.java:315) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.node.Node.<init>(Node.java:343) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.node.Node.<init>(Node.java:273) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:212) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.Bootstrap.setup(Bootstrap.java:212) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.Bootstrap.init(Bootstrap.java:378) ~[easysearch-1.6.1.jar:1.6.1]
at org.easysearch.bootstrap.easysearch.init(Easysearch.java:169) ~[easysearch-1.6.1.jar:1.6.1]
```
- ### 基于已有的 PVC 构建新集群报错
```text
2024-03-24T20:21:36.975095805+08:00 Caused by: org.easysearch.cluster.coordination.CoordinationStateRejectedException: join validation on cluster state with a different cluster uuid rCyF7ZGyRL275eWC0qoTDQ than local cluster uuid uSFTnF1kRU661ENZRAbq-A, rejecting
2024-03-24T20:21:36.975099446+08:00 	at org.easysearch.cluster.coordination.JoinHelper.lambda$new$5(JoinHelper.java:149) ~[easysearch-1.7.1.jar:1.7.1]
2024-03-24T20:21:36.975103455+08:00 	at com.infinilabs.security.ssl.transport.SecuritySSLRequestHandler.messageReceivedDecorate(SecuritySSLRequestHandler.java:201) ~[?:?]
2024-03-24T20:21:36.975107101+08:00 	at com.infinilabs.security.transport.SecurityRequestHandler.messageReceivedDecorate(SecurityRequestHandler.java:330) ~[?:?]
```

