# Java SE OpenShift cartridge

This cartridge can be used to execute any java code, is was designed initially for Spring boot, but now it supports any java server code.

Your java code must open port 8080 and be able to process http requests.

This first version supports.

* java markers
* source and binary deployment
* java debug
* Thread dump

> IMPORTANT: In order to use this cartridge your application and dependecies must be packaged in a runnable JAR called **application.jar**.


## Installation:

### From github:
* **rhc:** execute:
        
        rhc app create -a <app_name> -t https://github.com/Produban/ose_cartridge_javase.git
        
### From OSE
 If you use OSE I recommend to deploy the cartridge using oo-admin-ctl-cartridge
 
Once javase cartridge is installed, we can generate new applications using this command:    

        rhc app create -a <app_name> -t javase

## How to create a runnable JAR.

You can create a runnable JAR using differents tools, in this example we have used eclipse.

1. Just right-click on your project folder (in Eclipse) and select Export

2. Then select Java -> Runnable Jar

3. You will be asked to choose the location of the jar file

4. Finally, select the class that has the Main method that you want to run and choose **Package dependencies with the Jar file** and click Finish.



    210  2014-12-10 19:07   META-INF/MANIFEST.MF
        0  2014-12-10 19:07   org/
        0  2014-12-10 19:07   org/eclipse/
        0  2014-12-10 19:07   org/eclipse/jdt/
        0  2014-12-10 19:07   org/eclipse/jdt/internal/
        0  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/
      978  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/JIJConstants.class
      714  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/JarRsrcLoader$ManifestInfo.class
     4735  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/JarRsrcLoader.class
     1505  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/RsrcURLConnection.class
     1841  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/RsrcURLStreamHandler.class
     1149  2014-12-10 19:07   org/eclipse/jdt/internal/jarinjarloader/RsrcURLStreamHandlerFactory.class
        0  2014-12-10 18:08   com/
        0  2014-12-10 18:08   com/produban/
        0  2014-12-10 18:08   com/produban/simplehttpserver/
     1198  2014-12-10 19:06   com/produban/simplehttpserver/SimpleHTTPServer$MyHandler.class
     1959  2014-12-10 19:06   com/produban/simplehttpserver/SimpleHTTPServer.class
    489883  2014-12-10 19:07   log4j-1.2.17.jar


This is the META-INF/MANIFEST.MF


    Manifest-Version: 1.0
    Rsrc-Class-Path: ./ log4j-1.2.17.jar
    Class-Path: .
    Rsrc-Main-Class: com.produban.simplehttpserver.SimpleHTTPServer
    Main-Class: org.eclipse.jdt.internal.jarinjarloader.JarRsrcLoader


## Application deployment:

### Source deployment:
1. Create a new application using this cartridge.
2. Clone the new application.
        rhc git-clone -a <app_name>

2. Add or change new source code into directorio src. In order to meet dependencies configure pom.xml file.

    > This cartridge only execute the jar file called application.jar, by default the pom.xml creates the artifac application.jar, if you do a binary deployment, you must build the application.jar file.
    
    
3. Send changes to OpenSHift:
  
        git add .
        git commit -am "my message"
        git push

### Binary deployment
1. Configure application for binary deployment:
  
        rhc app configure -a <app_name> --deployment-type binary --no-auto-deploy [--keep-deployments 5]

1. Create deployment package:
        
        rhc snapshot save -a [app_name] --deployment -f /tmp/[app_name].tgz
        tar -xvf /tmp/[app_name].tgz 
        rm /tmp/[app_name].tgz
        cd repo
        rm -rf pom.xml  README.md  src
        cd ..

The OSE binary deployment package should looks like this:



     ./
    ./build-dependencies
    ./build-dependencies/.m2
    ./dependencies/
    ./repo/
    ./repo/apps/
    ./repo/apps/.gitkeep
    ./repo/apps/application.jar
    ./repo/.openshift/
    ./repo/.openshift/config/
    ./repo/.openshift/config/.gitkeep
    ./repo/.openshift/README.md
    ./repo/.openshift/action_hooks/
    ./repo/.openshift/action_hooks/README.md
    ./repo/.openshift/cron/
    ./repo/.openshift/cron/daily/
    ./repo/.openshift/cron/daily/.gitignore
    ./repo/.openshift/cron/hourly/
    ./repo/.openshift/cron/hourly/.gitignore
    ./repo/.openshift/cron/weekly/
    ./repo/.openshift/cron/weekly/chronograph
    ./repo/.openshift/cron/weekly/jobs.deny
    ./repo/.openshift/cron/weekly/jobs.allow
    ./repo/.openshift/cron/weekly/chrono.dat
    ./repo/.openshift/cron/weekly/README
    ./repo/.openshift/cron/monthly/
    ./repo/.openshift/cron/monthly/.gitignore
    ./repo/.openshift/cron/README.cron
    ./repo/.openshift/cron/minutely/
    ./repo/.openshift/cron/minutely/.gitignore
    ./repo/.openshift/markers/
    ./repo/.openshift/markers/README.md
    ./repo/.openshift/markers/java7


> ***NOTE:*** ***./repo/apps/application.jar*** file must contains your java code and all the dependencies, this JAR must be a runnable JAR.

1. Copy your application to /apps/application.jar:
        
        cp [local_path]/[your_app].jar repo/apps/application.jar

1. Create new deployment package:
        
        tar -cvzf /tmp/[app_name].tgz *
    
1. Deploy new package:
        
        rhc deploy -a /tmp/[app_name].tgz


## Logs
This cartridge uses the standard log directory **${OPENSHIFT_LOG_DIR}**. 
        
    rhc tail -a <app_name>

## Java Heap management
There are two standard OpenShift environment variable that are used to change java heap size 

**JVM_HEAP_RATIO**

**JVM_PERMGEN_RATIO**

Both variable are ratios, it means that the heap size depends on the GEAR size.

    rhc env set JVM_HEAP_RATIO=<NEW_RATIO> -a <app_name>
    
> NOTE: You must restart applicacion if you change **JVM_HEAP_RATIO** and **JVM_PERMGEN_RATIO**.

## How to add new Java properties
The variable JAVA_OPTS_EXT is used to configure new properties or override default java properties.

* How to set JAVA_OPTS_EXT:
        
        rhc env set -a <my_app> --env JAVA_OPTS_EXT="-D<my_key>=<my_value>"

* List environment variables:
        
        rhc env list -a <my_app> [--quote] [--table]
    
* How to delete a variable:
        
        rhc env unset -a <my_app> --env <my_key> [--confirm]
    
> Restart the application if you change the JAVA_OPTS_EXT variable.
