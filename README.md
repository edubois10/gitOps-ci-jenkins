## Setting up Jenkins

1. Create a new project where jenkins will be deployed.

    ``` bash
    oc new-project jenkins-ci
    ```

2. Create a persistent Jenkins instance with:

    * A persistent volume claim wit at elast 4GB.
    * CPU request 500m and CPU limit of 2 cores.
    * Memory request of 1Gi and memory limit of 2Gi.

    ``` bash
    oc new-app jenkins-persistent --param ENABLE_OAUTH=true --param MEMORY_LIMIT=2Gi --param VOLUME_CAPACITY=4Gi --param DISABLE_ADMINISTRATIVE_MONITORS=true --as-deployment-config=true
    ```

    ``` bash
    oc set resources dc jenkins --limits=memory=2Gi,cpu=2 --requests=memory=1Gi,cpu=500m
    ```

### Building Jenkins slave agent image
3. Build the image locally
    ``` bash
    # Be sure do be in the right path
    podman build -f ./Dockerfile -t jenkins/slave-image .
    ```

4. Copy the image to your registry
    ``` bash
    skopeo copy --dest-tls-verify=false containers-storage:localhost/jenkins/slave-image docker://<Your Registry>
    # Example
    skopeo copy --dest-tls-verify=false containers-storage:localhost/jenkins/slave-image docker://quay.io/edubois10/jenkins-slave-image
    ```

### Register a Jenkins agent pod

5. Create a file with the pod agent definition(see jenkins/jenkins-slave-xml).
    
6. Create ConfigMap with the agent definition file podTemplate.xml.
    ``` bash
    oc create configmap maven-skopeo-agent --from-file=./jenkins/jenkins-slave-xml -n jenkins-ci
    ```

7. Label the ConfigMap so that Jenkins know it's the definition for the agent.
    ``` bash
    oc label configmap serv-agent role=jenkins-slave
    ```