# HBaseDS
Source project - https://github.com/sematext/HBaseWD

###Changes
The libraries used (hbase, hadoop etc) has been upgraded to use HDP version of the libraries & also hbase-server & zookeeper dependencies scope have been changed to provided.

###Build instructions
  - Clone the source:

        git clone https://github.com/phaneesh/HBaseDS.git

  - Build

        mvn install

# Repo
```
<repository>
    <id>clojars</id>
    <name>Clojars repository</name>
    <url>https://clojars.org/repo</url>
</repository>
```
# Dependency

## Maven
```
<dependency>
    <groupId>com.sematext.hbase.ds</groupId>
    <artifactId>hbase-ds</artifactId>
    <version>0.0.2-SNAPSHOT</version>
</dependency>
```
## Leiningen
```
[com.sematext.hbase.ds/hbase-ds "0.0.2-SNAPSHOT"]
```

## Gradle
```
compile "com.sematext.hbase.ds:hbase-ds:0.0.2-SNAPSHOT"
```
