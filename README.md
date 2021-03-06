ClassloaderIsolation
====================

This is a test case to show how classloaders can be isolated in JBoss AS5 and 6.

#### I created 2 EJB projects(jars) `TestEJB1` and `TestEJB2`, and place a jar `A1.jar (version 1)` in the deployment directory. The classloader for `TestEJB1` and `A1` were scoped with same classloader domain and hence when I invoke the client `TestEJB1` picks the `A1.jar  (version 1)`. I have another `TestEJB2` which does not have any scoped classloader, I put `A1.jar (version 2)` inside `${profile}/lib` and when invoked the client, it picks the version 2. So in this way by isolating the classloader I was able to successfully have 2 different versions of jar in the same setup, without affecting their individual operations.


```
|-- TestEJB1.jar
|   |-- com
|   |   `-- test
|   |       |-- client
|   |       |   `-- TestSessionBeanClient.class
|   |       |-- TestSessionBean1.class
|   |       `-- TestSessionBean1Remote.class
|   |-- jboss-ejb-client.properties
|   `-- META-INF
|       |-- ejb-jar.xml
|       |-- jboss-classloading.xml                              <<<<<<   Here I scoped the classloader domain : "my.custom.classloading.scope"
|       `-- MANIFEST.MF


|-- TestEJB2.jar
|   |-- com
|   |   `-- test
|   |       |-- client
|   |       |   `-- TestSessionBeanClient.class
|   |       |-- TestSessionBean2.class
|   |       `-- TestSessionBean2Remote.class
|   `-- META-INF
|       |-- ejb-jar.xml
|       `-- MANIFEST.MF

```


#### Added a jar which has same classloader domain, to `deploy` directory  :

```

$ jar -tvf A1.jar
     0 Thu Apr 10 12:22:14 IST 2014 META-INF/
    68 Thu Apr 10 12:22:14 IST 2014 META-INF/MANIFEST.MF
     0 Thu Apr 10 12:20:34 IST 2014 com/
     0 Thu Apr 10 12:20:34 IST 2014 com/test/
     0 Thu Apr 10 12:20:34 IST 2014 com/test/nikhil/
   381 Thu Apr 10 11:36:36 IST 2014 com/test/nikhil/Test.class
   170 Thu Apr 10 12:21:46 IST 2014 META-INF/jboss-classloading.xml   <<<<< same domain : "my.custom.classloading.scope"
```

#### `jboss-classloading.xml` looks like below :

```

<classloading xmlns="urn:jboss:classloading:1.0"
	domain="my.custom.classloading.scope"
	export-all="NON_EMPTY"
	import-all="true"
	parent-first="false">
</classloading>

```


So here "TestEJB1.jar" and "A1.jar" have the same scope (A1.jar version 1), so when I run the EJB client it picks the version 1. At the same point of time I have kept the version 2 of the same jar inside "${profile}/lib" and when I run EJB client for "TestEJB2.jar" it picks the version 2 which is present in the lib.
