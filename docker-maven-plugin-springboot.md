---
title: 打包SpringBoot项目为docker镜像
date:  2023-03-02
---
> 使用docker-maven-plugin插件来实现打包SpringBoot项目为docker镜像
> 
> 开发机器需要安装docker

![项目][projectlayout]

### 方式一



1. 在pom.xml文件中增加docker-maven-plugin配置，填写imageName：最终打包成的镜像名称；baseImage：依赖的基础镜像；entryPoint：容器启动命令

``` xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.0</version>
    <configuration>
        <imageName>wangli/demo6:v2</imageName>
        <baseImage>ascdc/jdk8</baseImage>
        <entryPoint>["java","-jar","/${project.build.finalName}.jar"]</entryPoint>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

2. 执行maven打包命令，打包同时生成Docker镜像文件
```cmd
mvn clean package docker:build
```

3. 运行docker容器

```cmd
docker run -d --name demo9 -p 8888:8080 wangli/demo6:v2
```

### 方式二



1. 创建src/main/docker/Dockerfile文件,内容如下

```bash
FROM ascdc/jdk8
VOLUME /tmp
ADD demo6-0.0.1-SNAPSHOT.jar /app.jar
EXPOSE 8000
ENTRYPOINT  ["java","-jar","/app.jar"]
```



2. 修改pom.xml文件配置如下

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.0</version>
    <configuration>
        <imageName>wangli/demo6:v2</imageName>
        <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```


3. 执行maven打包命令,即可生成镜像
```cmd
mvn clean package docker:build

```

### 方式三



1. 将镜像打包绑定到maven的package阶段，同方式二第1步，需创建src/main/docker/Dockerfile文件



2. 修改pom.xml文件配置如下

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.2.0</version>
    <executions>
        <execution>
            <id>build-image</id>
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <imageName>wangli/demo6:v3</imageName>
        <dockerDirectory>${project.basedir}/src/main/docker</dockerDirectory>
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <directory>${project.build.directory}</directory>
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>

```



3. 执行mvn clen package 即会自动打包项目并创建docker镜像

```bash
mvn clean package
```

[projectlayout]data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAB38AAAO3CAIAAACstxmDAAAACXBIWXMAABYlAAAWJQFJUiTwAAAgAElEQVR42uy9CXsTd5qv3R/kfc876QQINsZ4k23Z2i3v+4ptvC/gBYzBNhjvBttAMCHB2YEsBIeddMgkE6e75yxzznXm9Jwz052e9Qv0+RhvlUpLqTaVNq/377ovLqtUqir9q1RCtx499atChwsAAAAAAAAAAAAAILH8iiEAAAAAAAAAAAAAAOwzAAAAAAAAAAAAAGCfAQAAAAAAAAAAAAD7DAAAAAAAAAAAAACAfQYAAAAAAAAAAAAA7DMAAAAAAAAAAAAAYJ8BAAAAAAAAAAAAALDPAAAAAAAAAAAAALBH7LO1afHR1tbP4dnaXGywO4V7GxY2hZuPFlr8M49sCDfvjjiTvcXGW6X9kPBtU2x5ohja2NIbAWmbtzZGk7f2OAdTc7OFZ7S1tTnfFO8+3VVPOfaxsrfMb4YOPGF3DtnjGplte8moj1LpUEzw+OzE04n5jBHDdkpDtw+O5ODOMn8KjfiiNrP34z9CErUrk31yiHNsd+QlZvL9y+T5UHlv3Ccczc2T1qJ+k0r4G+6uOrlJz07zGQlP1vxb9q49YwMAAAAAwEG0z3of23bWPkf1YTKiffZp1sTIRM2P2dIapQ3YnfZZ8+nvZvuckF0