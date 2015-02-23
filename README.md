External Configuration Properties with Jetty
============================================

Demonstration of how to configure simple properties that can be accessed
by Servlets within Jetty.

This demonstration shows 4 different ways to configure a property
at runtime, that can then be read by the Servlet running within
Jetty.

The props.war
-------------

This is a simple war file with a single HttpServlet and a WEB-INF/web.xml

```
[jetty-external-config]$ jar -tvf target/props.war 
     0 Mon Feb 23 09:02:14 MST 2015 META-INF/
   131 Mon Feb 23 09:02:14 MST 2015 META-INF/MANIFEST.MF
     0 Mon Feb 23 09:02:14 MST 2015 WEB-INF/
     0 Mon Feb 23 09:02:14 MST 2015 WEB-INF/classes/
     0 Mon Feb 23 09:02:14 MST 2015 WEB-INF/classes/org/
     0 Mon Feb 23 09:02:14 MST 2015 WEB-INF/classes/org/eclipse/
     0 Mon Feb 23 09:02:14 MST 2015 WEB-INF/classes/org/eclipse/demo/
  2188 Mon Feb 23 09:02:12 MST 2015 WEB-INF/classes/org/eclipse/demo/PropsServlet.class
   572 Mon Feb 23 08:45:22 MST 2015 WEB-INF/web.xml
```

See [PropsServlet.java](src/main/java/org/eclipse/demo/PropsServlet.java) for details of behavior.

Just compile the top level and the war file will be built and placed in all of the demo jetty.base locations for this project.

Example #1: Basic Command Line
------------------------------

The `/base-command-line` project contains a simple `start.ini` which starts jetty on port 9090, and deploys the webapp. no extra configuration is done by the on-disk configuration.

If you start it up like so ...

```
[base-command-line]$ java -jar /path/to/jetty-distribution-9.2.7.v20150116/start.jar
2015-02-23 09:15:46.088:INFO::main: Logging initialized @290ms
2015-02-23 09:15:46.222:INFO:oejs.Server:main: jetty-9.2.7.v20150116
2015-02-23 09:15:46.235:INFO:oejdp.ScanningAppProvider:main: Deployment monitor [file:/home/joakim/code/stackoverflow/jetty-external-config/base-command-line/webapps/] at interval 1
2015-02-23 09:15:46.325:INFO:oejw.StandardDescriptorProcessor:main: NO JSP Support for /props, did not find org.eclipse.jetty.jsp.JettyJspServlet
2015-02-23 09:15:46.343:INFO:oejsh.ContextHandler:main: Started o.e.j.w.WebAppContext@6e7f61a3{/props,file:/tmp/jetty-0.0.0.0-9090-props.war-_props-any-27537844855769703.dir/webapp/,AVAILABLE}{/props.war}
2015-02-23 09:15:46.353:INFO:oejs.ServerConnector:main: Started ServerConnector@67cd35c5{HTTP/1.1}{0.0.0.0:9090}
2015-02-23 09:15:46.353:INFO:oejs.Server:main: Started @555ms
```

you'll see that it has started up and deployed to the `/props` context path.

From here you can test for properties in the servlet via tooling like `wget` or `curl`.

Example:

```
$ curl http://localhost:9090/props/props

[java.runtime.name] = Java(TM) SE Runtime Environment
[sun.boot.library.path] = /home/joakim/java/jvm/jdk-7u75-x64/jre/lib/amd64
[java.vm.version] = 24.75-b04
[java.vm.vendor] = Oracle Corporation
[java.vendor.url] = http://java.oracle.com/
...
[file.separator] = /
[java.vendor.url.bug] = http://bugreport.sun.com/bugreport/
[sun.io.unicode.encoding] = UnicodeLittle
[sun.cpu.endian] = little
[sun.desktop] = gnome
[sun.cpu.isalist] = 

```

You can even request a specific property ..

```
$ curl http://localhost:9090/props/props/user.timezone

[user.timezone] = America/Phoenix
```

Lets stop the server and run it with a system property of our choice.

Notice the `-Dfoo=bar` ?

```
[base-command-line]$ java -Dfoo=bar -jar /path/to/jetty-distribution-9.2.7.v20150116/start.jar
2015-02-23 09:15:46.088:INFO::main: Logging initialized @290ms
2015-02-23 09:15:46.222:INFO:oejs.Server:main: jetty-9.2.7.v20150116
2015-02-23 09:15:46.235:INFO:oejdp.ScanningAppProvider:main: Deployment monitor [file:/home/joakim/code/stackoverflow/jetty-external-config/base-command-line/webapps/] at interval 1
2015-02-23 09:15:46.325:INFO:oejw.StandardDescriptorProcessor:main: NO JSP Support for /props, did not find org.eclipse.jetty.jsp.JettyJspServlet
2015-02-23 09:15:46.343:INFO:oejsh.ContextHandler:main: Started o.e.j.w.WebAppContext@6e7f61a3{/props,file:/tmp/jetty-0.0.0.0-9090-props.war-_props-any-27537844855769703.dir/webapp/,AVAILABLE}{/props.war}
2015-02-23 09:15:46.353:INFO:oejs.ServerConnector:main: Started ServerConnector@67cd35c5{HTTP/1.1}{0.0.0.0:9090}
2015-02-23 09:15:46.353:INFO:oejs.Server:main: Started @555ms
```

and look for it via curl ...

```
$ curl http://localhost:9090/props/props/foo

[foo] = bar
```

That demonstrates access of a property that was specified via the command line, now lets look at the other choices.


Example #2: Using start.ini 
------------------------------

The `/base-startini` project contains a simple `start.ini` which starts jetty on port 9090, and deploys the webapp.

This `start.ini` also contains a `foo.ish` property.

Lets start up Jetty and try our props servlet access again ...


```
[base-startini]$ java -jar /path/to/jetty-distribution-9.2.7.v20150116/start.jar
2015-02-23 09:16:46.088:INFO::main: Logging initialized @290ms
2015-02-23 09:16:46.222:INFO:oejs.Server:main: jetty-9.2.7.v20150116
```

and request it via curl ...

```
$ curl http://localhost:9090/props/props/foo.ish

[foo.ish] = bar
```

Example #3: Using start.d optional ini
--------------------------------------

The `/base-startd` project contains a simple `start.ini` which starts jetty on port 9090, and deploys the webapp.

This `start.ini` also contains no extra properties that we are interested in.

The `start.d/myconf.ini` contains a property called `foo.d` that we are interested in.

Lets start up Jetty and try our props servlet access again ...


```
[base-startd]$ java -jar /path/to/jetty-distribution-9.2.7.v20150116/start.jar
2015-02-23 09:19:46.088:INFO::main: Logging initialized @290ms
2015-02-23 09:19:46.222:INFO:oejs.Server:main: jetty-9.2.7.v20150116
```

and request it via curl ...

```
$ curl http://localhost:9090/props/props/foo.d

[foo.d] = over here
```

Example #4: Using --include-jetty-dir optional config
-----------------------------------------------------

The `/base-jettyinclude` project contains a new `start.ini` which starts jetty on port 9090, and deploys the webapp.

This `start.ini` also contains no extra properties that we are interested in.

However, the `start.ini` uses the `--include-jetty-dir=../jettydir` optional configuration that points to an entirely new interrim jetty.base configuration source.

The `../jettydir/start.ini` contains a property called `foo.jetty.dir` that we are interested in.

Lets start up Jetty and try our props servlet access again ...


```
[base-jettyinclude]$ java -jar /path/to/jetty-distribution-9.2.7.v20150116/start.jar
2015-02-23 09:24:46.088:INFO::main: Logging initialized @290ms
2015-02-23 09:24:46.222:INFO:oejs.Server:main: jetty-9.2.7.v20150116
```

and request it via curl ...

```
$ curl http://localhost:9090/props/props/foo.jetty.dir

[foo.jetty.dir] = more of the same
```











