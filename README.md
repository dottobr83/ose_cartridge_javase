# Java SE OpenShift cartridge

This cartridge can be used to execute any java code, is was designed initially for Spring boot, but now it supports any java server code.

Your java code must open port 8080 and be able to process http requests.

This first version supports.

* java markers
* source and binary deployment
* java debug
 
Soon we will improve this documentation also we will include examples.

## Installation:

### From github:
* **rhc:** execute:
        
        rhc app create -a <app_name> -t https://github.com/Produban/ose_cartridge_javase.git
        
### From OSE
 If you use OSE I recommend to deploy the cartridge using oo-admin-ctl-cartridge
 
* **rhc:** ejecutar:
        
        rhc app create -a <app_name> -t javase
        

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
