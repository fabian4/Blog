---
title: Maven 依赖分离打包
date: 2020/10/27
description: Maven 依赖分离打包
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/lVO4zdjItm217yA.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/lVO4zdjItm217yA.png'
categories:
  - Spring
tags:
  - SpringBoot
  - Maven
abbrlink: 25810
---

# Maven 依赖分离打包

springboot项目maven打包不配置的话，会把用到的jar包、resource文件、class文件总的打成一个jar包。

如果在测试环境或者正式环境，仅仅需要修改某一个class文件，或者引用到的jar（有可能是自己的项目的底层包jar），就要重新全部打一个包，比较麻烦，也怕万一正式环境全量更新出问题，因此需要分离出class文件、lib，方便增量更新。

修改 `pom.xml` 文件如下

~~~xml
<build>
    <plugins>
        <plugin>
            <!-- 要首先去掉 Springboot 提供的打包方式 -->
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <!-- 指定依赖文件目录，这样jar运行时会去找到同目录下的lib文件夹下查找 -->
                    <manifest>
                        <addClasspath>true</addClasspath>
                        <classpathPrefix>lib/</classpathPrefix>
                        <mainClass>me.fabian4.yocotoadmin.YocotoAdminApplication</mainClass>
                    </manifest>
                    <!-- 指定配置文件目录，这样jar运行时会去找到同目录下的resources文件夹下查找 -->
                    <manifestEntries>
                        <Class-Path>resources/</Class-Path>
                    </manifestEntries>
                </archive>
                <!-- 跳过配置文件和静态文件 -->
                <excludes>
                        <exclude>*.yml</exclude>
                        <exclude>*.xml</exclude>
                        <exclude>*.txt</exclude>
                    </excludes>
            </configuration>
        </plugin>
        <plugin>
            <!-- 移动资源文件 -->
            <artifactId>maven-resources-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                    <configuration>
                        <!-- 资源文件输出目录 -->
                        <outputDirectory>${project.build.directory}/resources</outputDirectory>
                        <resources>
                            <resource>
                                <directory>src/main/resources</directory>
                            </resource>
                        </resources>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <!-- 移动依赖jar文件 -->
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>prepare-package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/lib</outputDirectory>
                        <overWriteReleases>false</overWriteReleases>
                        <overWriteSnapshots>false</overWriteSnapshots>
                        <overWriteIfNewer>true</overWriteIfNewer>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>


~~~
