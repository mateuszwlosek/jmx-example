# JMX - Example

Example of Monitoring pods on minishift with JMX.

Example application was developed in [Spring Boot](https://projects.spring.io/spring-boot/).  
[Minishift](https://www.okd.io/minishift/) was used to test monitoring particular pods.

## How to run the environment?

### Before you start
* Install JAVA - we will need jconsole from java
[Java](https://www.oracle.com/pl/java/technologies/javase-downloads.html)

* Install `Docker`  
[Docker CE INSTALLATION](https://docs.docker.com/install/linux/docker-ce/ubuntu/)  

* Download `minishift`  
[Installing minishft](https://docs.okd.io/3.11/minishift/getting-started/installing.html)  
[Minishift releases](https://github.com/minishift/minishift/releases)

* Set up virtualization environment for minishift  
[Setting Up the Virtualization Environment](https://docs.okd.io/3.11/minishift/getting-started/setting-up-virtualization-environment.html)

* Download `oc tools`  
[oc tools](https://www.okd.io/download.html)


### Start the environment
1. Build service:  
    `docker build . -t jmx-example`
2. Run minishift:  
    `./minishift start --show-libmachine-logs -v5`
3. Configure minishift:  
     `oc login -u system:admin`  
     `./oc adm policy add-cluster-role-to-user cluster-admin admin`  
     `./oc create is test-image-stream -n myproject`
4. Do port forwarding:  
     `oc login -u system:admin`  
     `oc project default`
     `oc get pods` - check exact name of a pod with `docker-registry` in name  
     `oc port-forward docker-registry-1-zxsxx 8089:8089`
5. Login to minishift docker registry with token from minishift console:  
     - While starting minishift, there will be an url in logs to access minishift console  
     - Login with admin username and any password  
     - Click on right upper corner icon and chose `Copy Login Command`  
     - Login command has been placed in clipboard, paste it in any text editor and extract token part from it  
     `docker login -u admin -p hp57zxkJ8jvxgtDMgDIXNDTCeBwlML_l_csut1eyYVk localhost:5000` - use that token in docker login command
5. Upload docker image:  
     `docker tag jmx-example localhost:5000/myproject/jmx-example`  
     `docker push localhost:5000/myproject/jmx-example`
6. Use uploaded docker image:  
     - Log in to minishift web console  
     - Choose `My Project` project  
     - Choose `Deploy Image`  
     - In `Image Stream Tag` choose `myproject` then `test` and `latest` (if nothing can be selected in third box, just click on it and press enter), then click `Deploy` 
7. Configure environment variables:  
       - JAVA_OPTS: `-Djava.rmi.server.hostname=localhost -Dcom.sun.management.jmxremote.port=8080 -Dcom.sun.management.jmxremote.rmi.port=8080 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false`
8. Do port forwarding to the pod:  
       `oc get pods` - get pod name  
       `oc port-forward test-4-zd42c 8080:8080` - use pod name for port forwarding  
       - Open `jconsole` (from java directory)  
       - Type `localhost:8080` in remote address box  

![jmx](https://user-images.githubusercontent.com/15820051/83907296-cc85b080-a72a-11ea-8f1f-0e3341ec9f7b.png)

Completely reset minishift:  
     `minishift delete --force --clear-cache`  
     `minishift stop`
