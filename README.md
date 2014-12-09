# Java SE OpenShift cartridge

This cartridge can be used to execute any java code, is was designed initially for Spring boot, but now it supports any java server code.

Your java code must open port 8080 and be able to process http requests.

This first version supports.

* java markers
* source and binary deployment
* java debug
 
Soon we will improve this documentation also we will include examples.

## Installation:

### From GITHUB:
* **rhc:** execute:
        
        rhc app create -a <app_name> -t https://github.com/Produban/ose_cartridge_javase.git
        
* **Consola web:** Al crear la aplicación, añadir _https://github.com/Produban/ose_cartridge_javase.git_ 
en el bloque **Code Anything**. DespuÃ©s seguir el como si fuera cualquier otro cartucho.

### From OSE
 If you use OSE I recommend to deploy the cartridge using oo-admin-cartridge
 
* **rhc:** ejecutar:
        
        rhc app create -a <app_name> -t javase
        
* **Consola web:** Al crear la aplicaciÃ³n, seleccionar el cartucho **Java SE** y proceder como cualquier otro cartucho.

## Despliegue:

### Desde fuentes:
1. Clonar la aplicaciÃ³n a tu mÃ¡quina local. 
        rhc git-clone -a <app_name>

2. Editar los fuentes bajo el directorio src. Editar el fichero pom.xml si se necesita agregar una nueva dependencia.

    > Hay que tener en cuenta que para que OpenShift pueda desplegar el artefacto tiene que colocarse 
    en el directorio **apps** y llamarse **application.jar**. 
    Por defecto el pom.xml realiza una tarea para copiar el artefacto generado al directorio **apps** y lo renombra.

3. Enviar los cambios a OpenShift:
  
        git add .
        git commit -am "my message"
        git push

### Binario
1. Configurar el gear:
  
        rhc app configure -a xxx1 --deployment-type binary --no-auto-deploy [--keep-deployments 5]

1. Crear la estructura de despliegue binario:
        
        rhc snapshot save -a xxx1 --deployment
        tar -xvf xxx1.tar.gz 
        rm xxx1.tar.gz
        cd repo
        rm -rf pom.xml  README.md  src
        cd ..

1. Seleccionar la aplicaciÃ³n nueva:
        
        cp <local_path>/example.jar repo/apps/application.jar

1. Crear el empaquetado:
        
        tar -cvzf xxx1.tar.gz *
    
1. Desplegar:
        
        rhc deploy -a xxx1 xxx1.tar.gz

## Logs
Los logs de la aplicacikÃ³n quedan el en directorio **${OPENSHIFT_LOG_DIR}**. Comando para visualizarlos:
        
    rhc tail -a <app_name>

## GestiÃ³n de memoria
La configuraciÃ³n de la memoria heap del proceso java se gestiona mediante las variables de entorno **JVM_HEAP_RATIO** y **JVM_PERMGEN_RATIO**. Para cambiar el valor utilizar:

    rhc env set JVM_HEAP_RATIO=<NEW_RATIO> -a <app_name>
    
> Hay que reiniciar la aplicaciÃ³n para que los cambios se hagan efectivos.

## Propiedades JVM
* Setear variables:
        
        rhc env set -a <my_app> --env JAVA_OPTS_EXT="-D<my_key>=<my_value>"

* Listar variables:
        
        rhc env list -a <my_app> [--quote] [--table]
    
* Eliminar variables:
        
        rhc env unset -a <my_app> --env <my_key> [--confirm]
    
> Hay que reiniciar la aplicaciÃ³n para que los cambios se hagan efectivos.
