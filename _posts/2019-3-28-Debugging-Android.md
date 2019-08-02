---
layout: post
title: Debugging Android
---

The steps I took to fix a broken build when Android failed to build, and nothing had changed with the source code.

Recently one of our Android projects failed to build a release APK using the command `./gradlew assembleRelease`. It's profoundly odd since there were no changes to the Android source code between the time this command last worked and this recent compilation. 

To be clear, this was a React Native project, and the only changes made were in the Javascript files - this shouldn't have KO'd the Android build process at all. And when I say KO'd, I mean I got this error:

```
Dex: Error converting bytecode to dex:
Cause: Dex cannot parse version 52 byte code.
This is caused by library dependencies that have been compiled using Java 8 or above.
If you are using the 'java' gradle plugin in a library submodule add
targetCompatibility = '1.7'
sourceCompatibility = '1.7'
to that submodule's build.gradle file.
    UNEXPECTED TOP-LEVEL EXCEPTION:
    java.lang.RuntimeException: Exception parsing classes
        at com.android.dx.command.dexer.Main.processClass(Main.java:775)
        at com.android.dx.command.dexer.Main.processFileBytes(Main.java:741)
        at com.android.dx.command.dexer.Main.access$1200(Main.java:88)
        at com.android.dx.command.dexer.Main$FileBytesConsumer.processFileBytes(Main.java:1683)
        at com.android.dx.cf.direct.ClassPathOpener.processArchive(ClassPathOpener.java:284)
        at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:166)
        at com.android.dx.cf.direct.ClassPathOpener.process(ClassPathOpener.java:144)
        at com.android.dx.command.dexer.Main.processOne(Main.java:695)
        at com.android.dx.command.dexer.Main.processAllFiles(Main.java:592)
        at com.android.dx.command.dexer.Main.runMonoDex(Main.java:321)
        at com.android.dx.command.dexer.Main.run(Main.java:292)
        at com.android.builder.internal.compiler.DexWrapper.run(DexWrapper.java:54)
        at com.android.builder.core.DexByteCodeConverter.lambda$dexInProcess$0(DexByteCodeConverter.java:173)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
    Caused by: com.android.dx.cf.iface.ParseException: bad class file magic (cafebabe) or version (0034.0000)
        at com.android.dx.cf.direct.DirectClassFile.parse0(DirectClassFile.java:476)
        at com.android.dx.cf.direct.DirectClassFile.parse(DirectClassFile.java:406)
        at com.android.dx.cf.direct.DirectClassFile.parseToInterfacesIfNecessary(DirectClassFile.java:388)
        at com.android.dx.cf.direct.DirectClassFile.getMagic(DirectClassFile.java:251)
        at com.android.dx.command.dexer.Main.parseClass(Main.java:787)
        at com.android.dx.command.dexer.Main.access$1600(Main.java:88)
        at com.android.dx.command.dexer.Main$ClassParserTask.call(Main.java:1722)
        at com.android.dx.command.dexer.Main.processClass(Main.java:773)
        ... 16 more

1 error; aborting
:app:transformClassesWithDexForRelease FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:transformClassesWithDexForRelease'.
> com.android.build.api.transform.TransformException: java.lang.RuntimeException: java.lang.RuntimeException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: com.android.ide.common.process.ProcessException: Return code 1 for dex process

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

```

OK, *something* went catastrophically wrong here. It's tempting to search for the answer on something like StackOverflow and try to copy and paste the answer in a desperate attempt to get *something* to work. 

The problem with this approach, however, is the fact that Gradle errors are build errors that came from your specific project, so while it might seem sensible just to Google the answer, the problem is particular to the individual project, and what may work for one person is unlikely to going to work for another (unless they have a similar setup). 

One could keep Googling until they find someone else with a similar setup, but I'll show how I solved it for my project, which has numerous dependencies being a React Native project and some custom written dependencies due to the nature of the project. With this approach, you can at least go on the path to solving your Gradle based issues.

I got this error because I ran `./gradlew assembleRelease`, but luckily Gradle has a verbose mode and can spit out stack traces when it fails. So to start the investigation, I ran `./gradlew assembleRelease --debug --stacktrace`.

I got a massive trace out of it (to be expected), but I scrolled up to the point of seeing the previous message:

~~~
Dex: Error converting bytecode to dex:
Cause: Dex cannot parse version 52 byte code.
This is caused by library dependencies that have been compiled using Java 8 or above.
If you are using the 'java' gradle plugin in a library submodule add
targetCompatibility = '1.7'
sourceCompatibility = '1.7'
~~~

and I found the following **above** it:

~~~
16:09:02.565 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/BundleJSONConverter$6.class...
16:09:02.566 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/BundleJSONConverter$7.class...
16:09:02.566 [INFO] [org.gradle.api.Project] Dexing in-process : /Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-sdk-api/16.4.0/jars/classes.jar
16:09:02.566 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/BundleJSONConverter$Setter.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRLocalMessagingHelper.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRLocalMessagingPublisher.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRMessagingModule.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRMessagingModule$1.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRMessagingModule$2.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRMessagingModule$3.class...
16:09:02.567 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRMessagingModule$4.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRMessagingPackage.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/FIRSystemBootEventReceiver.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/InstanceIdService.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/MessagingService.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/MessagingService$1.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/MessagingService$1$1.class...
16:09:02.568 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/evollu/react/fcm/SendNotificationTask.class...
16:09:02.568 [DEBUG] [org.gradle.api.Project] writing classes.dex; size 35780...
16:09:02.568 [INFO] [org.gradle.api.Project] Dexing /Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/***/react-native-fcm/unspecified/jars/classes.jar took 42.66 ms.
16:09:02.569 [INFO] [com.android.build.gradle.internal.ApplicationTaskManager] predex called for /Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-api/16.4.0/jars/classes.jar
16:09:02.569 [INFO] [org.gradle.api.Project] AndroidBuilder::preDexLibrary /Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-api/16.4.0/jars/classes.jar
16:09:02.569 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing archive /Users/soheb/.gradle/caches/modules-2/files-2.1/com.google.auto.value/auto-value-annotations/1.6/da725083ee79fdcd86d9f3d8a76e38174a01892a/auto-value-annotations-1.6.jar...
16:09:02.570 [INFO] [org.gradle.api.Project] preDexLibrary : DxDexKey{dexKey=DxDexKey{buildTools=27.0.3, sourceFile=/Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-api/16.4.0/jars/classes.jar, mJumboMode=false, mOptimize=true}, mAdditionalParameters=[], mIsMultiDex=false}
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/...
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/MANIFEST.MF...
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/maven/...
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/maven/com.google.auto.value/...
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/maven/com.google.auto.value/auto-value-annotations/...
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/maven/com.google.auto.value/auto-value-annotations/pom.properties...
16:09:02.570 [INFO] [org.gradle.api.Project] StoredItem is null
16:09:02.570 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing META-INF/maven/com.google.auto.value/auto-value-annotations/pom.xml...
16:09:02.571 [INFO] [org.gradle.api.Project] Item from cache Item{mOutputFiles=[], mSourceFile=/Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-api/16.4.0/jars/classes.jar}
16:09:02.571 [INFO] [org.gradle.api.Project] AndroidBuilder::preDexLibraryNoCache /Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-api/16.4.0/jars/classes.jar
16:09:02.571 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/...
16:09:02.571 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/google/...
16:09:02.571 [INFO] [org.gradle.api.Project] Dexing in-process : /Users/soheb/Code/Work/***/android/app/build/intermediates/exploded-aar/com.google.android.gms/play-services-measurement-api/16.4.0/jars/classes.jar
16:09:02.571 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/google/auto/...
16:09:02.571 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/google/auto/value/...
16:09:02.571 [DEBUG] [com.android.build.gradle.internal.ApplicationTaskManager] processing com/google/auto/value/AutoAnnotation.class...
16:09:02.572 [ERROR] [org.gradle.api.Project] Dex: Error converting bytecode to dex:
Cause: Dex cannot parse version 52 byte code.
This is caused by library dependencies that have been compiled using Java 8 or above.
If you are using the 'java' gradle plugin in a library submodule add 
~~~

So from the above stack trace, it looks like we have some culprits:

* `auto-value-annotations` (1.6)
* `google-play-gms` (16.4.0)
* `react-native-fcm` (from `package.json`)

I look around, starting with the one that seems closest to the crash (`auto-value-annotations`) and there appears to be an [issue](https://github.com/google/auto/issues/655) wherein build in 1.6 that pulls in Java 8 bytecode, breaking compatibility.

If we follow the GitHub ticket, we can see the issue referenced in [another ticket](https://github.com/google/auto/pull/656), and the [pull request](https://github.com/google/auto/commit/9cc04ecb39166207a2835174b85ee209cc08aad0) is merged (referenced as `9cc04ec` - the first seven characters of the git commit ID). We can see in [this release](https://github.com/google/auto/releases/tag/auto-value-1.6.3) that the issue is resolved (2nd from the bottom). The git commit linked is the version of AutoValue we want our code to pull in.

But how do we tell Gradle that we want this version whenever dependencies ask for `auto-value-annotations`? Gradle has a feature that allows it to enforce a resolution strategy. After a bit of googling of what that was (it is ubiquitous for Google Play APIs as libraries often have a higher/lower version of Google Play APIs than you are using) I came up with the chunk of Groovy code to put in my Gradle file (just above the `dependencies` entry - it should be on the same indentation level as `dependencies`):

~~~gradle
configurations.all {
    resolutionStrategy.eachDependency { DependencyResolveDetails details ->
        if (details.requested.group == 'com.google.auto.value') {
            details.useVersion '1.6.3'
        }
    }
}
~~~

Running `./gradlew assembleRelease` finds that this issue has been resolved. If it had not been resolved, then I would have run `./gradlew app:dependencies` to find the list of dependencies and any conflicts that might arise and attempt to fix them using the above resolution strategy, or setting dependencies to certain versions and using `force = true` to enforce a certain version across the entire app.