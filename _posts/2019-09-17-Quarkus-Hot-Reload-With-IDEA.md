---
layout: post
title: Quarkus Hot Reload With IDEA
---

## [Quarkus](https://quarkus.io/)

> A Kubernetes Native Java stack tailored for GraalVM & OpenJDK HotSpot, crafted from the best of breed Java libraries and standards

For me ? quarkus is a standard  framework,it combine lots of useful web framework.And it provides amazing boot time and optimizes developer joy.Also it can **compile to native app** and support **hot reload**.YES,**hot reload with IDEA** is really awsome!when i found i can use IDEA deloyment Tools automatic upload file to a server machine and it will automic reload the server.Just like below picture say 👇:

![devjoy](/assets/images/devjoy.png)

It sounds easy but things not going well 😟.

## First try

I configuraed the IDEA delopyment tool and upload the [exmpale project](https://quarkus.io/guides/getting-started-guide) to server machine.

![deployment-conf](/assets/images/deployment-conf.jpg)

execute cmd to start up :
> ./mvnw compile quarkus:dev

And everythins same to be fine :

![running-well](/assets/images/running.png)

But no matter how i modify the *ExampleResource.java*,it didn't touch the hot reload.The *ExampleResource.java* did upload to the server machine.Why quarkus server not hot reload? So i using **vim** modify the *ExampleResource.java* on server machine,quarkus did hot reload.This is really confuse me.I'm wandering how's the quarkus hot reload works ? Maybe i can found why IDEA automatic upload not trigger the hot reload.

## Second Try

I notice that when i modify a file and only when i vistied the resource path on borwer quarkus would do hot reload.So i debug on the IDEA and watching what happend.

First i found the restart code ([DevModMain.java](https://github.com/quarkusio/quarkus/blob/master/core/devmode/src/main/java/io/quarkus/dev/DevModeMain.java)) on github:

```java
public synchronized void restartApp(Set<String> changedResources) {
        stop();
        Timing.restart();
        doStart(true, changedResources);
    }
```

And keep digging :

**RuntimeUpdatesProcessor.java**
```java
@Override
    public boolean doScan(boolean userInitiated) throws IOException {
        final long startNanoseconds = System.nanoTime();
        for (Runnable step : preScanSteps) {
            try {
                step.run();
            } catch (Throwable t) {
                log.error("Pre Scan step failed", t);
            }
        }
        //This is key point 
        boolean classChanged = checkForChangedClasses();
        Set<String> filesChanged = checkForFileChange();

       ...
    }
```

Finally i found key method on **checkForChangedClasses()**. 

**RuntimeUpdatesProcessor.java**

```java
 boolean checkForChangedClasses() throws IOException {
        boolean hasChanges = false;
        for (DevModeContext.ModuleInfo module : context.getModules()) {
            final List<Path> moduleChangedSourceFilePaths = new ArrayList<>();
            for (String sourcePath : module.getSourcePaths()) {
                final Set<File> changedSourceFiles;
                try (final Stream<Path> sourcesStream = Files.walk(Paths.get(sourcePath))) {
                    changedSourceFiles = sourcesStream
                            .parallel()
                            .filter(p -> matchingHandledExtension(p).isPresent() && wasRecentlyModified(p))
                            .map(Path::toFile)
                            //Needing a concurrent Set, not many standard options:
                            .collect(Collectors.toCollection(ConcurrentSkipListSet::new));
                }
                ...
    }
```

**wasRecentlyModified(p)**
```java
    private boolean wasRecentlyModified(final Path p) {
        try {
            long sourceMod = Files.getLastModifiedTime(p).toMillis();
            return sourceMod > lastChange;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

**quarkus hot reload depends on compare file last modified time with system last change time.** the whole process is : when i send http request,quarkus will handle the request and check the resource file whether to modify and do hot reload.

So i change the IDEA config like this :

![IDEA-conf](/assets/images/finally-conf.png)


Enjoying Coding with quarkus 👏!