---
title: Snap Development Documentation

language_tabs:

toc_footers:
  - <a href='http://doc.snaplogic.com/home'>Product Documentation</a>
  - <a href='http://www.snaplogic.com'>SnapLogic website</a>

includes:

search: true
---

# Introduction

This documentation guides a developer through the steps necessary to develop **Snaps** for the [SnapLogic Elastic Integration Platform](https://www.snaplogic.com/).

## Snaps and Snap Packs

SnapLogic [Snaps](https://www.snaplogic.com/snaps) are modular collections of integration components built for a specific application or data source. Snaps shield both business users and developers from much of the complexity of the underlying application, data model, and service.

![Transform Snap Pack](http://dl.dropboxusercontent.com/u/3519578/Screenshots/WmuJ.png)

Snap Packs logically organize Snaps and are the deployable unit when adding/modifying Snaps to the SnapLogic Elastic Integration Platform. For instance, in the above example, the Aggregate Snap is part of the Transform Snap Pack. 

Snaps may be related by functionality or share common code. A Snap Pack may contain a single Snap or multiple Snaps.</img>

## Snap Fundamentals

Snaps are streaming data processors. They can consume and/or produce Binary or Document data through input and output views, and can report error Documents to an optional error view. 

![Snap Anatomy](http://dl.dropboxusercontent.com/u/3519578/Screenshots/iKCt.png)

They have metadata that define their view settings and configuration. They can read/poll from an endpoint, batch execute or process each input individually, and may write to another endpoint.

Snaps can provide design-time Suggest/lookup and Preview/validate data capabilities, or run in full execution mode.

![Snap Execution](http://dl.dropboxusercontent.com/u/3519578/Screenshots/Yjfr.png)

## Snaps vs Scripts

SnapLogic's [Script Snap](http://doc.snaplogic.com/com-snaplogic-snaps-script-script_2) executes a JavaScript, Jython, or JRuby script using the JVM `ScriptEngine` mechanism. 

Consider the following when deciding whether to develop a custom Snap vs using the Script Snap:

Custom Snap | Script Snap
--------- | -----------
Supports easy configuration without coding | Working directly with code
Supports pipeline `_parameters` | Does not support pipeline `_parameters`
Supports Binary or Document input/output | Supports only Document input/output
Supports Accounts, simplify impersonation |	Does not support Accounts
Supports debugging during development | Often awkward to debug
Supports unit testing during/after development | Cannot be unit tested
Java | JavaScript, Jython, or JRuby
Need to build/deploy/etc | No deployment, quick & simple

# Prerequisites

* Java 8 (JDK)
* Maven 3
* A Snaplex, including the `keys.properties` and `global.properties` files/data provided by [SnapLogic Support](mailto:support@snaplogic.zendesk.com?subject=Requesting%20Snap%20Development%20Credentials) upon request for Snap development

<aside class="notice">
We also recommend using an Java IDE like <a href="https://www.jetbrains.com/idea">IntelliJ IDEA</a>, <a href="https://eclipse.org/ide">Eclipse</a>, or <a href="https://netbeans.org/">NetBeans</a>.
</aside>

# Setting up the Snaplex

A [Snaplex](http://doc.snaplogic.com/snaplex) is the data processing engine of the [SnapLogic Elastic Integration Platform](https://www.snaplogic.com/why-snaplogic/how-it-works). A locally-installed/on-premises Snaplex (also known as a "Groundplex" or "JCC") will be used throughout this guide to develop, execute and debug a Snap.

## Downloads

A Snaplex may be installed on:

* Linux
* Microsoft Windows
* Mac OS X (for development only)

<aside class="notice">
The current list of Snaplex downloads is maintained at <a href="http://doc.snaplogic.com/snaplex-downloads">http://doc.snaplogic.com/snaplex-downloads</a>.
</aside>

## Installation

### Linux & Microsoft Windows 

Please visit the official SnapLogic documentation for detailed instructions for installing a Snaplex on [Linux](http://doc.snaplogic.com/snaplex-installation-on-linux) or [Microsoft Windows](http://doc.snaplogic.com/installing-snaplex-on-windows). 

### Mac OS X

<aside class="warning">
Use of a Snaplex in a Mac OS X environment is supported only for developing Snaps.
</aside>

```shell
$ cd ~
$ wget https://s3.amazonaws.com/elastic.snaplogic.com/snaplogic-sidekick...rpm
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
$ brew install rpm2cpio
$ rpm2cpio.pl snaplogic-sidekick...rpm | cpio -idvm
$ echo 'export SL_ROOT=/opt/snaplogic' >> ~/.bash_profile
$ source ~/.bash_profile
```

> If successful, the output of the `cpio` command will list the Snaplex files populating the newly created `~/opt/snaplogic` directory, for example:

```shell
$ rpm2cpio.pl snaplogic-sidekick-4.mrc244-x86_64.rpm | cpio -idvm
./opt/snaplogic/bin/functions
./opt/snaplogic/bin/jcc.sh
./opt/snaplogic/bin/logstash.sh
./opt/snaplogic/etc/global.properties
./opt/snaplogic/etc/keys.properties
./opt/snaplogic/etc/logstash.conf
./opt/snaplogic/ldlib/libsapjco3.jnilib
./opt/snaplogic/ldlib/libsapjco3.so
./opt/snaplogic/pkgs/jre1.8.0_45/COPYRIGHT
...
./opt/snaplogic/run/lib/jcc.war
./opt/snaplogic/run/lib/logstash.jar
./opt/snaplogic/run/log
1044673 blocks
```

1. `cd` to your home directory
1. Download the latest RHEL/CentOS/Amazon RPM file from the [Snaplex downloads page](http://doc.snaplogic.com/snaplex-downloads).
1. Download and install [Homebrew](http://brew.sh/)
1. Install the `rpm2cpio` library
1. Extract the downloaded Snaplex `.rpm` file
1. Export the `SL_ROOT` environment variable to your `~/.bash_profile`

## Configuration

```shell
$ cat ~/opt/snaplogic/etc/keys.properties
cc.admin_uri = https://elastic.snaplogic.com:443
cc.username = rhowlett@snaplogic.com
cc.api_key = JDAFBDAD....9dAg43
cc.ssl_verify = True
```

For approved customers, SnapLogic Support will provide the necessary instructions for authentication and configuration of the Snaplex.

```shell
$ cat ~/opt/snaplogic/etc/global.properties
jcc.location = sidekick
jcc.sldb_uri = https://elastic.snaplogic.com:443
...
# The subscriber ID assigned for the customer. This will be available after
# the customer account is created.
jcc.subscriber_id = snaplogic

# This is the runtime environment in which the Snaplex should be started.
jcc.environment = devsnaplex
```

The `~/opt/snaplogic/etc/keys.properties` file contains the information required for authenticating to the SnapLogic Elastic Integration Platform APIs to run the Snaplex and deploy the developed Snaps.

The `~/opt/snaplogic/etc/global.properties` file contains the information required for general configuration of the Snaplex.

## Running

```shell
$ cd ~/opt/snaplogic/run/lib
$ java -jar jcc.war jcc
```

> If successful, the output of the `java` command will be similar to this:

```shell
...
	+- file:/Users/snapdev/opt/snaplogic/run/lib/jcc/zookeeper-3.4.6.jar
	+- sun.misc.Launcher$AppClassLoader@42a57993
		+- file:/Users/snapdev/opt/snaplogic/run/lib/jcc.war
		+- sun.misc.Launcher$ExtClassLoader@14ae5a5
2016-07-01T00:59:04,505 I main	[  ] Server listening for incoming connections
```
The simplest way to start the Snaplex is to run the packaged `jcc.war` file.

To check if the Snaplex started correctly, log in to the [SnapLogic Dashboard](https://elastic.snaplogic.com/sl/dashboard.html):

![SnapLogic Dashboard Snaplex Health](http://dl.dropboxusercontent.com/u/3519578/Screenshots/1Z0t.png)

<aside class="success">
A green check mark beside the Snaplex entry will indicate a healthy instance.
</aside>

# Snap Anatomy 101

```java
import com.snaplogic.api.ConfigurationException;
import com.snaplogic.api.ExecutionException;
import com.snaplogic.api.Snap;
import com.snaplogic.common.properties.builders.PropertyBuilder;
import com.snaplogic.snap.api.PropertyValues;
import com.snaplogic.snap.api.SnapCategory;
import com.snaplogic.snap.api.capabilities.Category;
import com.snaplogic.snap.api.capabilities.General;
import com.snaplogic.snap.api.capabilities.Version;
import com.snaplogic.snap.api.capabilities.Inputs;
import com.snaplogic.snap.api.capabilities.Outputs;
import com.snaplogic.snap.api.capabilities.Errors;
import com.snaplogic.snap.api.capabilities.ViewType;

@General(title = "Snap Name", purpose = "Description", 
		author = "Company Name", docLink = "http://www.docs.com/mysnap")
@Inputs(min = 0, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class MySnap implements Snap {

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
    }

    @Override
    public void execute() throws ExecutionException {
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

At its most basic, a Snap class declares the required name, view, version and category metadata annotations and implements the `Snap` interface.

### Metadata annotations

* `@General`: name, description, author, and documentation link for the Snap.
* `@Inputs`: specifies the minimum and maximum number of Input Views and the type of input they accept (JSON Documents or Binary data).
* `@Outputs`: specifies the minimum and maximum number of Output Views and the type of output they produce (JSON Documents or Binary data).
* `@Errors`: specifies whether an Error Views should be written to
* `@Version`: version number for the Snap (not to be confused with the Snap Pack version).
* `@Category`: [categorizes the Snap](#snap-categories) and determines the icon/color of the Snap within the SnapLogic Designer.

### Snap interface

The `com.snaplogic.api.Snap` interface that should be implemented by the Snap developers to convert their business logic into an entity that can be used inside SnapLogic Pipelines.
 
**`defineProperties`**

Defines the Snap properties using the given `PropertyBuilder`. This method is called during [the `compile` phase](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Build_Lifecycle_Basics) and generates the `settings` Snap schema property that the SnapLogic Designer uses to build the user interface.

**`configure`**

Configures the Snap with the `PropertyValues` provided by the user's input to the Snap Settings. Input validation should be done here - if there is an issue with the provided input, a `ConfigurationException` should be thrown.

**`execute`**

Executes the Snap's business logic. Should an error be encountered, an `ExecutionException` may be thrown.

**`cleanup`**

Cleans up any suitable resources after the Snap execution. Again, should an error be encountered, an `ExecutionException` may be thrown.

<aside class="notice">
Both <code>ConfigurationException</code> and <code>ExecutionException</code> are implementations of the root <code>SnapException</code> interface.

See the <a href="#exceptions-and-error-views">Exceptions and Error Views</a> section for more information.
</aside>

# Getting Started

```shell
$ mkdir ~/opt/snaplogic-dev
$ echo 'export SNAP_HOME=~/opt/snaplogic-dev' >> ~/.bash_profile
$ echo 'export JCC_DEBUG_PORT=9000' >> ~/.bash_profile
$ source ~/.bash_profile
```

If a healthy Snaplex is running locally, Snap Development may begin. 

Make a directory where Snap Development will occur, export the location as the `SNAP_HOME` environment variable and set the port to use for debugging.

## Debugging

```shell
$ cd ~/opt/snaplogic/run/lib
$ java -agentlib:jdwp=transport=dt_socket,server=y,address=${JCC_DEBUG_PORT},suspend=n -jar jcc.war jcc
```

> This will wait to launch the JCC until you have attached the debugger.

The Snaplex/JCC supports debugging a Snap's execution with your chosen IDE. Simply launch the JCC with the additional `agentlib` parameter with shown value, and attach to it using your IDE's remote debugging capability.

## Snap Maven Archetype

```shell
$ cd $SNAP_HOME
$ mvn archetype:generate -DarchetypeCatalog=http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] >>> maven-archetype-plugin:2.4:generate (default-cli) > generate-sources @ standalone-pom >>>
[INFO] 
[INFO] <<< maven-archetype-plugin:2.4:generate (default-cli) < generate-sources @ standalone-pom <<<
[INFO] 
[INFO] --- maven-archetype-plugin:2.4:generate (default-cli) @ standalone-pom ---
[INFO] Generating project in Interactive mode
[INFO] No archetype defined. Using maven-archetype-quickstart (org.apache.maven.archetypes:maven-archetype-quickstart:1.0)
Choose archetype:
1: http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/ -> com.snaplogic.tools:SnapArchetype (This is a sample snap project.)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 1
Choose com.snaplogic.tools:SnapArchetype version: 
1: 1.0
2: 1.1
3: 1.2
4: 1.3
5: 1.4
6: 1.5
Choose a number: 6: 
Define value for property 'groupId': : com.snaplogic.snaps
Define value for property 'artifactId': : demosnappack
Define value for property 'version':  1.0-SNAPSHOT: : 
Define value for property 'package':  com.snaplogic.snaps: : 
Define value for property 'organization': : snaplogic
Define value for property 'assetPath':  /snaplogic/shared: : /snaplogic/robin/shared
Define value for property 'snapPack':  demosnappack: : Demo Snap Pack
Define value for property 'user': : rhowlett@snaplogic.com
Confirm properties configuration:
groupId: com.snaplogic.snaps
artifactId: demosnappack
version: 1.0-SNAPSHOT
package: com.snaplogic.snaps
organization: snaplogic
assetPath: /snaplogic/robin/shared
snapPack: Demo Snap Pack
user: rhowlett@snaplogic.com
 Y: : Y
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Archetype: SnapArchetype:1.5
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: groupId, Value: com.snaplogic.snaps
[INFO] Parameter: artifactId, Value: demosnappack
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.snaplogic.snaps
[INFO] Parameter: packageInPathFormat, Value: com/snaplogic/snaps
[INFO] Parameter: package, Value: com.snaplogic.snaps
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: user, Value: rhowlett@snaplogic.com
[INFO] Parameter: organization, Value: snaplogic
[INFO] Parameter: assetPath, Value: /snaplogic/robin/shared
[INFO] Parameter: groupId, Value: com.snaplogic.snaps
[INFO] Parameter: snapPack, Value: Demo Snap Pack
[INFO] Parameter: artifactId, Value: demosnappack
[INFO] project created from Archetype in dir: /Users/snapdev/opt/snaplogic-dev/demosnappack
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

A [Maven Archetype](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html) is a Maven project templating toolkit. SnapLogic provides `SnapArchetype` for quickly starting Snap development.

<aside class="notice">
The Snap archetype catalog is available at <a href="http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/">http://maven.clouddev.snaplogic.com:8080/nexus/content/repositories/releases/</a>
</aside>

`SnapArchetype` ships with eight sample Snaps for demonstration purposes:

Sample | Description
--------- | -----------
**Single Doc Generator** | *Outputs a single document*
**Doc Consumer** | *Consumes documents*
**Doc Generator** | *Outputs `n` documents, where `n` is specified by the user as an expression*
**Suggest** | *Demonstrates suggest setting capabilities*
**Property Types** | *Shows creating a variety of different Snap UI settings*
**Two Inputs** | *Consuming from two different input sources*
**Two Inputs Two Outputs** | *Consuming from two different input sources, and outputting two different Documents*
**Schema Example** | *Demonstrating input and output schemas for IO validation*

<aside class="warning">
If you choose a <code>groupId</code>/<code>package</code> value other than <code>com.snaplogic.snaps</code>, you must provide a HTTP URL value to the <code>docLink</code> parameter of a Snap's <code>@General</code> annotation on a Snap.
</aside>

Once `SnapArchetype` has been generated, it can be imported as a Maven project into your IDE ([IntelliJ IDEA](https://www.jetbrains.com/help/idea/2016.1/importing-project-from-maven-model.html), [Eclipse](https://books.sonatype.com/m2eclipse-book/reference/creating-sect-importing-projects.html), [Netbeans](http://wiki.netbeans.org/MavenBestPractices)).

## Project Structure

> ![Project Structure](https://dl.dropboxusercontent.com/u/3519578/Screenshots/uzoL.png)

A Snap Project's structure follows [Maven's Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html).

### `src/main/assembly`

`final.xml` and `snap.xml` are used by the `maven-assembly-plugin` to build the Snap Pack zip file that can be uploaded to the SnapLogic Manager. You should not need to alter these files.

### `src/main/config`

`directives` contains basic Snap Pack metadata. 

The `NAME` property is the label of Snap Pack. This is visible when the SnapLogic Designer is grouping by Snap Pack e.g. the Snaps related to SQL Server are grouped under "SQL Server".

![SQL Server Snap Pack label](https://dl.dropboxusercontent.com/u/3519578/Screenshots/AUH9.png)

The `ASSET_DIR_PATH` property instructs the Snap Pack Installer to deploy the Snap Pack to the specified asset path.

A customer will be deploying to an Organization root, Project, or Project Spaces asset path e.g. `/snaplogic/shared`, `/snaplogic/projects/rhowlett`, and `/snaplogic/robin/shared` respectively (where "snaplogic" is the Organization name assigned by SnapLogic to your account, "rhowlett" is the Project folder name, and "robin" is the Project Spaces. 

This value can be changed dynamically at build time using the `-Dasset_path` Maven parameter (especially useful with a continuous integration tool, or if your company has multiple Organizations or wishes to deploy to multiple asset path locations).

### `src/main/java` and `src/main/resources`

Files and folders within these directories will be on the classpath, as per Maven convention. This is where you will write your Snap code.

### `src/test/java` and `src/test/resources`

Files and folders within these directories will be on the test classpath, as per Maven convention. Unit and integration tests (and supporting resources) are located in these directories.

### `pom.xml`

This is the Maven POM file. You will change this file when registering new Snaps and Accounts, when new dependencies or plugins are required, or when needing to modify the build properties.

# Developing Snaps

```xml
<!-- This identifies the classes which represent the actual Snaps
	(and become accessible on the Snaplex/JCC after deployment).
	
	After creating a new Snap class, add it here.
-->
<snap.classes>
	com.snaplogic.snaps.DocConsumer,
	com.snaplogic.snaps.DocGenerator,
	com.snaplogic.snaps.PropertyTypes,
	com.snaplogic.snaps.SchemaExample,
	com.snaplogic.snaps.SingleDocGenerator,
	com.snaplogic.snaps.Suggest,
	com.snaplogic.snaps.TwoInputs,
	com.snaplogic.snaps.TwoInputsTwoOutputs
</snap.classes>
<!-- After creating a new Account class, add it here -->
<account.classes>
</account.classes>
```

> pom.xml

SnapLogic provides the **JSDK**, a library of Java APIs to assist with building Snaps. 

The [Snap Maven Archetype](#snap-maven-archetype) ships with sample Snaps that we can use to highlight the JSDK's capabilities.

## Basic Snap Implementation

```java
// Set the Snap title, description etc
// Also, use the "docLink" parameter to set a link to the documentation
@General(title = "Single Doc Generator", purpose = "Generates one document and completes",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
// There is no input view for this Snap
@Inputs(min = 0, max = 0)
// This Snap has exactly one document output view
// Snaps can also have binary output i.e., offers={ViewType.BINARY}
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
// This Snap has an optional document error view
@Errors(min = 0, max = 1, offers = {ViewType.DOCUMENT})
// Version number of the Snap
@Version(snap = 1)
// This Snap belongs to SnapCategory.READ as is does not require input.
@Category(snap = SnapCategory.READ)
public class SingleDocGenerator implements Snap {
    private static final Logger log = LoggerFactory.getLogger(SingleDocGenerator.class);
    
    int counter = 0;
    
    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues)
            throws ConfigurationException {
    }

    @Override
    public void execute() throws ExecutionException {
        counter++;
        log.debug("counter=" + counter);

        /*
         * Write a single document to outputView. The next Snap in the pipe will
         * only ever see a single document from this Snap.
         */
        Map<String, String> data = new LinkedHashMap<String, String>() {{
            put("key", "value");
        }};
        outputViews.write(documentUtility.newDocument(data));
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

`SingleDocGenerator` is one of the sample Snaps included in the Snap Archetype. 

It is an uncomplicated Snap that generates a single document, without requiring any input data or user input. It implements the `Snap` interface.

![SingleDocGeneratorPipeline](https://dl.dropboxusercontent.com/u/3519578/Screenshots/o6o1.png)

[Snap Anatomy 101](#snap-anatomy-101) explains the purpose of each annotation and the overridden methods. The comments above each annotation describe their specific usage for this Snap.

<aside class="notice">
For simplicity, the <strong>"Single Doc Generator"</strong> Snap does not permit any input views. However, for general Snap development, there should <strong>almost always</strong> be an input view. See <a href="#reading-documents-from-an-input-view">Reading Documents from an Input View</a> for more.
</aside>

### Snap Lifecycle

At build time, `defineProperties` is called to generate the Snap Schema. This will construct the Snap's settings tab.

At runtime, the Snap is initialized by the Snaplex/JCC. Next, [Guice](https://github.com/google/guice) injects the various declared dependency instances (`documentUtility` and `outputViews` in this Snap).

The Snaplex's runtime then instructs the Snap's `configure` method to be called to validate and/or operate on any provided `PropertyValues` created from a user's interaction with the Snap's settings.

Next, the `execute` method is called. This is where the majority of the Snap's business logic will initiate.

Finally, the `cleanup` method is called to clean-up any resources used.

Due to no user input being required, nor clean-up needed, the `defineProperties`, `configure`, and `cleanup` methods have no implementation for this Snap. 

The `execute` method increments an `int` member variable, logs a debug statement, creates data, and generates a new Document with this data to be subsequently written to the output view.

## Logging

```shell
$ brew install lnav
$ lnav -i https://snaplogic.box.com/v/lnav-log-format
$ lnav ~/opt/snaplogic/run/log/jcc.json

2016-07-13T13:10:35.072 DEBUG [Single Doc Generator[5d64230b-fa22-4696-b736-05b599b06ba9 -- 9a3a88ef-d5b1-4715-945d-0c6a595f7940]] (SingleDocGenerator.java:77) - counter=1                                                                                      │
  snrd: 9a3a88ef-d5b1-4715-945d-0c6a595f7940                                                                                                                                                                                                                     │
  com.snaplogic.logging.id: 57868ceb64cabf28873d20a7.5d64230b-fa22-4696-b736-05b599b06ba9.9a3a88ef-d5b1-4715-945d-0c6a595f7940                                                                                                                                   │
  plrd: 5d64230b-fa22-4696-b736-05b599b06ba9                                                                                                                                                                                                                     │
  snlb: Single+Doc+Generator                                                                                                                                                                                                                                     │
  ccid: 57868ceb64cabf28873d20a7                                                                                                                                                                                                                                 │
  snii: 898c5459-0419-455c-9493-a0d6fbae9c8a                                                                                                                                                                                                                     │
  xid: a83bcfdd-2bc7-480e-bf1e-5fbdfef3c726   
```

In [the "Single Doc Generator" sample Snap](#basic-snap-implementation) above, an SLF4J `Logger` static class variable is declared and initialized and is used to write debug messages to the Snaplex/JCC's log.

For example, when previewing or executing the "Single Doc Generator" Snap in the Designer, the value of the `counter` variable is written to the log. 

This entry can then be viewed in the Snaplex's `jcc.json` log file (we recommend using [LNAV, the log file navigator](http://www.snaplogic.com/blog/lnav/) created by SnapLogic's own Timothy Stack):

## Creating Documents with DocumentUtility

```java
package com.snaplogic.snap.api;

public interface DocumentUtility {
	...
	/**
	 * Returns a new document with the given data for the given incoming document.
	 *
	 * @param incomingDocument
	 * @param data
	 * @return document
	 */
	Document newDocumentFor(Document incomingDocument, Object data);

	/**
	 * Returns a new document object with the given data and merges the original data into the
	 * new returned document.
	 * This allows to "pass through" the data from an incoming document and merge it into the new
	 * document being written to the output. The new document will have an {@literal original}
	 * key holding the original document to avoid data being overwritten when merging the data
	 * with original data
	 *
	 * @param document     as the document being written
	 * @param originalData as original data being merged with the document.
	 *
	 * @return document
	 * @throws UnsupportedOperationException only data and originalData of type Map is currently
	 *                                       supported
	 */
	Document newDocument(Document document, Object originalData);
	...
}
```

Guice's `@Inject` annotation will instruct the Snaplex/JCC to inject an instance of `DocumentUtility` and assign it to `documentUtility` ([see "Single Doc Generator" Snap above](#basic-snap-implementation)).

Documents may be created with given data. 

<aside class="success">
Data should be created either as a single <code>LinkedHashMap</code> instance (to preserve field ordering) or a <code>List</code> of <code>LinkedHashMap</code> instances. Avoid data being just a primitive data type or <code>null</code>.
</aside>

### Lineage

For Snaps that have input views and process documents, you should preserve [data lineage](https://en.wikipedia.org/wiki/Data_lineage) (required by [Ultra Pipelines](https://www.snaplogic.com/ultra-pipelines)) between the new output document to be written and the incoming original document by using one of the highlighted `DocumentUtility` methods:

*Establishes lineage between the new document and the incoming original document, but discards the incoming data:*<br />
`Document newDocumentWithLineage = documentUtility.newDocumentFor(originalDocument, data);`

*Establishes lineage between the new document and the incoming original document and "passes through" the incoming document's data by adding a key named "original" (containing the incoming original document's data) to the new document:*<br />
`Document seed = documentUtility.newDocument(newData);`<br />
`Document newDocumentWithLineage = documentUtility.newDocument(seed, originalDocument);`

<aside class="success">
We recommend that you do pass through the original document for greatest flexibility.
</aside>

## Writing Documents to an Output View

```java
package com.snaplogic.snap.api;

/**
 * OutBoundViews is the base interface for all the output views - {@link OutputView} and
 * {@link com.snaplogic.snap.view.ErrorView}
 */
public interface OutBoundViews<V extends OutputView> extends Views<V> {
	...
	/**
	 * Writes the given document to all the outbound views of the Snap.
	 *
	 * @param document
	 */
	void write(Document document);

	/**
	 * Writes a new document object with the given data and merges the original data into the
	 * new returned document.
	 * This allows to "pass through" the data from an incoming document and merge it into the new
	 * document being written to the output. The new document will have an {@literal original}
	 * key holding the original document to avoid data being overwritten when merging the data
	 * with original data.
	 *
	 * @param document     as the document being written
	 * @param originalData as original data being merged with the document.
	 *
	 * @return document
	 * @throws UnsupportedOperationException only data and originalData of type Map is currently
	 *                                       supported
	 */
	void write(Document document, Object originalData);
	...
}
```

Guice's `@Inject` annotation will instruct the Snaplex/JCC to inject an instance of `OutputViews` and assign it to `outputViews` ([see "Single Doc Generator" Snap above](#basic-snap-implementation)).

### Lineage

Similarly to [DocumentUtility](#creating-documents-with-documentutility), output views can establish lineage between incoming and outgoing documents. This is useful when an outgoing Document has been created without lineage but the incoming original document is still accessible:

`outputViews.write(documentWithoutLineage, originalDocument);`

## Unit Testing Snaps with SnapTestRunner

```java
import com.snaplogic.snap.test.harness.OutputRecorder;
import com.snaplogic.snap.test.harness.SnapTestRunner;
import com.snaplogic.snap.test.harness.TestFixture;
import com.snaplogic.snap.test.harness.TestResult;

/**
 * Tests that the {@link SingleDocGenerator} Snap sent one Document to the output view.
 */
@RunWith(SnapTestRunner.class)
public class SingleDocGeneratorTest {

    @TestFixture(snap = SingleDocGenerator.class,
            outputs = "output0")
    public void testSingleDocGeneratorFunctionality(TestResult testResult)
            throws Exception {
        assertNull(testResult.getException());
        OutputRecorder outputRecorder = testResult.getOutputViewByName("output0");
        assertEquals(1, outputRecorder.getRecordedData().size());
    }
}
```

The `SingleDocGeneratorTest` class outlines basic usage of **jtest**, a JUnit-based Snap unit testing framework.

jtest provides `SnapTestRunner`, a custom JUnit test runner which simulates the targeted Snap being executed within a pipeline. 

Test methods are annotated with the `@TestFixture` annotation, which specifies what Snap implementation to use, input/output/error views configured, property settings specified, accounts associated etc.

The test method should take a single `TestResult` argument, which can be used within the test to determine whether the correct output was written (using an `OutputRecorder`), if errors occurred etc.

In the `SingleDocGeneratorTest` sample provided, we can see that the test method focuses on examining that the `TestResult` did not receive any thrown exceptions, and that the declared output view registered the expected number of documents processed.

The ["jtest: Snap Unit Testing Framework"](#jtest-snap-unit-testing-framework) section contains much more information about testing Snaps, including working towards a purely declarative style of Snap unit testing using just the test method's `@TestFixture` annotation.

## Reading Documents from an Input View

```java
@General(title = "Doc Consumer", purpose = "Consumes the incoming documents",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 1, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 0, max = 0)
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.WRITE)
public class DocConsumer implements Snap {
    private static final Logger log = LoggerFactory.getLogger(DocConsumer.class);
    private AtomicInteger count = new AtomicInteger(0);

    @Inject
    private InputViews inputViews;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
    }

    @Override
    public void execute() throws ExecutionException {
        InputView inputView = inputViews.get();
        Iterator<Document> documentIterator = inputViews.getDocumentsFrom(inputView);
        while (documentIterator.hasNext()) {
            Document doc = documentIterator.next();
            log.debug("Received Document {}", doc.toString());
            count.getAndIncrement();
        }
    }

    @Override
    public void cleanup() throws ExecutionException {
        log.debug("Consumed {} documents", count.get());
    }
}
```

The `DocConsumer` sample Snap that ships with the [Snap Archetype](#snap-maven-archetype) demonstrates how to read from a Snap's input view.

Similarly to [Writing Documents to an Output View](#writing-documents-to-an-output-view), Guice will inject an instance of `InputViews` and assign it to the `inputViews` variable.

`inputViews.get()` returns the default view associated with the Snap. If the Snap has more than one view, a `ViewException` is thrown.

An `Iterator` of `Document` objects can then be acquired by asking the `InputViews` instance for the `Document` objects associated with the specified `InputView` instance.

Iterating over the `Document` objects, the Snap logs that a document was consumed and increments its `AtomicInteger` counter.

The `cleanup()` method, which is called after the Snap has completed its execution phase, logs the total number of documents consumed.

![Consuming 3 Documents](http://dl.dropboxusercontent.com/u/3519578/Screenshots/KZdv.png)

<aside class="notice">
SnapLogic's Platform will enforce that all input documents have been fully consumed, otherwise an error will occur and the pipeline will fail.
<br />
<br />If some documents are consumed but "dropped on the floor" (not written to either an output or error view, or used to create another <code>Document</code>), call <code>document.acknowledge</code> to indicate it has been consumed.
</aside>

## Snap Categories

The [Doc Consumer Snap's](#reading-documents-from-an-input-view) category (`SnapCategory.WRITE`) differs from [SingleDocGenerator's](#basic-snap-implementation) `SnapCategory.READ` category. 

This difference between category types manifests in three ways:

* the color and icon of the Snaps are different,
* the "Doc Consumer" Snap inherits an "Execute during Preview" setting to toggle whether the Snap will execute when the Pipeline is validating/previewing,
* the Snaps are separated when "Group By Type" is chosen.

![Snap Category Difference](http://dl.dropboxusercontent.com/u/3519578/Screenshots/hEPI.png)

Icon | Snap Category | Description
----------- | ----------- | -----------
<img src="http://dl.dropboxusercontent.com/u/3519578/Screenshots/xzQs.png" /> | **READ** | Sources of data in the pipeline.<br />*Example: File Reader*
<img src="http://dl.dropboxusercontent.com/u/3519578/Screenshots/QJ63.png" /> | **WRITE** | Data destinations or sinks in the pipeline. They also inherit an "Execute during Preview" setting.<br />*Example: File Writer* 
<img src="http://dl.dropboxusercontent.com/u/3519578/Screenshots/HmYs.png" /> | **PARSE** |	Parse unstructured input data into structured output data.<br />*Example: CSV Parser*
<img src="http://dl.dropboxusercontent.com/u/3519578/Screenshots/LNZC.png" /> | **FORMAT** | Change data format.<br />*Example: CSV Formatter*
<img src="http://dl.dropboxusercontent.com/u/3519578/Screenshots/oETU.png" /> | **TRANSFORM** | Modify data significantly.<br />*Example: Aggregate, Join*
<img src="http://dl.dropboxusercontent.com/u/3519578/Screenshots/NQNT.png" /> | **FLOW** | Change the direction or output of data in the pipeline.<br />*Example: Filter, Router*

As you develop Snaps, choose the appropriate Snap Category for your use cases.

## Bootstrapping with SimpleSnap

```java
/**
 * This Snap accepts two inputs and outputs to a single output. To use it, feed
 * it at least one document in each view. It will output a single stream which consists
 * of the combination of the two inputs.
 * 
 * This Snap extends {@link SimpleSnap} (instead of implementing {@link Snap}).
 * This means that instead of having a method called 'execute' which is called once,
 * it has a method called 'process' which is called for every document the Snap receives.
 * This means you do not have to iterate over incoming documents as 'process' does that
 * for you.
 */
@General(title = "Two Inputs", purpose = "Accepts two inputs, merges them",
        author = "Your Company Name" docLink = "http://yourdocslinkhere.com")
@Inputs(min = 2, max = 2)
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 0, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class TwoInputs extends SimpleSnap {

    private static final Logger log = LoggerFactory.getLogger(TwoInputs.class);
    private int count = 0;
    @Inject
    private OutputViews outputViews;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues)
            throws ConfigurationException {
    }

    @Override
    public void process(Document document, String inputViewName) {
        // Demonstrates how to create a new copy of the document
        Document newdoc = document.copy();

        // get a map representation of the document
        // NOTE: we are *not* duplicating the data.
        // We will *not* have to convert the map back to a doc.
        // It's just a pointer representation.
        Map<String, Object> data = newdoc.get(Map.class);

        // assign a value to a field of the map
        // In this case, we are just assigning 'processed=True'
        // to signal that the document has been processed.
        data.put("processed", "True");

        count++;
        // log current document number
        log.debug("count=" + count);

        // log current document
        log.debug("document: " + newdoc.toString());

        // send new document to next Snap
        outputViews.write(newdoc, document);
    }

    @Override
    public void cleanup() throws ExecutionException {
        // Log final number of documents processed
        log.debug("Final count=" + count);
    }
}
```

Now that we understand the basics of a Snap, we can extend the `SimpleSnap` class to have some of the heavy lifting done for us.

`SimpleSnap` provides all the necessary hooks for writing a new Snap. A Snap author can write a new Snap by simply extending `SimpleSnap`, adding the desired metadata annotations (`SimpleSnap` already declares an `@Errors` view that can be overridden), and providing the implementation for the `process` method (and optionally, for the `defineProperties` and `configure` methods too).

`SimpleSnap` takes care of iterating over the incoming documents (if any) and allowing you to process each document at a time. It also handles the dependency injection for the `Input/Output/ErrorViews`, the `DocumentUtility`, and the `ExecutionUtil` (which handles iterating over the documents in the incoming input view(s) on your behalf, and writing any thrown `SnapDataException` instances to the error view).

![Two Inputs](https://dl.dropboxusercontent.com/u/3519578/Screenshots/IVTO.png)

There are a number of other "Simple" Snap implementations that can help getting started quickly, including:

* `SimpleReadSnap`
* `SimpleWriteSnap`
* `SimpleBinarySnap`
* `SimpleBinaryReadSnap`
* `SimpleTransformSnap`
* `SimpleBinaryWriteSnap`

## Writing Binary Data to an Output View

```java
package com.snaplogic.snaps;
...
@General(title = "Character Counter", purpose = "Demo writing to Binary Output View",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Outputs(min = 1, max = 1, offers = ViewType.BINARY)
@Errors(min = 1, max = 1, offers = ViewType.DOCUMENT)
@Version(snap = 1)
public class CharacterCounter extends SimpleBinaryWriteSnap {

    private static final String UTF_8 = "UTF-8";
	...
    @Override
    protected void process(final Document header, final ReadableByteChannel readChannel) {
        outputViews.write(new BinaryOutput() {
            @Override
            public Document getHeader() {
            	// maintain the incoming header
                return header;
            }

            @Override
            public void write(WritableByteChannel writeChannel) throws IOException {
                try (InputStream inputStream = Channels.newInputStream(readChannel)) {
                    OutputStream outputStream = Channels.newOutputStream(writeChannel);

                    // Guava Multiset
                    Multiset<Character> bagOfChars = HashMultiset.create();

                    Reader reader = new InputStreamReader(
                            new BufferedInputStream(inputStream), UTF_8);

                    // read in each character
                    int characterRead;
                    while ((characterRead = reader.read()) != -1) {
                        // add the lowercase version of the character to the Multiset
                        bagOfChars.add((char) Character.toLowerCase(characterRead));
                    }

                    try {
                        // for each letter of English alphabet, write a line with the number of times
                        // it appeared in the input data
                        for (char letter = 'a'; letter <= 'z'; letter++) {
                            String countForLetter = letter + ":" + bagOfChars.count(letter) + System.lineSeparator();
                            IOUtils.write(countForLetter, outputStream, UTF_8);
                        }
                    } finally {
                        IOUtils.closeQuietly(outputStream);
                    }
                }
            }
        });
    }
}
```

While JSON-based Documents offer the greatest flexibility of manipulating data within a pipeline, Binary data has advantages too. The costs associated with parsing and formatting data to JSON are avoided, and it can allow easier integration with processes running externally to the pipeline.

The `CharacterCounter` sample Snap shown demonstrates writing to a binary output view. It counts the number of times each letter of the English alphabet appears in the incoming binary data, and then writes the results to the binary output view.

In the sample, the `process()` method's implementation writes an anonymous `BinaryOutput` instance to the Snap's output view. 

The provided `ReadableByteChannel` argument is used to create an `InputStream`. The logic for counting each occurrence of each letter then follows before the results are written to an `OutputStream`, which is sent the `write()` method's `WritableByteChannel`, and then on to the Snap's binary output view:

![Character Counter Binary Output](https://dl.dropboxusercontent.com/u/3519578/Screenshots/xNKZ.png)

## Reading Binary Data from an Input View

```java
package com.snaplogic.snap.api.write;
...
@Inputs(min = 1, max = 1, accepts = {ViewType.BINARY})
@Outputs(min = 0, max = 0)
@Category(snap = SnapCategory.WRITE)
public abstract class SimpleBinaryWriteSnap extends SimpleBinarySnap {

    @Inject
    private InputViews inputViews;

	// called by SimpleBinarySnap.execute()
    @Override
    protected void doWork() {
        for (InputView inputView : inputViews) {
            for (BinaryInput binaryData : inputViews.binaryInputs(inputView)) {
                ReadableByteChannel readChannel = null;
                try {
                    readChannel = binaryData.getChannel();
                    process(binaryData.getHeader(), readChannel);
                } catch (IOException e) {
                    throw new ExecutionException(e,
                            EXCEPTION_WHILE_READING_FROM_BINARY_INPUT);
                } finally {
                    IOUtils.closeQuietly(readChannel);
                }
            }
        }
    }

    /**
     * Processes the incoming binary data.
     */
    protected abstract void process(Document header, ReadableByteChannel readChannel);
}
```

The `CharacterCounter` class above extends the abstract `SimpleBinaryWriteSnap` class which, similarly to [Bootstrapping with SimpleSnap](#bootstrapping-with-simplesnap), provides some sensible default values for metadata annotations and some initial implementation.

The `doWork()` method iterates over the Snap's binary input views, returning a `BinaryInput` instance for each view. The `BinaryInput` contains a `ReadableByteChannel` (which can read the bytes of the incoming binary data) and a Document header containing basic metadata (`content-type` etc.).

As shown [in the **"Character Counter"** sample above](#writing-binary-data-to-an-output-view), these can then be used to create an `InputStream` or simliar to access and/or manipulate the binary data.

### Headers

Binary Headers are Documents used to store useful metadata about the binary data stream.

When creating `BinaryOutput` instances, a Document header should be created and returned in `getHeader()`.

<aside class="success">
We recommend using standard/popular specification field names, lowercased, when possible e.g. <code>"content-type"</code>, from the HTTP specification.
</aside> 

## Accepting User Input with PropertyBuilder

```java
@General(title = "Doc Generator", purpose = "Generates documents based on the configuration",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 0, max = 0)
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 0, max = 0)
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class DocGenerator implements Snap {
    private static final String COUNT = "count";

    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;

    private int count;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(COUNT, "Number of documents to create")
                .type(SnapType.INTEGER).required().add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
        BigInteger countValue = propertyValues.get(COUNT);
        count = countValue.intValue();
    }

    @Override
    public void execute() throws ExecutionException {
        for (int i = 0; i < count; i++) {
            Map<String, String> data = new LinkedHashMap<>();
            data.put("key", "value" + (i + 1));
            outputViews.write(documentUtility.newDocument(data));
        }
    }

    @Override
    public void cleanup() throws ExecutionException {
        // NOOP
    }
}
```

The `DocGenerator` Snap is similar to the [`SingleDocGenerator`](#basic-snap-implementation) Snap, except the number of documents generated is configurable through user input:

![Doc Generator](http://dl.dropboxusercontent.com/u/3519578/Screenshots/lJqe.png)

<aside class="success">
All Snaps will have a required <strong>Label</strong> property for customizing the display label of the Snap within a Pipeline.
</aside>

To accept user input, the `defineProperties` method is used.

[As described earlier](#snap-anatomy-101), `defineProperties` is called when the Snap is being built. It constructs the Snap Schema which the Designer UI interprets and assembles into the Snap's Settings tab.

In `DocGenerator.defineProperties()`, a required `count` property of type `SnapType.INTEGER` is described and added to the provided `PropertyBuilder` instance.

Once the user has input their data and the Snap is executing, the `configure` method can access the data through the provided `PropertyValues` instance. As shown, this could be stored in an instance variable and used in the Snap's `execute()` method.

See the [PropertyBuilder Reference](#propertybuilder-reference) section for more examples.

## Validating User Input

The SnapLogic Platform will attempt to provide some basic type validation on user input:

![Auto validation](http://dl.dropboxusercontent.com/u/3519578/Screenshots/mi8k.png)

If more advanced validation is required, provide the implementation in the `configure` method and throw a `ConfigurationException` when invalid input is encountered. See [Exceptions and Error Views](#exceptions-and-error-views) for more.

## Expression-enabled Properties

```java
public class DocGenerator implements Snap {
    private static final String COUNT = "count";
    private static final Document NO_DOCUMENT = null;

    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;
    
	private int count;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(COUNT, "Number of documents to create")
                .type(SnapType.INTEGER)
                .required()
                .expression()
                .add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
        ExpressionProperty countValueExp = propertyValues.getAsExpression(COUNT);

    	// evaluate the expression directly (without the context of a Document)
		BigInteger countValue = countValueExp.eval(NO_DOCUMENT);
		
		count = countValue.intValue();
    }

    @Override
    public void execute() throws ExecutionException {
        for (int i = 0; i < count; i++) {
            Map<String, String> data = new LinkedHashMap<>();
            data.put("key", "value" + (i + 1));
            outputViews.write(documentUtility.newDocument(data));
        }
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

Snaplogic's [Expression Language](http://doc.snaplogic.com/expression-language) is an incredibly powerful tool available to Snaps. Using a JavaScript-like syntax, users have access to functions and properties to dynamically set property values.

Comparing to the [previous implementation](#accepting-user-input-with-propertybuilder) of `DocGenerator`, we can see that `.expression()` has been added to the `propertyBuilder()`. 

The `configure` method's `BigInteger countValue` local variable has been replaced by the `ExpressionProperty countValueExp` local variable, which is retrieved from the `PropertyValues` with the `getAsExpression` method, passing the field name "count" in. The `countValueExp` expression property is evaluated against `null` (as there is no incoming document to evaluate against e.g. when referencing a key in the incoming document), so the expression is evaluated directly. Since a number value was evaluated, we assign it to a local variable of type `BigInteger`, and then get its `intValue()`.

Within `execute()`, we can then use the `count` in the loop.

To demonstrate this new capability, we can use a "count" [Pipeline Property](http://doc.snaplogic.com/pipeline-properties) and then reference this in the Snap within a `parseInt` expression function:

![Doc Generator with Expression](https://dl.dropboxusercontent.com/u/3519578/Screenshots/Zin9.png)

## Suggesting Property Values

```java
/**
 * This Snap has one output and writes one document that contains the suggested
 * value. This Snap demonstrates the suggest value functionality that uses the
 * partial configuration information to suggest a property value.
 */
@General(title = "Suggest", purpose = "Demo suggest functionality.",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 0, max = 0)
@Outputs(min = 1, max = 1)
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class Suggest implements Snap {

    public static final String PROP_NAME = "name";
    public static final String PROP_ECHO = "echo";
    
    private String valueToWrite;
    
    @Inject
    private DocumentUtility documentUtility;
    @Inject
    private OutputViews outputViews;

    @Override
    public void defineProperties(final PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(PROP_NAME, PROP_NAME)
                .add();
        propertyBuilder.describe(PROP_ECHO, PROP_ECHO)
                .withSuggestions(new Suggestions() {
                    @Override
                    public void suggest(SuggestionBuilder suggestionBuilder,
                            PropertyValues propertyValues) {
                        String name = propertyValues.get(PropertyCategory.SETTINGS, PROP_NAME);
                        suggestionBuilder.node(PROP_ECHO).suggestions(name);
                    }
                }).add();
    }

    @Override
    public void configure(final PropertyValues propertyValues) throws ConfigurationException {
        valueToWrite = propertyValues.get(PROP_ECHO);
    }

    @Override
    public void execute() throws ExecutionException {
        Map<String, String> data = new LinkedHashMap<String, String>() {{
            put("key", valueToWrite);
        }};
        outputViews.write(documentUtility.newDocument(data));
    }

    @Override
    public void cleanup() throws ExecutionException {
    }
}
```

A Snap can make "suggestions" to the user about property values. This is useful to lookup appropriate values using either another data source or the input value of a different property.

In the "Suggest" sample Snap, the user enters in a value for the "name" property. The "echo" property's suggest action then reads that value and allows the user to select it.

![Suggest](https://dl.dropboxusercontent.com/u/3519578/Screenshots/e2S6.png)

The suggest capability is enabled through use of the the `PropertyBuilder.withSuggestions(Suggestions suggestions)` method. Implementations of the `Suggestions` interface should be provided to it.

<aside class="notice">
Notice that the UI also added the <strong>"="</strong> expression toggle button - suggestible properties <a href="#expression-enabled-properties">should always be expression-enabled too</a>. 

This was not done in the <code>Suggest</code> example above. The work required to support expressions for the field is left as an exercise to the reader.
</aside>

The `Suggestions` instance (often an anonymous instance) implements the single `suggest` method. It provides, as method arguments, references to the `SuggestionBuilder` and the `PropertyValues`. 

Using the `SuggestionBuilder` instance, we can configure simple String varargs for `suggestions()` for a selected `node()` that will result in a list of suggested values.

<aside class="success">
Alternatively, use <code>objectSuggestions()</code> to add a list of values, with each entry representing a suggestion. The entry must be a map which provides a <strong>"label"</strong> key.

The label will be displayed by the UI as the selectable entry, the rest of the object definition will be hidden.
</aside>

### Suggestions within Tables and Composites

To add suggestions to a Table element property, select the table node and use the `over()` builder method:

<div class="inline-code">
SnapProperty echo = propertyBuilder.describe(PROP_ECHO, PROP_ECHO)
	.type(SnapType.STRING)
	.withSuggestions(new Suggestions() {
		@Override
		public void suggest(SuggestionBuilder suggestionBuilder, 
				PropertyValues propertyValues) {
			suggestionBuilder
				.node("table")
				.over("echo")
				.suggestions("X", "Y");
		}
	}).build();

propertyBuilder.describe("table", "table")
	.type(SnapType.TABLE)
	.withEntry(echo)
	.add();
</div>

![Table Suggest](http://dl.dropboxusercontent.com/u/3519578/Screenshots/NlQw.png)

<aside class="warning">
Suggestions are not supported within Composite properties.
</aside>

## Input/Output View Schemas

```java
/**
 * A Pass-through Snap to provide schema on its input and output view.
 * If the incoming document matches the expected schema, document is written
 * to the output view. If the document does not match the schema, it is written
 * to the error view.
 */
@General(title = "View Schema", purpose = "Demonstrates view schema definition",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 1, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.TRANSFORM)
public class SchemaExample extends SimpleSnap implements InputSchemaProvider,
        OutputSchemaProvider {
    private static final String COL_A = "colA";
    private static final String COL_B = "colB";
    private static final String COL_C = "colC";
    private static final String INPUT_VIEW_NAME = "input0";
    private static final String OUTPUT_VIEW_NAME = "output0";

    @Override
    public void defineInputSchema(final SchemaProvider provider) {
        Schema colA = provider.createSchema(SnapType.STRING, COL_A);
        Schema colB = provider.createSchema(SnapType.STRING, COL_B);
        Schema colC = provider.createSchema(SnapType.STRING, COL_C);
        provider.getSchemaBuilder(INPUT_VIEW_NAME)
                .withChildSchema(colA)
                .withChildSchema(colB)
                .withChildSchema(colC)
                .build();
    }

    @Override
    public void defineOutputSchema(SchemaProvider provider) {
        Schema colA = provider.createSchema(SnapType.STRING, COL_A);
        Schema colB = provider.createSchema(SnapType.STRING, COL_B);
        Schema colC = provider.createSchema(SnapType.STRING, COL_C);
        provider.getSchemaBuilder(OUTPUT_VIEW_NAME)
                .withChildSchema(colA)
                .withChildSchema(colB)
                .withChildSchema(colC)
                .build();
    }

    @Override
    public void defineProperties(final PropertyBuilder propertyBuilder) {
    }

    @Override
    public void configure(PropertyValues propertyValues)
            throws ConfigurationException {
    }

    @Override
    protected void process(Document document, final String inputViewName) {
        // Validate incoming data against the schema
        validate(document);
        // Pass the document to the downstream Snap in the pipeline.
        outputViews.write(document);
    }

    private void validate(Document document) {
        Map<String, Object> content = document.get(Map.class);
        for (String key : new String[]{COL_A, COL_B, COL_C}) {
            if (!content.containsKey(key)) {
                // Throwing a SnapDataException instructs the SnapLogic platform
                // to forward this document to the error view with the
                // provided error message.
                throw new SnapDataException(document,
                        String.format("Data map does not contain key:%s", key));
            }
        }
    }
}
```

Snap View Schemas allow Snaps to declare the structure of the data they are expecting to consume and/or generate. 

When understanding how to use a Schema, you must first understand how declaring an input or output schema affects the Snaps in the pipeline immediately preceding and following, respectively.

A Snap's input view schema becomes the Target Schema of the Snap (e.g. Mapper) immediately preceding it in the pipeline. This allows the user the opportunity to shape the data to the desired form:

![Input Schema](http://dl.dropboxusercontent.com/u/3519578/Screenshots/M3WA.png)

In the `SchemaExample` sample (which creates the **"View Schema"** Snap), an input schema is declared by implementing the `InputSchemaProvider` interface. This interface has a `defineInputSchema` method, where the `SchemaProvider` argument can be used to create child schemas and add them to the provider through a `SchemaBuilder`.

Similarly the Snap's output view schema becomes the Input Schema of the Snap immediately following it in the pipeline, allowing users to anticipate the shape of the data exiting the Snap:

![Output Schema](http://dl.dropboxusercontent.com/u/3519578/Screenshots/gqxF.png)

### Enforcing a Schema

Defining the schema as above will aid users in understanding what should be the structure of the data entering and exiting the Snap. However, the Snap will not reject data that does not conform to the schema.

If you wish to be strict about enforcing a schema, then you may provide your own validation of the input and output data during `execute`/`process`.

In the `SchemaExample` sample above, the `validate` method is called before the data is written to the output view. It checks that the incoming document has the properties declared in the schema present (`colA`, `colB`, and `colC`).

If the input data contains those keys, the document is written to the output view:

![Valid Input](http://dl.dropboxusercontent.com/u/3519578/Screenshots/fiAa.png)

If the input data does not contain all of those keys, a `SnapDataException` is thrown and the document is written to the error view:

![Invalid Input](http://dl.dropboxusercontent.com/u/3519578/Screenshots/f4sZ.png)

## Exceptions and Error Views

```java
package com.snaplogic.api;

/**
 * This is the root exception for the jsdk module.
 *
 * All the other exceptions in jsdk module should derive from this exception. This exception
 * should not be used directly by the Snap author. Snap author should use
 * {@link com.snaplogic.api.ConfigurationException} or
 * {@link com.snaplogic.api.ExecutionException}
 */
public class SnapException extends RuntimeException {
    private static final long serialVersionUID = 1L;
    
    private String message;
    private String reason;
    private String resolution;

    private static Pattern STACK_TRACE_PATTERN = Pattern.compile("\n[ \t]*at[ \t]+");

    public SnapException(String message) {
        super();
        this.message = stripStackTrace(message);
    }

    public SnapException(String message, String reason) {
        this(message);
        this.reason = stripStackTrace(reason);
    }

    public SnapException(Throwable cause, String message) {
        this(message);
        super.initCause(cause);
    }

    public SnapException(Throwable cause, String message, String... params) {
        this(cause, stringFormat(message, params));
        this.reason = this.getMessage();
    }

    /**
     * Applies the given parameter values to the message template string.
     */
    public SnapException formatWith(Object... params) {
        String message = getMessage();
        this.message = stripStackTrace(stringFormat(message, params));
        return this;
    }
    ...
    /**
     * Sets the exception message for the user.
     */
    public final <T extends SnapException> T  withReason(final String reason) {
        this.reason = stripStackTrace(reason);
        return (T) this;
    }

    /**
     * Sets the resolution for fixing this exception.
     */
    public final <T extends SnapException> T withResolution(String resolution) {
        this.resolution = stripStackTrace(resolution);
        return (T) this;
    }

    public final <T extends SnapException> T  withResolutionAsDefect() {
        this.resolution = Messages.PLEASE_FILE_A_DEFECT_AGAINST_SNAP;
        return (T) this;
    }
    ...
```

`SnapException` is the root exception type and extends `RuntimeException`. However, you should instead use one of the following implementations instead:

Class | Where to Use | Description
--------- | ----------- | -----------
`ConfigurationException` | `configure()` | Throw when the Snap's settings are invalid.
`ExecutionException` | `execute()` | Throw for internal, non-data-related ex errors. 
`SnapDataException` | `execute()` | Throw for data-related errors.

<aside class="notice">
If your Snap extends <code>SimpleSnap</code>, thrown <code>SnapDataException</code>s will be automatically written to the error view.
</aside>

### Constructing Exceptions

At it's most basic, a `SnapException` subclass instance just requires a failure message to be communicated to the user:

<div class="inline-code">
@Override
public void execute() throws ExecutionException {
	throw new ExecutionException("Message");
}
</div>

This results in the following user experience:

![Basic Exception](https://dl.dropboxusercontent.com/u/3519578/Screenshots/eX3K.png)

When creating exceptions, we recommend providing a failure message, a reason, and a resolution:

Parameter | Description
--------- | -----------
Message | High level explanation of the issue (what was the Snap trying to do).
Reason | Detailed explanation of the issue (defaults to the root cause message).
Resolution | What can the user do to fix the issue

<div class="inline-code">
@Override
public void execute() throws ExecutionException {
	String deptName = "deptName";

	String message = "The \"%s\" field was missing.";
	String reason = "This Snap requires all fields to be present in the data source.";
	String resolution = String.format("Contact your DBA to ensure all model fields " +
			"are present in the data source.", deptName);

	throw new ExecutionException(message)
		.formatWith(deptName)
		.withReason(reason)
		.withResolution(resolution);
}		
</div>

![Reason and Resolution](https://dl.dropboxusercontent.com/u/3519578/Screenshots/FjIr.png)

### ConfigurationException

`ConfigurationException` instances should be thrown during `configure()`, when the pipeline is validating:

<div class="inline-code">
@Override
public void configure(PropertyValues propertyValues) throws ConfigurationException {
	throw new ConfigurationException("Error Message for User")
		.withReason("A good reason.")
		.withResolution("What to do next.");
}
</div>

![Config Exception](https://dl.dropboxusercontent.com/u/3519578/Screenshots/JAB9.png)

### Writing Exceptions to the Error View

Throwing a new `SnapDataException` and writing it to the error view along with the input document that generated the data error, will display both the exception stacktrace and the original data. For example:

<div class="inline-code">
@Override
public void execute() throws ExecutionException {
	try {
		throw new IllegalArgumentException("Illegal Argument.");
	} catch (IllegalArgumentException e) {
		SnapDataException snapDataException = new SnapDataException(e, "Error Message.")
			.withResolution("Provide the correct argument.");

		errorViews.write(snapDataException, document);
	}
}
</div>

will result in the following content written to the error view:

![Error View Exception](http://dl.dropboxusercontent.com/u/3519578/Screenshots/DmKE.png)

If your Snap extends `SimpleSnap`, it is even easier - just throw the `SnapDataException` and it will be written to the error view as above:

<div class="inline-code">
@Override
public void process(Document document, String inputViewName) throws ExecutionException {
	try {
		throw new IllegalArgumentException("Illegal Argument.");
	} catch (IllegalArgumentException e) {
		throw new SnapDataException(e, "Error Message.")
			.withResolution("Provide the correct argument.");
	}
}
</div>

<aside class="success">
SnapLogic recommends always enabling a Document error view:

<p><code>@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})</code></p>
</aside>

## Customizing Input/Output Views

```java
/**
 * This Snap demonstrates two inputs and two outputs.
 *
 * It will output two streams, one for only males and another for females. 
 * Unknowns will get sent to both.
 */
@General(title = "Two Ins/Outs", purpose = "Accepts two inputs, sends to two outputs.",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 2, max = 2)
@Outputs(min = 2, max = 2, offers = {ViewType.DOCUMENT})
@Errors(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
public class TwoInputsTwoOutputs extends SimpleSnap implements ViewProvider {
    private static final Logger log = LoggerFactory.getLogger(TwoInputsTwoOutputs.class);
    
    private final static String INPUT_0 = "input0";
    private final static String INPUT_1 = "input1";
    private final static String MALE_VIEW = "output_male";
    private final static String FEMALE_VIEW = "output_female";
    
    @Inject
    private OutputViews outputViews;
    
    @Inject
    private ErrorViews errorViews;

    @Override
    public void defineViews(final ViewBuilder viewBuilder) {
        viewBuilder.describe(INPUT_0)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.INPUT);
        viewBuilder.describe(INPUT_1)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.INPUT);

        viewBuilder.describe(MALE_VIEW)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.OUTPUT);
        viewBuilder.describe(FEMALE_VIEW)
                .type(ViewType.DOCUMENT)
                .add(ViewCategory.OUTPUT);
    }

    ...

    @Override
    public void process(Document document, String inputViewName) {
        Document newdoc = document.copy();
        Map<String, Object> data = newdoc.get(Map.class);

        // Add a new field "processed"
        data.put("processed", "True");

        // send males to male output view
        if (data.get("gender").equals("male")) {
            outputViews.getDocumentViewFor(MALE_VIEW).write(newdoc);
        } else if (data.get("gender").equals("female")) {
            // send females to female output view
            outputViews.getDocumentViewFor(FEMALE_VIEW).write(newdoc);
        } else {
            // send unknowns to error view
            errorViews.write(newdoc, document);
        }
    }
    
    ...
}
```

The `TwoInputsTwoOutputs` sample accepts documents from two input views and routes to a particular output view based on the value of the "gender" field:

![TwoInOut](http://dl.dropboxusercontent.com/u/3519578/Screenshots/oHpC.png)

### Reading from Multiple Input Views

As it extends `SimpleSnap`, the "Two Ins/Outs" Snap benefits from already knowing how to read from multiple input views. `ExecutionUtil` iterates over the available input views and send each document to the `process()` method, [as shown previously](#bootstrapping-with-simplesnap). If you can guarantee that each input view will have documents sent to it, this is an acceptable solution. 

<aside class="success">
If you cannot guarantee that all input views will be written to, use <code>InputViews.select()</code> to poll the input views in a non-blocking manner. SnapLogic recommends this approach for most multiple-input view scenarios.
</aside>

### Configuring Optional Views

It is rarely a good idea to prevent either an input or an output view for a Snap. For one, it prevents using that Snap as unlinked input at the beginning of [a Task pipeline](http://doc.snaplogic.com/tasks), and restricts its ability to be used mid-pipeline.

However, if you wish to allow the user to remove all input and output views, it's best to declare the views as optional, to open them by default, and then have the user remove them through the **Views** tab of the Snap.

This can be done by implementing the `ViewProvider` interface and using the `defineViews()` method's `ViewBuilder` argument:

<div class="inline-code">
@Inputs(min = 0, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 0, max = 1, offers = {ViewType.DOCUMENT})
...
// this creates the two open input views and the two named output views
@Override
public void defineViews(final ViewBuilder viewBuilder) {
	viewBuilder.describe(INPUT_0)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.INPUT);
	viewBuilder.describe(INPUT_1)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.INPUT);

	viewBuilder.describe(MALE_VIEW)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.OUTPUT);
	viewBuilder.describe(FEMALE_VIEW)
			.type(ViewType.DOCUMENT)
			.add(ViewCategory.OUTPUT);
}
</div>

![Remove Views](http://dl.dropboxusercontent.com/u/3519578/Screenshots/nFE8.png)

### Writing to Multiple Output Views

`OutputViews.getDocumentViewFor(viewName)` can be used to return the document output view for the given view name:

<div class="inline-code">
// send males to male output view
if (data.get("gender").equals("male")) {
	outputViews.getDocumentViewFor(MALE_VIEW).write(newdoc);
} else if (data.get("gender").equals("female")) {
	// send females to female output view
	outputViews.getDocumentViewFor(FEMALE_VIEW).write(newdoc);
} else {
	// send unknowns to error views
	errorViews.write(newdoc, document);
}
</div>

## Pipeline Validation/Preview-specific Behavior

```java
package com.snaplogic.api;
...
/**
 * This interface provides a way to implement the configuration and the execution
 * of the snap for Preview/Validation (formerly known as "Suggest" action).
 *
 * NOTE: This interface is not mandatory to implement for a Snap author.
 *
 * If implemented, the <code>configureForSuggest</code> method will be called in place of the
 * <code>configure</code> method of the Snap interface and the <code>executeForSuggest</code>
 * method will be called in place of <code>execute</code> method of the Snap interface during
 * Preview/Validation.
 *
 * If not implemented, the <code>configure</code> and the <code>execute</code> methods of the
 * snap interface will be called during Preview/Validation.
 *
 * Usecase: A JMS consumer snap
 *
 * The JMS consumer indefinitely polls for the messages to arrive during the execution of the
 * snap. This situation is not desirable for the Suggest/Save action as it will never complete.
 * In addition, we also don't want to remove messages from the destination for the Suggest/Save
 * action by acknowledging them.
 *
 * Hence, the snap author may implement the interface as following:
 * <code>
 *     ...
 *
 *     public void configure(PropertyValues propertyValues) throws ConfigurationException {
 *         ...
 *     }
 *
 *     public void configureForSuggest(PropertyValues propertyValues)
 *              throws ConfigurationException {
 *         // Fetch all the property values EXCEPT Acknowledgment mode.
 *         // Lookup for ConnectionFactory and Destination objects.
 *         // Create Connection object using the above ConnectionFactory
 *         // Create Session object using Session.CLIENT_ACKNOWLEDGE as an acknowledgment mode
 *     }
 *
 *     public void execute() throws ExecutionException {
 *         ...
 *     }
 *
 *     public void executeForSuggest() throws ExecutionException {
 *         try {
 *             Message message = consumer.receive(5000);
 *             if (message != null) {
 *                 processMessage(message);
 *                 // Don't acknowledge the message so that it does't
 *                 // get removed from the destination
 *             }
 *         } catch (JMSException e) {
 *             // Handle the exception
 *         }
 *
 *     }
 *     ...
 * </code>
 */
public interface SuggestExecutionProvider {
    /**
     * This method will be called for the Snap configuration during Suggest
     *
     * @param propertyValues
     * @param maxSuggestValue as the maximum of rows/time being read
     */
    void configureForSuggest(PropertyValues propertyValues,
            BigInteger maxSuggestValue) throws ConfigurationException;

    /**
     * This method will be called for the Snap execution during Suggest
     */
    void executeForSuggest() throws ExecutionException;
}
```

If pipeline validation is enabled and the user saves the pipeline in the Designer, the SnapLogic Platform attempts to validate the pipeline and displays preview data between Snaps.

When validating the pipeline, the Platform will attempt to limit the amount of data processed (for example, up to 50 input documents) as it is not a full pipeline execution but rather a tool to give visual feedback on the data as it flows through the pipeline.

If your Snap wishes to customize its behaviour during pipeline, it can implement the `SuggestExecutionProvider` interface.

<aside class="notice">
"Suggest" is the legacy name for the Pipeline Validation and Preview feature and not related to <a href="#suggesting-property-values">the current Property Suggest feature</a> in a Snap's Settings. Our apologies for any confusion this will cause.
</aside>

As per the JavaDocs of `SuggestExecutionProvider`, implementing the `configureForSuggest()` and `executeForSuggest()` methods can trigger specific behavior during Snap configuration and execution when the pipeline is being validated.

A thrown `SuggestViewAbortException` can indicate that the pipeline validation/preview-specific behavior threshold has been reached.

# Deploying Snap Packs

Now that we understand how to build a Snap, let's see how to deploy it so it can be used within the SnapLogic Platform.

Snap Packs are Maven projects, so they benefit from rich tooling support on the terminal, within IDE plugins, and from Continuous Integration servers.

There are two ways to deploy a Snap Pack - one targeted for developers seeking rapid deployment during Snap development, the other for Organization admin and/or operational users to upload through the SnapLogic Manager UI.

## mvn snappack:deploy

```shell
$ cd $SNAP_HOME/demosnappack
$ mvn clean install
$ mvn snappack:deploy
```

The [Snap Archetype](#snap-maven-archetype) POM includes the `snappack-installer` plugin which contains the `snappack:deploy` goal.

After building the Snap Pack with `mvn clean package`, the `snappack:deploy` goal POSTs the Snap Schema to endpoint associated to the `jcc.sldb_uri` property in `~/opt/snaplogic/etc/keys.properties`. 

It will use the properties and credentials defined in the Snap's `pom.xml` and the `~/opt/snaplogic/etc/keys.properties` file respectively - it is imperatively that they are configured correctly.

By default, the binaries will not be deployed (it will look for the ZIP of JARs using the `$SNAP_HOME` environment variable) and will deploy to [the `ASSET_DIR_PATH` specified in the `directives` file](#project-structure) in *src/main/config*.

### asset_path

Use the `asset_path` parameter if you wish to override the value of the `ASSET_DIR_PATH` specified in the directives file e.g. `mvn snappack:deploy -Dasset_path=/snaplogic/shared`.

This can be a useful feature for Continuous Integration and Continuous Deployment tools.

### binaries

Use the `binaries` boolean parameter to permit the Snap to be executed on Snaplexes other than the development Snaplex configured earlier, you can instruct the goal to upload the ZIP of the Snap's JAR files (source and dependencies) to SLDB.

`mvn snappack:deploy -Dbinaries=true`

<aside class="success">
We recommend using this Maven goal during Snap development, to be run on a development Snaplex, and to deploy to a dedicated development project folder or a Project Space, rather than your Organization's root shared directory.
<br />
<br />
While every effort has been made to ensure the stability of this feature, it is possible for it to upload versions in conflict with other versions of the Snap deployed through the Manager, or for other errors to occur. If that takes place, use the Manager UI Snap Pack ZIP upload method instead (see below).
</aside>

## Uploading ZIP File through Manager UI

```shell
$ ls -al ~/opt/snaplogic-dev/demosnappack/target/ | grep zip
-rw-r--r--   1 snapdev  staff  52588 Jul 21 01:46 demosnappack-1-0001.zip
```

After a `mvn clean install` execution, the Snap's `target` directory will contain a Snap Pack ZIP file containing the zipped source and dependency JARs of the Snap, the Snap and Account schema files, and a `MANIFEST`.

This ZIP file can then be uploaded through the Manager UI to the root/project folder/Project Space directory of your choice.

![Manager](https://dl.dropboxusercontent.com/u/3519578/Screenshots/WhVl.png)

<aside class="success">
We recommended using this method of deployment when releasing a stable version of your custom Snap to your users, and when uploading to the Organization's root shared directory. Preferably, development versions of the Snap Pack should be deleted from development project folders and/or Project Spaces prior to this to avoid confusion.
<br />
<br />
In the near future, SnapLogic will update the <code>snappack:deploy</code> goal to upload the Snap Pack ZIP file to match this process.
</aside>

### Troubleshooting

The SnapLogic Platform will attempt to load the version of the Snap Pack associated with the pipeline's saved location, falling back to the Organization's root shared directory if needed.

If multiple versions of a custom Snap exist in different location, and with incompatible Snap Schemas, the Platform can experience issues if pipelines containing that custom Snaps are moved between the locations.

Also, always refresh the browser after uploading a new version of a Snap Pack containing Snap or Account Schema changes. If the problem persists, also consider deleting the entries in your browser's session storage:

![Session Storage](http://dl.dropboxusercontent.com/u/3519578/Screenshots/s45d.png)

If during local development your find your local Snaplex/JCC picking up Snap changes, stop and restart it.

# Authenticating with Accounts

```java
package com.snaplogic.account.api;

/**
 * Interface for Account classes which can be used by Snap classes
 *
 * @param <T> as the return type for {@link #connect}
 */
public interface Account<T> {
    /**
     * Defines the properties of the account using the provided {@link PropertyBuilder}
     * The property category is set to {@link PropertyCategory#ACCOUNT} and should not be
     * changed.
     *
     * @param propertyBuilder as the builder
     */
    void defineProperties(final PropertyBuilder propertyBuilder);

    /**
     * Sets the property values on the account
     *
     * @param propertyValues as the property values
     */
    void configure(final PropertyValues propertyValues);

    /**
     * Allows to connect to the endpoint
     *
     * @return an optional value that might be needed to access the session
     * @throws ExecutionException
     */
    T connect() throws ExecutionException;

    /**
     * Allows to disconnect from the endpoint
     *
     * @throws ExecutionException
     */
    void disconnect() throws ExecutionException;
}
```

Snap Accounts allow the re-use of authentication credentials and properties across Snaps. 

They also permit storing, encrypting, and obfuscating sensitive information like passwords, secret keys etc.

## Account Configuration

```java
package com.snaplogic.snaps;
...
@General(title = "Example Snap Account")
@Version(snap = 1)
@AccountCategory(type = AccountType.CUSTOM)
public class ExampleAccount implements Account<String> {

    protected static final String USER_ID = "userId";
    protected static final String PASSPHRASE = "passphrase";

    private String userId;
    private String passphrase;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(USER_ID, "User ID", "ID of the user")
                .required()
                // for Enhanced Account Encryption; indicate to the SnapLogic Platform
                // that Medium/High Sensitivity-configured Organizations should encrypt
                // this data
                .sensitivity(SnapProperty.SensitivityLevel.MEDIUM)
                .add();

        propertyBuilder.describe(PASSPHRASE, "Passphrase", "The user's passphrase")
                .required()
                .obfuscate() // masks user's input and sets SensitivityLevel to HIGH
                .add();
    }

    @Override
    public void configure(PropertyValues propertyValues) {
        // Exercise: sanitize and validate
        userId = propertyValues.get(USER_ID);
        passphrase = propertyValues.get(PASSPHRASE);
    }
    
    @Override
    public String connect() throws ExecutionException {
        /*
        Return a String that conforms to this simple hash-token scheme:

        base64(userId + ":" + expirationTimestamp + ":" +
             md5(userId + ":" + expirationTimestamp + ":" passphrase))
         */
        long expiration = DateTime.now().plusDays(1).getMillis();
        byte[] md5;

        try {
            MessageDigest md5MessageDigest = MessageDigest.getInstance("MD5");
            md5 = md5MessageDigest.digest(
                    (getUserId() + ":" + expiration + ":" + getPassphrase()).getBytes(UTF_8));
        } catch (NoSuchAlgorithmException e) {
            throw new ExecutionException(e, "Unable to get MD5 MessageDigest instance")
                    .withResolution("Contact the Snap Developer");
        }

        return encodeBase64String(
                (userId + ":" + expiration + ":" + new String(md5)).getBytes(UTF_8));
    }

    @Override
    public void disconnect() throws ExecutionException {
        // no-op
    }
    
    ...
}
```

At their most basic, Accounts will implement the `Account` interface and provide a Generic Type that is used as the return type of the `Account.connect()` method.

Similarly to Snaps, you may use the `defineProperties()` method to build the Settings tab UI, and the `configure()` method to bind/validate the user's input. 

<aside class="warning">
Unlike Snaps, you cannot <a href="#expression-enabled-properties">expression-enable</a> Account properties.
</aside>

![Create Custom Account](https://dl.dropboxusercontent.com/u/3519578/Screenshots/fNOT.png)

<aside class="success">
Remember to declare the new Account in the Snap POM's <code>&lt;account.classes&gt;</code> element e.g.
<br />
<br /><code><strong>&lt;account.classes&gt;com.snaplogic.snaps.ExampleAccount&lt;/account.classes&gt;</strong></code>
</aside>

### Encrypting and Obfuscating User Input

It can also be particular useful to mark particular properties as containing sensitive information and/or requiring input masking. 

The `sensitivity()` builder method allows setting a `LOW`, `MEDIUM`, or `HIGH` `SensitivityLevel`, which instructs the SnapLogic Platform's [Enhanced Account Encryption](http://doc.snaplogic.com/account-encryption) feature to encrypt the marked account data.

The `obfuscate()` builder method masks user input and automatically sets the property to `SensitivityLevel.HIGH`.

### Connecting the Account

The `Account` interface also defines a `connect()` method (whose return type is determined by the [Generic Type](https://docs.oracle.com/javase/tutorial/java/generics/types.html) provided by the implementing class) and a `disconnect()` method.

For demonstration purposes, we've chosen to implement a simple hash-token scheme for creating an authentication String to be used within a Snap to connect to some service.

## Associating Accounts with Snaps

```java
package com.snaplogic.snaps;
...
@General(title = "Account Example", purpose = "Authenticates with an Account",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Inputs(min = 0, max = 1, accepts = {ViewType.DOCUMENT})
@Outputs(min = 1, max = 1, offers = {ViewType.DOCUMENT})
@Version(snap = 1)
@Category(snap = SnapCategory.READ)
@Accounts(provides = {ExampleAccount.class}, optional = true)
public class SnapWithAccount extends SimpleSnap {

    private static final Logger log = LoggerFactory.getLogger(SnapWithAccount.class);

    private static final String USER_ID_COPY = "userIdCopy";

    private String userIdCopy;

    @Inject
    private ExampleAccount snapAccount;

    @Override
    public void defineProperties(PropertyBuilder propertyBuilder) {
        propertyBuilder.describe(USER_ID_COPY, "User ID Copy", 
        		"Demonstrates account.userId expression variable")
                .required()
                .expression(SnapProperty.DecoratorType.ACCEPTS_SCHEMA)
                .add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
        userIdCopy = propertyValues.getAsExpression(USER_ID_COPY).eval(null);
    }

    @Override
    protected void process(Document document, String s) {
        String token = snapAccount.connect();

        Map<String, Object> data = new LinkedHashMap<>();
        data.put("token", token);
        data.put("userId", userIdCopy);

        // do something with the token

        outputViews.write(documentUtility.newDocumentFor(document, data));
    }
}
```

The `@Accounts` metadata annotation is used to declare what Accounts are supported by the Snap. The `provides` argument value lists the supported Account classes, and the `optional` argument instructs the Designer whether to prompt for an Account as soon the Snap is dragged onto the canvas.

The `ExampleAccount` instance bound to the "Example Account" Account created earlier can then be injected into the Snap and used, for example, in the `execute()`/`process()` method to `connect()` to a target service provider or to generate an authentication token etc.

<aside class="success">
Remember to declare the new Snap in the Snap POM's <code>&lt;snap.classes&gt;</code> element e.g.
<br />
<br />
<code>&lt;snap.classes&gt;
<br />&nbsp;&nbsp;com.snaplogic.snaps.DocConsumer,
<br />&nbsp;&nbsp;com.snaplogic.snaps.DocGenerator,
<br />&nbsp;&nbsp;com.snaplogic.snaps.PropertyTypes,
<br />&nbsp;&nbsp;com.snaplogic.snaps.SchemaExample,
<br />&nbsp;&nbsp;com.snaplogic.snaps.SingleDocGenerator,
<br />&nbsp;&nbsp;com.snaplogic.snaps.Suggest,
<br />&nbsp;&nbsp;com.snaplogic.snaps.TwoInputs,
<br />&nbsp;&nbsp;com.snaplogic.snaps.TwoInputsTwoOutputs,
<br />&nbsp;&nbsp;<strong>com.snaplogic.snaps.SnapWithAccount</strong>
<br />&lt;/snap.classes&gt;
</code>
</aside>

## Exposing Account Properties

```java
@General(title = "Example Snap Account")
@Version(snap = 1)
@AccountCategory(type = AccountType.CUSTOM)
public class ExampleAccount implements Account<String>, AccountVariableProvider {
	
	...
	
    @Override
    public Map<String, Object> getAccountVariableValue() {
        return new ExpressionVariableAdapter() {
            @Override
            public Set<Entry<String, Object>> entrySet() {
                return new ImmutableSet.Builder<Entry<String, Object>>()
                        .add(entry(USER_ID, getUserId()))
                        .add(entry(PASSPHRASE, getPassphrase()))
                        .build();
            }
        };
    }
}
```

The `SnapWithAccount` sample also demonstrates another Snap/Account integration by exposing property values of the associated `Account` instance in the Settings of the Snap.

To do this, have the Account implement the `AccountVariableProvider` interface and its `getAccountVariableValue()` method, and store the  e.g.

The `account.userId` and `account.passphrase` expression variables can then be used within the Snap's expression-enabled properties:

![Account Expression Variable](https://dl.dropboxusercontent.com/u/3519578/Screenshots/yF6t.png)

<aside class="notice">
Do not prefix the <code>account</code> expression variable with <code>$</code> as it is not in the document. It also has the limitation that it is not listed within the expression UI's "Functions and properites" dropdown/tree.
</aside>

### Account Types

In the `ExampleAccount` sample provided, the Account has been marked with the `@AccountCategory` annotation, whose `type` argument value is `AccountType.CUSTOM`.

The list of provided Account Category Types are:

* `NONE`
* `CUSTOM`
* `BASIC_AUTH`
* `SSH`
* `SSL`
* `DATABASE`
* `OAUTH2`
* `OAUTH1`
* `AWS_S3`
* `SAP`

## Validating Accounts

```java
package com.snaplogic.account.api;

/**
 * A marker interface for the accounts that can be validated
 *
 * @param <T> as the return type for {@link #connect}
 */
public interface ValidatableAccount<T> extends Account<T> {
}
```

Implementing the marker interface `ValidatableAccount`, which extends `Account`, will add a **"Validate"** button to the Account Settings's UI. 

<div class="inline-code">
public class ExampleAccount implements <strong>ValidatableAccount&lt;String&gt;</strong>, AccountVariableProvider {
...
}
</div>

When clicked, this will trigger a call to the Account's `connect()` method. If the `connect()` method returns successfully (an exception is not thrown), the Account will be deemed validated:

![Successful Account Validation](https://dl.dropboxusercontent.com/u/3519578/Screenshots/KsZR.png)

## Validation-specific Behavior

```java
package com.snaplogic.account.api;
...
/**
 * A marker interface for the accounts that can be validated.
 * This interface can be implemented when connect() method is not sufficient to validate the
 * account.
 *
 * @param <T> as a return type for {@link #validate}
 * @param <R> as an argument for {@link #validate}
 */
public interface ExtendedValidatableAccount<T, R> extends ValidatableAccount<T> {
    /**
     * Validates the account. Please note that connect() should not be called by this validator.
     * If needed, one needs to call connect() separately.
     *
     * @param arg - argument needed to validate the account
     *
     * @return an optional value that might be needed to access the session
     * @throws ExecutionException
     */
    T validate(R arg);
}
```

Implementing `ExtendedValidatableAccount`, which extends `ValidatableAccount`, will also add a **"Validate"** button to the Account Settings's UI. 

However, it will instead trigger a call to `Account.validate()`, rather than `connect()`, when the **"Validate"** button in the Account Settings' UI is clicked. A separate Generic Type is required for the return type of the `validate()` method.

<div class="inline-code">
public class ExampleAccount implements ExtendedValidatableAccount&lt;String, String&gt;, AccountVariableProvider {
	...
	@Override
	public String validate(String arg) {
		// some validation-specific actions
	}
}
</div>

This is useful when an Account can be validated in a different manner versus connecting the Account during Snap execution.

## Supporting Multiple Accounts

```java
package com.snaplogic.snaps;
...
import com.snaplogic.account.api.capabilities.MultiAccountBinding;
...
@Accounts(provides = {ExampleAccount.class, AnotherAccount.class})
public class SnapWithAccount extends SimpleSnap implements MultiAccountBinding<Account<String>> {
	
	/*
    Step #1: when the Snap is initialized, the Account type (with String generic type) is 
    bound to the Account instance registered with the Snap.
     */
    @Override
    public Module getManagedAccountModule(final Account<String> account) {
        return new AbstractModule() {
            @Override
            protected void configure() {
                bind(Key.get(new TypeLiteral<Account<String>>() {})).toInstance(account);
            }
        };
    }
    
    ...
    
    /*
    Step #2: Guice will inject the Account instance that was bound below.
     */
    @Inject
    private Account<String> snapAccount;
	
	...
    
    /*
    Step 3: by the time it gets here, snapAccount will have been bound to the Account 
    instance in the Account Registry for this Snap.
     */
    @Override
    protected void process(Document document, String s) {
        String token = snapAccount.connect();
        ...
    }
}
```

The `@Accounts` annotation allows multiple Account classes to be registered to the `provides` argument.

The `MultiAccountBinding` interface possesses a `getManagedAccountModule()` method that returns a [Guice `Module`](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Module.html).

Implementations of this method can then return an [`AbstractModule`](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/AbstractModule.html) instance which binds the `Account<String>` type literal [to the instance](https://github.com/google/guice/wiki/InstanceBindings) of the `Account` registered with the Snap.

Then, when it is time for the `Account<String>` instance to be injected, Guice will use the instance bound to above and assign it to the `snapAccount` instance variable.

# jtest: Snap Unit Testing Framework

We briefly touched on unit testing Snaps with **jtest** earlier in the [Unit Testing Snaps with SnapTestRunner](#unit-testing-snaps-with-snaptestrunner) section. 

In the provided `SingleDocGeneratorTest` sample, we saw how to use the `SnapTestRunner` JUnit runner, and how to use the `@TestFixture` annotation to target a Snap, simulate its execution, and verify its output.

## Declarative Testing with TestFixture

`@TestFixture` is the annotation for declaratively setting up a Snap unit test.

It provides a number of methods to make testing a Snap as easy as possible. Indeed, it is possible to test a Snap entirely without writing an test-specific implementation.

Method | Description
--------- | -----------
`snap` 				| Returns the Snap class to be tested.
`input` 			| Returns the path to the file that represents the input data.
`inputs` 			| Returns the names of the input views for this unit test.<br />The input name has to match the one<br />defined in the input properties file.
`outputs` 			| Returns the names of the output views for this unit test.
`errors` 			| Returns the names of the error views for this unit test.
`exception` 		| Returns the exception class that is expected to be thrown.
`expectedOutputPath` | The path to the directory where expected output<br />files can be found.<br />The files are named based on the test method name followed by<br />"-out.json" (e.g. testBasic-out.json).<br />Expected output will be written to a<br />temporary directory on failure,<br />so you can just copy the files from there<br />to the expected directory if an update is needed.
`expectedErrorPath` | The path to the directory where expected error output<br />files can be found.<br />The files are named based on the test method name followed by<br />"-err.json" (e.g. testBasic-err.json).<br />Expected output will be written to a<br />temporary directory on failure,<br />so you can just copy the files from there<br />to the expected directory if an update is needed.
`account` 			| Returns the Account class to be tested.
`accountProperties` | Returns the path to the Account properties<br />file to use for this unit test.
`properties` 		| Returns the path to the Snap properties<br />file to use for this unit test.
`propertyOverrides` | Overrides for Snap properties specified by<br />the properties file.<br />The value should be pairs of strings where the left-side<br />is the JSON-Path to write and the right-side is the<br />JSON-encoded value to store in the path.
`injectorModule` 	| [Guice Module](https://google.github.io/guice/api-docs/latest/javadoc/index.html?com/google/inject/Module.html) that can override bindings<br />in the default test module.
`dataFiles` 		| Returns an array of data file paths.<br />These files will be available in the `URLStreamHandlerFactory` factory.
`dataFilesSupplier` | If specified, an instance of this class will be created<br />and used to supply an array of data file paths<br />for the `URLStreamHandlerFactory` factory.<br />This is an alternative to specifying `dataFiles` when<br />the same set of data files is needed by multiple tests.<br />If both `dataFiles` and `dataFilesSupplier` are specified,<br />`dataFiles` will be used.
`dataFilesSupplier` | If specified, an instance of this class will be created<br />and used to supply an array of data file paths<br />for the `URLStreamHandlerFactory` factory.<br />This is an alternative to specifying `dataFiles` when<br />the same set of data files is needed by multiple tests.<br />If both `dataFiles` and `dataFilesSupplier` are specified<br />, `dataFiles` will be used.

The following sections will use the unit tests included in the [Snap Maven Archetype](#snap-maven-archetype).

## Recording Output Test Data

```java
@RunWith(SnapTestRunner.class)
public class SingleDocGeneratorTest {

    @TestFixture(snap = SingleDocGenerator.class,
            <strong>outputs = "output0"</strong>)
    public void testSingleDocGeneratorFunctionality(TestResult testResult)
            throws Exception {
        assertNull(testResult.getException());
        <strong>OutputRecorder outputRecorder = testResult.getOutputViewByName("output0");</strong>
        assertEquals(1, outputRecorder.getRecordedData().size());
    }

}
```

Specify an output view name in `outputs` and then call `getRecordedData()` on the `OutputRecorder` instance returned by the `testResult.getOutputViewByName()` call using that same output view name.

## Examining Error Views

```java
@RunWith(SnapTestRunner.class)
public class SchemaExampleTest {

    ...

    @TestFixture(snap = SchemaExample.class,
            input = "data/schema_invalid_input.data",
            outputs = "output0",
            <strong>errors = "error0"</strong>)
    public void testInValidDataAgainstSchema(TestResult testResult)
            throws Exception {
        // Since the input document does not conform to the expected schema,
        // there should be no output.
        OutputRecorder outputRecorder = testResult.getOutputViewByName("output0");
        assertEquals(0, outputRecorder.getDocumentCount());

        // Input document should be forwarded to the error view as it does not
        // match the expected schema.
        <strong>OutputRecorder errorRecorder = testResult.getErrorViewByName("error0");</strong>
        assertEquals(1, errorRecorder.getDocumentCount());
    }
}
```

Very similar to [Recording Output Test Data](#recording-output-test-data), except the `testResult.getErrorViewByName()` method is called instead.

## Providing Input Test Data

```java
@RunWith(SnapTestRunner.class)
public class SchemaExampleTest {

    @TestFixture(snap = SchemaExample.class,
            input = "data/schema_valid_input.data",
            outputs = "output0")
    public void testValidDataAgainstSchema(TestResult testResult)
            throws Exception {
        // Input document should appear on the output
        OutputRecorder outputRecorder = testResult.getOutputViewByName("output0");
        assertEquals(1, outputRecorder.getDocumentCount());
    }

    ...
}
```

`input` specifies the path to the file that represents the data entering the input view, for example:

*data/schema_valid_input.data*
<div class="inline-code">
{
    "input0" : [
        {
            "colA" : "colA",
            "colB" : "colB",
            "colC" : "colC"
        }
    ]
}
</div>

The format of the input data file is:

<div class="inline-code">
{
	"input_view_name" : data (can be a dict, an array or primitives),
	"input_view_name" : data (can be a dict, an array or primitives)
}
</div>

Example:

<div class="inline-code">
{
	"input0" : [1, 2, 3, 4, 5],
	"input1" : ["foo", "bar", "baz"]
}
</div>

The format of a binary input data file is:

<div class="inline-code">
{
	"input_view_name" : [data_file_path, ...],
	"input_view_name" : [data_file_path, ...]
}
</div>

Example:

<div class="inline-code">
{
	"input0" : ["data/files/json_input1.json", "data/files/json_input1.json"],
	"input1" : ["data/files/json_input2.json"]
}
</div>

Optionally, headers can be specified for any array element as follows:

<div class="inline-code">
{
	"input0" : [
		{
			"header" : {
				"content-type" : "application/json"
			},
			"content" : "data/files/json_input1.json"
		}, 
		"data/files/json_input1.json"
	],
	"input1" : ["data/files/json_input2.json"]
}
</div>

## Specifying Snap Test Properties

```java
@RunWith(SnapTestRunner.class)
public class DocGeneratorTest {

    @TestFixture(snap = DocGenerator.class,
            outputs = "output0",
            properties = "data/doc_generator_properties.json")
    public void testDocGeneratorFunctionality(TestResult testResult) throws Exception {
        assertNull(testResult.getException());
        OutputRecorder outputRecorder = testResult.getOutputViewByName("output0");
        assertEquals(5, outputRecorder.getRecordedData().size());
    }
}
```

`properties` can be used to simulate user input into a Snap's settings, including expression-enabled properties.

The format of the property data file is:

<div class="inline-code">
{
	"category" : { <em>a dict</em> },
	"another" : { <em>a dict</em> }
}
</div>

Example:

<div class="inline-code">
{
	"settings" : {
		"filename" : {
			"value": "$someProperty",
			"expression": true
		},
		"delim" : {
			"value" : ","
		},
		"composition" : {
			...
		}
	}
}
</div>

`propertiesOverrides` can override the value of a Snap property specified in the `properties` file.

The value should be pairs of strings where the left-side is the JSON-Path to write and the right-side is the JSON-encoded value to store in the path.

Example:

<div class="inline-code">
...
propertyOverrides = {
	"$.settings.delim.value", ";"
}
...
</div>

## Injecting Test Dependencies

```java
@RunWith(SnapTestRunner.class)
public class DocConsumerTest {

    @TestFixture(snap = DocConsumer.class)
    public void testDocConsumerFunctionality(TestSetup testSetup) throws Exception {
        testSetup.addInputView("input0", Arrays.<Object>asList("1", "2"));
        
        // This demonstrates how to inject instances/stubs/mocks into Snap fields.
        AtomicInteger count = new AtomicInteger(0);
        testSetup.inject().fieldName("count").dependency(count).add();

        TestResult testResult = testSetup.test();
        assertNull(testResult.getException());
        assertEquals(2, count.get());
    }
}
```

The [`DocConsumer` sample Snap](#reading-documents-from-an-input-view) contains an `AtomicInteger` "count" field.

If we wish to set this field directly, we can use `inject()` from `TestSetup`.

`injectorModule` allows providing a custom `AbstractModule` class to override Guice bindings and influence fields annotated with `@Inject`.

# PropertyBuilder Reference

```java
@General(title = "Property Types", purpose = "Demonstrates use of different property types",
        author = "Your Company Name", docLink = "http://yourdocslinkhere.com")
@Category(snap = SnapCategory.READ)
@Version(snap = 1)
@Inputs(min = 0, max = 0)
@Outputs(min = 1, max = 1)
public class PropertyTypes extends SimpleSnap {
    private static final String PASSWORD_PROP = "password_prop";
    private static final String FILE_BROWSER_PROP = "file_browser_prop";
    private static final String CHILD_FILE_BROWSER_PROP = "child_file_browser_prop";
    private static final String CHILD_PROP = "child_prop";
    private static final String PARENT_PROP = "parent_prop";
    private static final String COLUMN_FILE_BROWSER_PROP = "column_file_browser_prop";
    private static final String COLUMN_PROP = "column_prop";
    private static final String TABLE_PROP = "table_prop";
    private static final String PASSWORD_PROP_VALUE = "abc";

    @Inject
    private DocumentUtility documentUtility;

    @Override
    public void defineProperties(final PropertyBuilder propBuilder) {
        // Password (obfuscate)
        propBuilder.describe(PASSWORD_PROP, PASSWORD_PROP)
                .required()
                .defaultValue(PASSWORD_PROP_VALUE)
                .obfuscate()
                .add();
                
        // File Browsing
        propBuilder.describe(FILE_BROWSER_PROP, FILE_BROWSER_PROP)
                .required()
                .defaultValue(PASSWORD_PROP_VALUE)
                .fileBrowsing()
                .add();

        // Composite Property
        SnapProperty fileBrowsingChild = propBuilder.describe(CHILD_FILE_BROWSER_PROP,
                CHILD_FILE_BROWSER_PROP)
                .required()
                .fileBrowsing()
                .build();
        SnapProperty child = propBuilder.describe(CHILD_PROP,
                CHILD_PROP)
                .required()
                .build();
        propBuilder.describe(PARENT_PROP, PARENT_PROP)
                .type(SnapType.COMPOSITE)
                .required()
                .withEntry(fileBrowsingChild)
                .withEntry(child)
                .add();
                
        // Table Property
        SnapProperty fileBrowsingColumn = propBuilder.describe(COLUMN_FILE_BROWSER_PROP,
                COLUMN_FILE_BROWSER_PROP)
                .required()
                .fileBrowsing()
                .build();
        SnapProperty column = propBuilder.describe(COLUMN_PROP,
                COLUMN_PROP)
                .required()
                .build();
        propBuilder.describe(TABLE_PROP, TABLE_PROP)
                .type(SnapType.TABLE)
                .withEntry(fileBrowsingColumn)
                .withEntry(column)
                .add();
    }

    @Override
    public void configure(PropertyValues propertyValues) throws ConfigurationException {
    }

    @Override
    public void process(Document document, String inputViewName) {
        Map<String, String> data = new LinkedHashMap<String, String>() {{
            put("key", "value");
        }};
        outputViews.write(documentUtility.newDocument(data), document);
    }
}
```

The `PropertyTypes` sample Snap in the [Snap Maven Archetype](#snap-maven-archetype) demonstrates a number of the various UI components available to a Snap's Settings UI.

![Property Values](http://dl.dropboxusercontent.com/u/3519578/Screenshots/qJXX.png)

### SnapType Reference

<div class="inline-code">
NUMBER("number", BigDecimal.class),
INTEGER("integer", BigInteger.class),
STRING("string", String.class),
DATETIME("date-time", DateTime.class),
LOCALDATETIME("local-date-time", LocalDateTime.class),
BOOLEAN("boolean", Boolean.class),
DATE("date", LocalDate.class),
TIME("time", LocalTime.class),
BINARY("binary", Byte.class),
TABLE("array", List.class),
ANY("any", Object.class),
COMPOSITE("object", Map.class),
BYTES("bytes", String.class);
</div>