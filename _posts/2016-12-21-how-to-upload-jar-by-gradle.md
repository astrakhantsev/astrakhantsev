---
title: 'How to upload (Scala) jar to JCenter and MavenCentral with Gradle'
excerpt: "After the library creation you usually want to upload it to central repositories; in case of gradle, it can be non-trivial, so I've prepared a brief how-to."
permalink: /how-to-upload-jar-by-gradle/
tags:
  - Tools
  - Scala
---

_This is a purely technical post, more like an instruction/tutorial._

Assume that you built you project (written Scala, but it is relevant only for one point, see below) with gradle and now you want to upload jar, so that anyone can use it by maven/gradle/sbt/whatever -- even ant, altough I used with manually collected jars.

It is well known that there are 2 main repos: mavenCentral and jcenter. 
_They remind me Python 2 and Python 3, respectively: everyone says that Python 3 is better, but if you really want your lib to be used by everyone, use Python 2 -- or, ideally, write ["code that runs under both Python2 and 3"](https://wiki.python.org/moin/PortingToPy3k/BilingualQuickRef) (so that you have to take worst from both worlds)._ 

Fortunately, [bintray team](https://bintray.com/) provides easy way to achieve both repos without much efforts.

Unfortunately, their original [documentation](https://bintray.com/docs/usermanual/uploads/uploads_promotingyourmaterial.html#_including_your_package_in_jcenter) describes evident cases, but omits what is really incomprehensible, e.g. they link to the start page of maven (however, in [gradle plugin page](https://github.com/bintray/gradle-bintray-plugin) they provide much more useful [link](http://central.sonatype.org/pages/requirements.html)), so I decided to clarify it a little.

## Brief how-to

1. Go to [OSS](https://sonatype.org/), register, create issue for your future artifact
That guy warns that mavnecentral site waits for approving, but in my cases it took less than 10 minutes for both reactions.
2. Go to [bintray](https://bintray.com/), register there and create repo (this is repo for artifacts, not your repo for code).
3. Add code to build.gradle - see comments below or just copy-paste examples from [pu4spark](https://github.com/ispras/pu4spark/blob/master/build.gradle) or [atr4s](https://github.com/ispras/atr4s/blob/master/build.gradle). Also see how to hide credentials  at [stackoverflow answer](http://stackoverflow.com/questions/38501402/hide-credentials-for-all-projects-in-build-gradle).
4. Run `gradle bintrayUpload`.
5. Go to [bintray](https://bintray.com/) again, press Sync with JCenter. Wait for some time (usually, hours).
6. Sync with mavenCentral.

*Also take a look at this nice [guide](https://inthecheesefactory.com/blog/how-to-upload-library-to-jcenter-maven-central-as-dependency/en) with a lot of screenshots; it is designed for Android, but the first part remains valid for any jars.

## Brief comments to build.gradle

Note that bintray's signing can be used, _(if you are more lazy than paranoid)_.

MavenCentral doesn't pass files without javadocs (see the almost-5-year-old [issue](https://issues.sonatype.org/browse/OSSRH-2932)) and they don't care if you are using Scala.
There are [projects](https://github.com/typesafehub/genjavadoc) for generating javadocs,
but much simpler seems to be just replace javadocs by scaladocs (see [example](https://github.com/ispras/pu4spark/blob/master/build.gradle#L23)).

Include of `mavenCentralSync` section, which was in [original guide](https://github.com/bintray/gradle-bintray-plugin), seems to be useless, because you need to manually sync package with jcenter first.

Also be careful with `pom.xml` generation:
I didn't research this bug, but if you move any of 3 `append` lines to `pomConfig` method, i.e. replace them by lines like the following: 
```
description 'A library for Positive-Unlabeled Learning for Apache Spark MLlib (ml package)'
```
then gradle fails to add description tag to `pom.xml` for no reason!
Even if we place something before description in pomConfig, it still won't shown.






