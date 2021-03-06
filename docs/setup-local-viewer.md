# Local Setup

## Before you start

Make sure you have the following installed:

- git

        $ sudo apt install git

This will get you the latest package that is supported by your OS version. In Ubuntu 18.04 (LTS) this will be quite 
out of date (v. 2.17.1). If you prefer to install the latest version of git following 
the instructions [here](https://itsfoss.com/install-git-ubuntu/).

- Java 8 JDK

        $ sudo apt-get install openjdk-8-jdk

Make sure the JAVA_HOME environmental variable points to your installation of openjdk-8-jdk. For example,
 add this line to your /etc/environment file (replacing the path with the path where your JDK is 
 installed):

    JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

- Apache Maven 3.1+

        $ sudo apt install maven

- Docker and Docker Compose

See the official instructions [Install Docker Engine - Community](https://docs.docker.com/engine/install/ubuntu/) and 
[Install Compose on Linux Systems](https://docs.docker.com/compose/install/#install-compose-on-linux-systems). 

## Dependencies
    
### JDK Maven Toolchain

To make sure that Java 1.8 is used to build the artefacts, you need to configure the location
 in the ~/.m2/toolchains.xml file:

    <?xml version="1.0" encoding="UTF8"?>
    <toolchains>
      <!-- JDK toolchains -->
      <toolchain>
        <type>jdk</type>
        <provides>
          <version>1.8</version>
          <vendor>openjdk</vendor>
        </provides>
        <configuration>
          <jdkHome>/usr/lib/jvm/java-8-openjdk-amd64</jdkHome>
        </configuration>
      </toolchain>
    </toolchains>

Replacing /usr/lib/jvm/java-8-openjdk-amd64 with the path where your version is installed. 

### Maven Dependencies

CUDL has its own Maven repository stored on GitHub where the CUDL-specific dependencies are stored for access by Maven.
This should be open and you should be able to download dependencies.  
You can setup the repo in your ~/.m2/settings.xml file as follows:

    <activeProfiles>
    <activeProfile>github</activeProfile>
    </activeProfiles>
    
    <profiles>
    <profile>
      <id>github</id>
      <repositories>
        <repository>
          <id>central</id>
          <url>https://repo1.maven.org/maven2</url>
        </repository>
        <repository>
          <id>github</id>
          <url>https://maven.pkg.github.com/cambridge-collection/*</url>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </repository>
      </repositories>
    </profile>
    </profiles>
    
    <servers>
    <server>
    <id>github</id>
    <username>USERNAME</username>
    <password>TOKEN</password>
    </server>
    </servers>
    </settings>

### Data dependencies

Data (TEI and JSON), some HTML content and images served by the image servers are not managed by Maven and their 
resources are configured in docker/cudl-global.properties for local development deployment. 
Ordinarily, you should not need to change this config to get the viewer running on your local machine. 

- Check out sample data repo: 

  https://github.com/cambridge-collection/dl-data-samples.git
  
  This is a small sample data set containing:
   - TEI
   - JSON
   - TIFF images (blank samples)
   - DATABASE export
   - HTML CONTENT
   
## Setting up the viewer 

The cudl-viewer will need to be checked out at the same level as the dl-data-samples.

- Check out cudl-viewer from:

  https://github.com/cambridge-collection/cudl-viewer.git
  
### Maven Build   
   Open a shell on your machine, or a Terminal in IntelliJ IDEA. (If you cannot see the Terminal
   button in the bottom left-hand corner, hover over the square icon in the bottom left-hand corner
   and choose Terminal from the list.)
   
   ![Terminal in IntelliJ IDEA.png](images/Terminal_in_IntelliJ_IDEA.png)
      
   To pull in dependencies and build a WAR file using Maven:
   
    $ mvn clean package
   
   The first time you do this may take some time as Maven will have to download lots of dependencies. This will create a compiled file /target/FoundationsViewer.war in your local Maven repository.
   Running the CUDL App
   
   The CUDL app is set up to run locally using Docker Compose. To run the app with the default configuration file docker-compose.yml:
   
    $ docker-compose up
   
   while in the project directory.
   
   Access the app at [http://localhost:8888/](http://localhost:8888/) and check that it is working.
   
## Image Server

The image server that zoomable image tiles come from is configured in the cudl-global.properties
file. This can be altered by changing the IIIFImageServer and ImageServer properties.  
We are using the IIIF image server [IIPImage](https://iipimage.sourceforge.io/). 

    imageServer=https://images.lib.cam.ac.uk/
    IIIFImageServer=https://images.lib.cam.ac.uk/iiif/


## Further Information

For further information on debugging and developing using the cudl-viewer 
see [Developing and Debugging the Viewer](./developing-debugging-viewer.md).


