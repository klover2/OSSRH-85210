## 准备工作

开发环境为：win11 + jdk8 + maven

## 注册账户

[Sign up for Jira - Sonatype JIRA](https://issues.sonatype.org/secure/Signup!default.jspa)
选择中文 比较好

如果未使用过 Sonatype 系统，需要先注册一个帐号。

- Email: 填写自己的邮箱帐号即可，在 Sonatype 上的相关操作，会通知到这个邮箱帐号来提醒你相关进度。
- Full name: 填写联系人名称。
- Username: 这个就是你在 Sonatype 的登录帐号了。
- Password: 为登录密码，要求至少 8 位，并带有大小写字母和字符。

## 项目申请

### 创建工单

可以理解为在 Sonatype 系统中申请一个自己的项目，以 Group Id 为单位，Group Id 支持域名地址和 Github 地址两种方式验证，大家可根据实际情况自行选择。申请后需要经过管理员对我们填写的 Group Id 所有权进行验证，审核通过后方可上传 jar 包。我们在页面顶部点击 Create 按钮来创建工单。

- Project: 项目类型。选择 Community Support - Open Source Project Repository Hosting (OSSRH)
- Issue Type: 工单类型。选择 New Project
- Summary: 填写 jar 包名称即可
- Description: 选填项，写一些描述
- Group Id: 填写项目的 Group Id，域名和 Github Group Id 两种方式下面分别介绍申请过程
- Project URL: 填写项目主页，可以是托管地址，也可以是官网地址
- SCM URL: 填写项目的 Git 地址
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/7320f8f0f35e4d069fad0082c289a67a.png)
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/5204dc640c6e442584af3cdc78eeebf6.png)
  按照图片中提示 在 githup 中创建名字为`OSSRH-85210`(每个人不一样)的空项目，同时把 project url 和 scm url 这两个地址改成你当前项目名称，然后就是等待审核。

### 审核通过

当你收到如下图的回复 说明你就成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/8b04fa253c3d4fb99c0ef62683d6657d.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/159e7360235d47cea7e641caa7bbea2c.png)

## GPG 签名

GPG 的主要作用是生成密钥对，发布到 Maven 仓库中的所有文件都要使用 GPG 签名，以保障完整性。使用 Windows 的可以下载 gpg4win 地址：https://www.gpg4win.org/download.html ，安装后执行 gpg --gen-key 命令新建密钥对。

执行命令后，需要输入 姓名 和 邮箱 还有 Passphase（证书密码），Passphase 需要记住，在我们使用证书的时候会要求我们输入密码。

![在这里插入图片描述](https://img-blog.csdnimg.cn/07483a7bc7244173bf44e1f8eb019889.png)
执行完之后我们会得到上图中的 pub 串，接着我们执行以下命令将 pub 上传到 key 验证库(不成功多试几次)
`gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys E89F2504BF0CE3337B26E45E485D1495DA08F9F6 `
查询是否成功
`gpg --keyserver hkp://keyserver.ubuntu.com:11371 --recv-keys E89F2504BF0CE3337B26E45E485D1495DA08F9F6`

获取 sign3 (后面 maven setting.xml 用得上)
`gpg --list-signatures --keyid-format 0xshort`
![在这里插入图片描述](https://img-blog.csdnimg.cn/d6ddfb5b492140d9bd2bc64c2baa6e12.png)

## Maven 配置

- 在 Maven 全局配置文件 setting.xml 中配置

```xml
<server>
      <id>ossrh</id>
      <username>sonatype 账户名称</username>
	<password>sonatype 密码</password>
    </server>
```

- 配置 gpg 的账户密码，复制以下配置，修改为自己的密码后，添加到 setting.xml 文件中的

```xml
<profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <gpg.executable>gpg</gpg.executable>
        <gpg.keyname>上面你获取的sign3</gpg.keyname>
        <gpg.passphrase>gpg 你设置的密码</gpg.passphrase>
      </properties>
    </profile>
```

## 项目配置

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 对应上面的groupId 不能乱写 -->
    <groupId>io.github.klover2</groupId>
    <artifactId>你自己的项目名称</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>
    <name>名称</name>
    <description>描述</description>
    <url>https://github.com/klover2/**</url>

    <!--  apache 2.0 协议  -->
    <licenses>
        <license>
            <name>The Apache Software License, Version 2.0</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
        </license>
    </licenses>

    <!-- SCM信息 -> git在github上托管 -->
    <scm>
        <connection>scm:git:https://github.com/klover2/**.git</connection>
        <url>git:https://github.com/klover2/**</url>
    </scm>
    <!-- 开发者信息 -->
    <developers>
        <developer>
            <name>klover</name>
        </developer>
    </developers>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <java.version>8</java.version>

        <!-- 插件版本 直接拷贝 start -->
        <nexus-staging-maven-plugin.version>1.6.7</nexus-staging-maven-plugin.version>
        <maven-source-plugin.version>3.2.1</maven-source-plugin.version>
        <maven-javadoc-plugin.version>3.4.1</maven-javadoc-plugin.version>
        <maven-gpg-plugin.version>3.0.1</maven-gpg-plugin.version>
        <maven-release-plugin.version>2.5.3</maven-release-plugin.version>
        <!-- 插件版本 直接拷贝 end -->
    </properties>

    <dependencies>
        <!-- 插件包 直接拷贝 start -->
        <dependency>
            <groupId>org.sonatype.plugins</groupId>
            <artifactId>nexus-staging-maven-plugin</artifactId>
            <version>${nexus-staging-maven-plugin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>${maven-source-plugin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>${maven-javadoc-plugin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>${maven-gpg-plugin.version}</version>
        </dependency>
        <!-- 插件包 直接拷贝 end -->
    </dependencies>


    <!-- 默认配置 直接拷贝 start -->
    <profiles>
        <profile>
            <id>release</id>
            <build>
                <plugins>
                    <!--生成源码插件-->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>${maven-source-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>attach-sources</id>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!--生成API文档插件-->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>${maven-javadoc-plugin.version}</version>
                        <executions>
                            <execution>
                                <id>attach-javadocs</id>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

    <build>
        <!--发布到中央SNAPSHOT仓库插件-->
        <plugins>
            <plugin>
                <groupId>org.sonatype.plugins</groupId>
                <artifactId>nexus-staging-maven-plugin</artifactId>
                <version>${nexus-staging-maven-plugin.version}</version>
                <extensions>true</extensions>
                <configuration>
                    <serverId>ossrh</serverId>
                    <nexusUrl>https://s01.oss.sonatype.org/</nexusUrl>
                    <!-- 开启true 自动同步-->
                    <autoReleaseAfterClose>false</autoReleaseAfterClose>
                </configuration>
            </plugin>
            <!-- 使用Maven Release插件执行发布部署 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>${maven-release-plugin.version}</version>
                <configuration>
                    <autoVersionSubmodules>true</autoVersionSubmodules>
                    <useReleaseProfile>false</useReleaseProfile>
                    <releaseProfiles>release</releaseProfiles>
                    <goals>deploy</goals>
                </configuration>
            </plugin>
            <!--gpg插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-gpg-plugin</artifactId>
                <version>${maven-gpg-plugin.version}</version>
                <executions>
                    <execution>
                        <id>sign-artifacts</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>sign</goal>
                        </goals>
                        <configuration>
                            <!--suppress UnresolvedMavenProperty -->
                            <keyname>${gpg.keyname}</keyname>
                            <!--suppress UnresolvedMavenProperty -->
                            <passphraseServerId>${gpg.keyname}</passphraseServerId>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <distributionManagement>
        <snapshotRepository>
            <!--注意,此id必须与setting.xml中指定的一致-->
            <id>ossrh</id>
            <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
        </snapshotRepository>
        <repository>
            <id>ossrh</id>
            <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
    </distributionManagement>
    <!-- 默认配置 直接拷贝 end -->
</project>
```

## 打包发布

完成以上配置，我们就可以往 Maven 中央仓库上传 jar 包了，我们使用 Maven 命令上传。

> `mvn clean deploy -P release`

如果是子项目 > `mvn clean deploy -projects demo -P release`
如下图 就说明发布成功了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a75459d2b6754beeb2973fc5305713aa.png)

届时你可以在[Nexus Repository Manager](https://s01.oss.sonatype.org/):中找到.
![在这里插入图片描述](https://img-blog.csdnimg.cn/24027c5a1c584085bd6d187f9a8f6eab.png)

4 小时内你可以在 [Maven Central Repository Search](https://search.maven.org/)中搜索到.

在[mvnrepository](https://mvnrepository.com/)中搜索找可能需要几天。

## 注意

1. 如果你版本号是以 SNAPSHOT 结尾，不会发布 maven 中央仓库。
2. `<autoReleaseAfterClose>false</autoReleaseAfterClose>` 由于我关闭了自动同步，所以需要在 Staging Repositories 手动点 release 发布
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/09b78d71349b4d4b86951141f3b045d4.png)
3. 是否需要发布单元测试文件

```bash
-DskipTests，不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下

-Dmaven.test.skip=true，不执行测试用例，也不编译测试用例类
```

## 出现的问题

1. nexus staging maven 插件：maven 部署失败：执行时遇到 API 不兼容

配置环境变量：> `export MAVEN_OPTS="--add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.text=ALL-UNNAMED --add-opens=java.desktop/java.awt.font=ALL-UNNAMED"`

![在这里插入图片描述](https://img-blog.csdnimg.cn/7582168ed594457a85165ed15b50ce2e.png)
