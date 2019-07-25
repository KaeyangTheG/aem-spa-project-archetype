## Maven Archetype for AEM React SSR project

## System requirements

- [Java](https://www.java.com/en/download/) 1.8 or higher
- [Maven](https://maven.apache.org/) 3.5.0 or higher
- Include the [Adobe Public Maven Repository](adobe-public-maven-repo) in your maven settings

It is recommended to set up the local AEM instances with `nosamplecontent` run mode.

Modules of the generated project is defined in [src/main/resources/archetype-resources](src/main/resources):

* [core](core/): OSGi bundle containing:
  * Java classes (e.g. Sling Models, Servlets, business logic)
* [ssr](ssr/): Contains a class for fetching server side markup from the nodejs proxy
* [ui.apps/src/main/content/jcr_root/apps](content/jcr_root/apps/):
  * AEM components with their scripts and dialog definitions
* [ui.content/src/main/content/jcr_root/conf](content/jcr_root/conf/): 
  * AEM content package with editable templates stored at `/conf`
* [ui.content/src/main/content/jcr_root/content](content/jcr_root/content/): 
  * AEM content package containing sample content (for development and test purposes)
* [react-app](react-app/): The React application
* [all](all/): Combines all modules to be installed as content package in AEM

## Required parameters

This archetype requires following parameters:
- `groupId` - Maven artifact groupId for all projects
- `artifactId`(default is `${groupId}.${projectName}`) - Maven artifact "root" artifactId, is suffixed for the individual modules
- `version` (default is `1.0.0-SNAPSHOT`) - Maven artifact version
- `package` (default is `${groupId}.${projectName}`) - Java class package name
- `projectName` (default is `mysamplespa`) - Used for building AEM apps path, content path, conf etc. Should not include spaces or special character.
- `projectTitle` (default is `My Sample SPA`) - Descriptive project name
- `componentGroup` (default is `${projectTitle}`) - Name of the component group in AEM Editor

## Building SPA Starter Kit Archetype

```
$ mvn clean install archetype:update-local-catalog
```

## Updating list of locally available archetypes

```
$ mvn archetype:crawl
```

## Archetype catalog variants

Depending on the use case maven can use different archetype variant (use `-DarchetypeCatalog` to choose one):
- `internal` represents `~/.m2/repository/`
- `local` represents `~/.m2/archetype-catalog.xml`
- `remote` represents http://repo.maven.apache.org/maven2/archetype-catalog.xml

## Provided Maven profiles
The generated maven project support different deployment profiles when running the Maven install goal `mvn install` within the reactor.

Id                        | Description
--------------------------|------------------------------
autoInstallBundle         | Install core bundle with the maven-sling-plugin to the felix console
autoInstallPackage        | Install the ui.content and ui.apps content package with the content-package-maven-plugin to the package manager to default author instance on localhost, port 4502. Hostname and port can be changed with the aem.host and aem.port user defined properties. 
autoInstallPackagePublish | Install the ui.content and ui.apps content package with the content-package-maven-plugin to the package manager to default publish instance on localhost, port 4503. Hostname and port can be changed with the aem.host and aem.port user defined properties.

The profile `integrationTests` is also available for the verify goal, to run the provided integration tests on the AEM instance.

## Using SPA Starter Kit Archetype

Archetype `aem-spa-project-archetype` must be available locally (by cloning this repo and building it) or on artifactory.

You must be in a directory without a `pom.xml` file. A sub-folder will be created for the newly created project.

Starter Kit project can be created using following options:
- in command line in **interactive** mode
- in command line in **batch** mode

### Creating project in interactive mode

In interactive mode a series of questions will be asked set parameters for new project.

```
$ mvn archetype:generate \
     -DarchetypeCatalog=local \
     -DarchetypeGroupId=com.adobe.cq.spa.archetypes  \
     -DarchetypeArtifactId=aem-spa-project-archetype  \
     -DarchetypeVersion=1.1.1-SNAPSHOT
```

Please note that properties declared in [archetype-metadata.xml](src/main/resources/META-INF/maven/archetype-metadata.xml) with `defaultValue` are not asked during interactive mode and are defaulted to suggested values. 

### Working with the created project

First cd in the root and build and deploy to your local instance:

```
$ mvn -PautoInstallPackage -Padobe-public clean install

```

Next configure two cross-origin policies on your AEM instance: 

1. Navigate to the Configuration Manager on the AEM instance at http://localhost:4502/system/console/configMgr
2. Look for the configuration: Adobe Granite Cross-Origin Resource Sharing Policy
3. Create a new configuration with the following additional values:
    * Allowed Origins: http://localhost:3000
    * Supported Headers: Authorization
    * Allowed Methods: OPTIONS
4. Repeat steps 1-3 on a new policy with http://localhost:4200

Run the frontend locally.  cd into ./react-app and run the following:

```
$ API_HOST=http://localhost:4205 npm run start

```

You can navigate to `http://localhost:3000/content/${projectName}/en/home.html to see it working.

Test out the app with SSR.  In the root of ./react-app run: 

```
$ API_HOST=http://localhost:4205 APP_ROOT_PATH=/content/${projectName}/en npm run start:server

```

This starts a node server that renders the app server-side.  If you inspect and create a breakpoint within the POST method you can test it is working by going to http://localhost:4502/content/${projectName}/en/home.html?wcmmode=disabled.

