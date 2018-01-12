---
layout: post
title:  "Maven Install Error"
subtitle: ""
date:  2018-1-12
author: "ABei"
catalog: true
tags: 
    - BUG
---

> 遇到问题别害怕，冷静思考就会解决。

今天用maven install失败，我还以为和以前一样，是相关依赖不对，所以失败。试了一会儿，发现不对还是有问题，接着去问同事，发现他们都没有问题。接着，我想可能是我本地有问题。仔细看了一下log：

```text
[INFO] Scanning for projects...
[WARNING] 
[WARNING] Some problems were encountered while building the effective model for com.cat.mcportal.shared:CatLightPortlet:war:1.0.0-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: commons-io:commons-io:jar -> duplicate declaration of version (?) @ com.cat.mcportal.shared:CatLightPortlet:[unknown-version], /home/portaldev/portal85workspace/CatLightPortlet/pom.xml, line 113, column 15
[WARNING] 
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING] 
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING] 
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building CatLightPortlet 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] Downloading: https://artifacts.cat.com/artifactory/cat-release/org/apache/maven/plugins/maven-dependency-plugin/2.8/maven-dependency-plugin-2.8.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.210 s
[INFO] Finished at: 2018-01-12T15:54:27+08:00
[INFO] Final Memory: 10M/120M
[INFO] ------------------------------------------------------------------------
[ERROR] Plugin org.apache.maven.plugins:maven-dependency-plugin:2.8 or one of its dependencies could not be resolved: Failed to read artifact descriptor for org.apache.maven.plugins:maven-dependency-plugin:jar:2.8: Could not transfer artifact org.apache.maven.plugins:maven-dependency-plugin:pom:2.8 from/to central ...: Access denied to ... Error code 401, Unauthorized -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginResolutionException
```

*Error code 401, Unauthorized*，错误显示账号未验证。在stackoverflow上面找了一个[高分答案](https://stackoverflow.com/questions/24830610/why-am-i-getting-a-401-unauthorized-error-in-maven)
看了一下settings.xml，我发现是账号密码过期了。改了一下密码，重新mvn install，BUILD SUCCESS!