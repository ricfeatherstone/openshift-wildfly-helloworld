# OpenShift Wildfly HelloWorld

blah, blah, ...


## Create CICD Project and Deploy Jenkins

```bash
oc new-project helloworld-cicd --display-name="HelloWorld - CICD"
oc new-app jenkins-ephemeral \
    -p JENKINS_IMAGE_STREAM_TAG=jenkins:2 \
    -p MEMORY_LIMIT=2Gi
```

If working in a proxied environment add the following environment variables with `-e x=y` so Jenkins can access GitHub

* https_proxy
* http_proxy
* no_proxy

## Create the ImageStream and BuildConfig for Application S2I Builds

```bash
oc project helloworld-cicd
oc process -f openshift/templates/helloworld.yaml | oc create -f -
```

If working in a proxied environment pass the following parameters when processing the template with `-p x=y` in order 
to be able to access Maven repos.

* HTTP_PROXY_HOST
* HTTP_PROXY_PORT

## Create the Projects for the Deployment Environments and Allow Access to the Jenkins Service Account

```bash
for i in dev test prod
do
    oc new-project helloworld-$i --display-name="HelloWorld - $i"
    oc policy add-role-to-user edit system:serviceaccount:helloworld-cicd:jenkins -n helloworld-$i
done
```

## Create the Jenkins Pipeline

```bash
oc project helloworld-cicd
oc new-build https://github.com/ricfeatherstone/openshift-wildfly-helloworld.git \
    --name=helloworld-pipeline \
    --strategy=pipeline \
    --context-dir="openshift/pipelines/e2e" 
```

## Cleanup 

```bash
for i in dev test prod cicd
do
    oc delete project helloworld-$i
done
```
