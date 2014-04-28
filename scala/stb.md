SBT
===

Install
-------

```shell
wget http://dl.bintray.com/sbt/debian/sbt-0.13.2.deb
sudo dpkg -i sbt-0.13.2.deb
```

Create Project Direction
------------------------

```
mkdir $project
cd $project
mkdir -p src/main/scala
mkdir -p src/main/java
mkdir -p src/main/resources
mkdir -p src/test/scala
mkdir -p src/test/java
mkdir -p src/test/resources
```

Build definition
----------------

*build.sbt*

```scala
name := "<project>"

version := "1.0"

scalaVersion := "2.10.3"

```

Setting the sbt version
-----------------------

*project/build.properties*

```scala
sbt.version=0.13.2
```

Configuring version control
---------------------------

.gitignore

```
target/

```


Build Definition
----------------

An sbt build definition can contain files ending in .sbt, located in the base directory of a project, and files ending in .scala, located in the project/ subdirectory of the base directory.



Managed Dependencies
--------------------

```scala
libraryDependencies += groupID % artifactID % revision

libraryDependencies += groupID % artifactID % revision % configuration

libraryDependencies ++= Seq(
    groupID % artifactID % revision,
    groupID % otherID % otherRevision
)

libraryDependencies += "org.scala-tools" % "scala-stm_2.9.1" % "0.3"

libraryDependencies += "org.scala-tools" %% "scala-stm" % "0.3"

resolvers += "Sonatype OSS Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots"

resolvers += "Local Maven Repository" at "file://"+Path.userHome.absolutePath+"/.m2/repository"

```

