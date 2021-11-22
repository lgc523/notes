---
title: "Maven CLI、Plugin、CFG"
date: 2021-11-12T23:18:53+08:00
draft: true
toc: true
images:
tags: 
  - engineering
---

开源项目比较大的一般都是用gradle构建，之前看的gradle也快忘了，最近开始看一些开源的项目，逐步完善记录一下。

个人觉得使用 CLI 是一件很酷很优雅的事情，搭配环境变量可以很方便的操作，并且知道每一步在做什么。

## maven

``base v3.8.2``

### Common commands

1. **mvn clean install -T 1C -Dmaven.test.skip=true**
2. **mvn mybatis-generator:generate**
3. **mvn spring-boot:run**
4. **mvn dependency:tree -T 1C**
5. **mvn deploy xxx:jar**
6. **mvn help:describe -Dplugin="groupId:artifactId:version" [-Ddetail]**
7. mvn clean versions:set -DnewVersion=2.0.0-SNAPSHOT #替换版本号 后面在进行操作
8. mvn versions:revert #回滚版本号
9. mvn deploy -s /xxxx/conf/settings.xml #使用指定配置文件
9. **mvn help:active-profiles**

### Test

Maven Surefire Plugin，Bound to phase test

```
-Dmaven.test.skip=true 					 跳过单元测试编译和执行
-DskipTests 					 					 跳过测试执行，会进行编译
-Dmaven.test.failure.ignore=true 忽略错误继续编译构建 <testFailureIgnore>true</testFailureIgnore> 
-Dtest=TestClassName test 			 指定测试类

```

```
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-surefire-plugin</artifactId>  
    <configuration>  
        <skip>true</skip>  
         <skipTests>true</skipTests> 
    </configuration>  
</plugin>
#springboot 需要在 properties 中配置
<properties>
<skipTests>true</skipTests>
</properties>
```



### Jar struct

```
├── 1.0
│   ├── _remote.repositories
│   ├── scala-action-1.0.jar
│   └── scala-action-1.0.pom 完成的pom
└── maven-metadata-local.xml <metadata>.<lastUpdated>有格式化的时间yyyyMMddHHmmss
```

### snapshot

开发过程中，模块之间相互依赖，迭代版本比较快，**Snapshot 版本每次发布会带上时间戳**，构建拉取依赖总是会拉到最新的版本。

maven-repository-meta 定义的 snapshot metadata

```
    private String timestamp;
    private int buildNumber = 0;
    private boolean localCopy = false; # 是否本地副本
```



### release



### dependency transitive

Maven 的**最近依赖策略**：使用依赖树  ``mvn dependency:tree`` 中离项目最近的版本

optional 在单纯的项目依赖中可以减少依赖大小，A->B->{C，D}(optional)，A 项目需要**显示的引用**C/D，**父子工程下不适用**

底层依赖没有用optional，上层可以通过 exclusions 排除，底层依赖越多，exclusions 写的就越多。

### dependency conflict

1. mavenHelper 插件
2.  <execlusions> 插件排除依赖
3. ``mvn dependency:tree -T 1C|grep org.yaml:snakeyaml``  检测依赖的版本
3. ``mvn dependency:list/tree/analyze``
3. **shade plugin**  **relocation** ，将冲突的包改包名 

### lifeCycle

- clean
- validate
- compile
- test
- package
- verify
- install
- site
- deploy

每个生命周期包含一系列 phase，由插件完成（goal），与插件绑定。

**mvn [pulgin]:[goal]** 

可以将插件和抽象的声明周期绑定，或者指定插件和目标

一共23个阶段，主要是一些环境配置的校验、代码的生成、编译、拷贝。。。

### scope

- compile	 # default 
- provided   #**编译期需要，运行期不需要**
- runtime     
- test
- system      # 引入本地lib
- optional    # 当项目是依赖时，用于连续依赖使用
- import
- compile + runtime
- runtime + system

### properties

可用的配置变量 ${}

- env.x 大写环境变量
- project.x 
- settings.x
- Java properties  System.getProperties().list(System.out)
- properties 定义的标签名



### java 9 or later 

**maven-compiler-plugin** 在 **java9+** 版本中，版本要求 **3.6.0+** ，并且需要设置 properties

```
<properties>
        <maven.compiler.release>11</maven.compiler.release>
</properties>
```

### relativePath

配置父模块的相对路径，默认是 ``../pom.xml``，设置空值向本地仓库和远程仓库找。

### CLI options

V3.8.2 https://maven.apache.org/ref/3.8.2/maven-embedder/cli.html

### Describe

看一些开源项目，总是用了一堆的插件，help:describe 可以看插件的描述信息(goals)，可以指定版本号，可以使用 **Goal Prefix** 作为plugin标识，但是只能显示最新版本的插件信息

**mvn help:describe -Dplugin="groupId:artifactId:version" [-Ddetail] [-Dgoal=xxx]**

**正常执行会每次去拉取一些插件的信息，如果插件本地存在，可以加上 -o 使用 offline 模式**

### ext

用户可以自己拓展插件，命名规范 <my>-maven-plugin

- packing

  ```
   <packaging>maven-plugin</packaging>
  ```

- dependency

  ```
   <dependency>
              <groupId>org.apache.maven</groupId>
              <artifactId>maven-plugin-api</artifactId>
              <version>3.0</version>
              <scope>provided</scope>
          </dependency>
  
          <dependency>
              <groupId>org.apache.maven.plugin-tools</groupId>
              <artifactId>maven-plugin-annotations</artifactId>
              <version>3.4</version>
              <scope>provided</scope>
          </dependency>
  ```

- extend

  ```
  @Mojo(name = "hello")
  public class HelloMojo extends AbstractMojo {
  
      Log lg = getLog();
      Map ctx = getPluginContext();
  
      public void execute() {
          lg.info("Hello，definition use @goal at doc cause conflict");
          ctx.put("mojoName", this.getClass().getName());
          getLog().info("inject context over!");
      }
  }
  只有 execute 里面可以执行，携带 log,context 实例
  ```

- lifecycle

  ```
  涉及到 plexus 容器
  @Component(role = AbstractMavenLifecycleParticipant.class)
  public class MavenLIfeCycleHook extends AbstractMavenLifecycleParticipant {
  
      @Override
      public void afterSessionStart(MavenSession session) throws MavenExecutionException {
          System.out.println("sessions start ...");
          MavenExecutionRequest request = session.getRequest();
          System.out.println(JSON.toJSONString(request));
          super.afterSessionStart(session);
      }
  
      @Override
      public void afterProjectsRead(MavenSession session) throws MavenExecutionException {
          MavenExecutionRequest request = session.getRequest();
          System.out.println("after projects read...");
          System.out.println(JSON.toJSONString(request));
          Settings settings = session.getSettings();
          System.out.println(JSON.toJSONString(settings));
          super.afterProjectsRead(session);
      }
  
      @Override
      public void afterSessionEnd(MavenSession session) throws MavenExecutionException {
          System.out.println("session end...");
          MavenExecutionRequest request = session.getRequest();
          System.out.println(JSON.toJSONString(request));
          super.afterSessionEnd(session);
      }
  }
  
  ```

  

### repository

通过 maven-core  包下面的 RepositoryRequest 接口可以看出，repository 可配置的有 网络权限、强制更新、获取本地、远程仓库

```
boolean isOffline();
RepositoryRequest setOffline( boolean offline );
boolean isForceUpdate();
RepositoryRequest setForceUpdate( boolean forceUpdate );
ArtifactRepository getLocalRepository();
RepositoryRequest setLocalRepository( ArtifactRepository localRepository );
RepositoryRequest setRemoteRepositories( List<ArtifactRepository> remoteRepositories );
```

通过构造参数，可以看到 本地仓库只能是一个，远程仓库可以是多个。

### artififact

```
		private String groupId;

    private String artifactId;

    private String baseVersion;

    private final String type;

    private final String classifier;

    private volatile String scope;

    private volatile File file;

    private ArtifactRepository repository;

    private String downloadUrl;

    private ArtifactFilter dependencyFilter;

    private ArtifactHandler artifactHandler;

    private List<String> dependencyTrail;

    private volatile String version;

    private VersionRange versionRange;

    private volatile boolean resolved;

    private boolean release;

    private List<ArtifactVersion> availableVersions;

    private Map<Object, ArtifactMetadata> metadataMap;

    private boolean optional;
```

### threadSafe

有些插件不支持 threadSafe 开启多线程需要注意警告信息

### spring-boot

``mvn help:describe -Dplugin=spring-boot -Ddetail [-Dgoal=xxx]``

```
Name: Spring Boot Maven Plugin
Description: Spring Boot Maven Plugin
Group Id: org.springframework.boot
Artifact Id: spring-boot-maven-plugin
Version: 2.2.5.RELEASE
Goal Prefix: spring-boot
This plugin has 6 goals:
```

关于 springboot 打包的问题，通过 ``mvn help:describe -Dplugin=spring-boot -Ddetail`` 可以看到几个关键的 goal，repackage、a tech，会影响打包的结果。

能不能通过插件影响 springboot jar包结构，在编译期影响文件，来提高项目的启动速度， 目前有一个实现是在 spring 环境 启动 be an post 回调完成的，会很慢。

### plugin

#### shade

Shade plugin 将所有依赖的包打到一块(repackage)，指定 main 函数，构建可执行 jar 包，配合 classifierName 可以隔离版本解决依赖冲突。

```
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <shadedArtifactAttached>true</shadedArtifactAttached>
                            <shadedClassifierName>jdk15</shadedClassifierName>
                            <createDependencyReducedPom>false</createDependencyReducedPom>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass>dev.spider.cli.ApacheCli</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
</plugin>
```

#### javadoc

```
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-javadoc-plugin</artifactId>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <charset>UTF-8</charset>
                    <docencoding>UTF-8</docencoding>
                </configuration>
                <executions>
                    <execution>
                        <id>attach-javadocs</id>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
</plugin>
```

#### versions

常用的升级版本策略

```
mvn versions:set -DnewVersion [=1.2.0-SNAPSHOT] 升级版本
mvn versions:revert
mvn versions:set -DremoveSnapshot=true -DgenerateBackupPoms=false 去掉 snapshot
mvn help:describe -Dplugin=versions -o -Dgoal=set -Ddetail 
```

#### release

发布 release 版本，可选 tag/branch，实现递增版本号，发布release jar

可以从 SNAPSHOT 转换到 release ，还没用过，后面用了再说.
