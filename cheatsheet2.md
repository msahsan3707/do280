<pre>
OpenShift - EX280 Exam
Red Hat EX280 exam study points with some notes for each of them.

Cluster version: V4.12
Exam version: EX280V412K
Manage OpenShift Container Platform
Use the web console to manage and configure an OpenShift cluster
Get the Web Console URL through oc login command

Login

  # login
  oc login -u user -p passwd https://api.ocp4.example.com:6443

  # Get the console url
  oc whoami --show-console
  https://console-openshift-console.apps.ocp4.example.com

  # Note: https://api.ocp4.example.com:6443 & https://console-openshift-console.apps.ocp4.example.com are
  # examples url from RedHat Lab
Here is the Web Console documentation.

Use the command-line interface to manage and configure an OpenShift cluster
You can manage an OpenShift cluster from the web console or by using the kubectl or oc command-line interfaces (CLI).

The kubectl commands are native to Kubernetes, and are a thin wrapper over the Kubernetes API.
The OpenShift oc commands are a superset of the kubectl commands, and add commands for the OpenShift-specific features

The main method of interacting with an RHOCP cluster is by using the oc command.

To install kubectl, follow this kubernetes documentation
To install oc download it from the web console to ensure that the CLI tools are compatible with the RHOCP cluster.
From the web console, navigate to Help → Command line tools.
Or append /command-line-tools in the Console Url
You can also download oc and openshit-installer previous versions on Mirror Repo
oc

  # login & contexts

  oc login $cluster_url
  oc login -u user -p passwd $cluster_url

  ## login with api token: generate & copy the token from the CLI download page
  oc login --token=sha256-xxx --server=$cluster_url

  oc config 
  oc config get-contexts

  # check cluster version - apis

  oc version
  oc cluster-info
  oc get clusterversion

  oc api-versions
  oc api-resources

  ## resources from core api group
  oc api-resources --api-group ''

  # projects

  ## create project
  oc new-project myapp

  ## Switch to the specific project
  oc projetc previous-projet

  ## display current project
  oc project

  # oc get

  oc get po
  oc status #Display the status of the containers in the selected namespace.

  oc get clusteroperators
  oc get operators
  oc get all # some resources like secrets, serviceaccounts are not displayed by all
  ...

  # For further options
  oc -h
Query, format, and filter attributes of Kubernetes resources
Filtering


You can use the same filters, formats as you used to do with kubectl

* `-l`
* `jq`
* `--sort-by`
* `-o jsonpath`
* and a subcommand specific flag ...
API resources - Filtering

  # filter api resources
  oc api-resources --namespaced
  oc api-resources --api-group '' # resources in the core api
  oc api-resources --api-group config.openshift.io

  oc get pods -A -l=app=olm-operator
  oc explain pod.spec

  oc get events -n openshift-image-registry --sort-by .metadata.creationTimestamp

  oc get node master01 -o json | jq '.status.conditions'
  oc get node master01 -o jsonpath={.status.conditions}

  oc get no master01 -o jsonpath='{.status.allocatable}{"\n"}'
  oc get no -o jsonpath='{range .items[*]}{.metadata.name}{": "}{.status.addresses[?(@.type=="InternalIP")]}{"\n"}{end}'
Import, export, and configure Kubernetes resources
As with kubectl you can use create/apply/patch subcommands

oc create/apply/run/patch

  oc create -f
  oc run 
  oc apply
  oc patch
Locate and examine container images
Locate - registries

Red Hat distributes container images by using two registries:

registry.access.redhat.com where no authentication is required
registry.redhat.io where authentication is required).
You can also use another registries, your own or public ones.

Inspect container images with skopeo

Various tools can inspect and manage container images, including the oc image command and skopeo.

Skopeo is another tool to inspect and manage remote container images. With skopeo, you can copy and sync container images from different container registries and repositories.

To install skopeo, follow the install.md

skopeo login/list-tags/inspect

  # log in registry where authentification is reguired
  skopeo login $registry

  # list available tags of an image
  skopeo list-tags docker://registry.access.redhat.com/ubi9/httpd-24
      {
          "Repository": "registry.access.redhat.com/ubi9/httpd-24",
          "Tags": [
              "1-229",
              "1-217.1666632462",
              "1-201",
              "1-194.165519"]

  # inspect an image
  skopeo inspect docker://registry.access.redhat.com/ubi8:latest
      {
          "Name": "registry.access.redhat.com/ubi8",
          "Digest": "sha256:70fc...1173",
          "RepoTags": [
              "8.7-1054-source",
              "8.6-990-source",
              "8.6-754",
              "8.4-203.1622660121-source",
      ...output omitted...

  # inspect an image with config option to show the image config field
  skopeo inspect --config docker://registry.ocp4.example.com:8443/redhattraining/docker-nginx:1.23
      ...output omitted...
          "config": {
              "ExposedPorts": {
                  "80/tcp": {}
              },
              "Env": [
                  "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                  "NGINX_VERSION=1.23.3",
                  "NJS_VERSION=0.7.9",
                  "PKG_RELEASE=1~bullseye"
              ],
              "Entrypoint": [
                  "/docker-entrypoint.sh"
              ],
              "Cmd": [
                  "nginx",
                  "-g",
                  "daemon off;"
              ],
              "Labels": {
                  "maintainer": "NGINX Docker Maintainers \u003cdocker-maint@nginx.com\u003e"
              },
              "StopSignal": "SIGQUIT"
          }

  # inspect with format option

  skopeo inspect --format \
  "Name: {{.Name}}\n Digest: {{.Digest}}\n Release: {{.Labels.release}}" \
  docker://registry.ocp4.example.com:8443/rhel9/mysql-80:latest

  Name: registry.redhat.io/rhel9/mysql-80
  Digest: sha256:d282...f38f
  Release: 237
  ...

  # copy an image from a SRC registry to a DST registry
  skopeo copy docker://quay.io/skopeo/stable:latest docker://registry.example.com/skopeo:latest

  # Sync an image between two locations
  skopeo sync --src docker --dest docker registry.access.redhat.com/ubi8/httpd-24 registry.example.com/httpd-24

  # delete an image
  skopeo delete docker://registry.example.com/skopeo:latest
Inspect - retrieve information about image with oc image

The oc image info command inspects and retrieves information about a container image.
You can use the oc image info command to identify the ID/hash SHA and to list the image layers of a container image.

oc image info/append/mirror

  oc image info registry.access.redhat.com/ubi9/httpd-24:1-233 --filter-by-os amd64

      Name:          registry.access.redhat.com/ubi9/httpd-24:1-233
      Digest:        sha256:4186...985b
      ...output omitted...
      Image Size:    130.8MB in 3 layers
      Layers:        79.12MB sha256:d74e...1cad
                  17.32MB sha256:dac0...a283
                  34.39MB sha256:47d8...5550
      OS:            linux
      Arch:          amd64
      Entrypoint:    container-entrypoint
      Command:       /usr/bin/run-httpd
      Working Dir:   /opt/app-root/src
      User:          1001
      Exposes Ports: 8080/tcp, 8443/tcp
      Environment:   container=oci
      ...output omitted...
                  HTTPD_CONTAINER_SCRIPTS_PATH=/usr/share/container-scripts/httpd/
                  HTTPD_APP_ROOT=/opt/app-root
                  HTTPD_CONFIGURATION_PATH=/opt/app-root/etc/httpd.d

  oc image append
  # to add layers to container images, and then push the container image to a registry.

  oc image extract
  # to extract or copy files from a container image to a local disk. 
  # Use this command to access the contents of a container image without first running the image as a container.

  oc image mirror
  # copy or mirror container images from one container registry or repository to another
Create and delete projects
oc new-project/project

  # create project
  oc new-project test-proj

  # show current project
  oc project

  # delete project
  oc  delete project test-proj
Examine resources and cluster status
Operators

  oc get clusteroperators
  oc describe clusteroperators openshift-apiserver
  oc get node master01 -o jsonpath={.status.conditions}

  oc adm top po -A --sum
View logs
Logs - Crictl

  oc logs $mypod

  oc adm node-logs master01 -u crio --tail 1
  oc adm node-logs master01 -u kubelet --since=2023-09-10 11:12:13'

  # you can ssh on a node and use critcl command
  crictl pods
  crictl ps
  crictl logs $container
  ...
Monitor cluster events and alerts
Events - Components

  oc get events -A
  oc get events -n openshift-image-registry --sort-by .metadata.creationTimestamp

  # check monitoring stack logs
  oc get all -n openshift-monitoring --show-kind
      NAME                                              READY  STATUS   RESTARTS AGE
      pod/alertmanager-main-0                           6/6    Running  85       34d
      pod/cluster-monitoring-operator-56b769b58f-dtmqj  2/2    Running  34       35d
      pod/kube-state-metrics-75455b796c-8q28d           3/3    Running  51       35d
      ...output omitted...

  oc logs alertmanager-main-0 -n openshift-monitoring

      ts=2023-03-16T14:21:50.479Z caller=main.go:231 level=info msg="Starting Alertmanager" version="(version=0.24.0, branch=rhaos-4.12-rhel-8, revision=519cbb87494d2830821a0da0a657af69d852c93b)"

  # check cluster components errors/events
  oc get co
Assess the health of an OpenShift cluster
Health - Debug

  # Check cluster core components status
  oc get co

  # check clusteroperators conditions
  oc get clusteroperators
  oc describe clusteroperators xxx

  ## check a clusteroperator pods
  oc get po -n openshift-apiserver

  # check operators
  oc get operators
  oc get pod -n openshift-dns-operator dns-operator-64688bfdd4-8zklh -o json | jq .status

  # examining Cluster Metrics
  oc adm top po -A --sum

  # you can view Cluster Metrics on Web console

  # check Node Status
  oc get no
      NAME       STATUS   ROLES                         AGE   VERSION
      master01   Ready    control-plane,master,compute   35d   v1.25.4+77bec7a

  oc get node master01 -o json | jq '.status.conditions'

  oc get node master01 -o jsonpath=\
  *'{"Allocatable:\n"}{.status.allocatable}{"\n\n"}
  {"Capacity:\n"}{.status.capacity}{"\n"}'

  # check node logs
  oc adm node-logs master01 -u crio --tail 1

      -- Logs begin at Thu 2023-02-09 21:19:09 UTC, end at Fri 2023-03-17 15:11:43 UTC. --
      Mar 17 06:16:09.519642 master01 crio[2987]: time="2023-03-17 06:16:09.519474755Z" level=info msg="Image status:
      &ImageStatusResponse{Image:&Image{Id:6ef8...79ce,RepoTags:[],RepoDigests:
Troubleshoot common container, pod, and cluster events and alerts
containers - pods

Debug pod/node

  # the basics
  oc get po
  oc logs $mypod
  oc describe $mypod

  # check tag availability if ImagePullbackOff
  skopeo list-tags ...
  # edit/patch/describe pod
  oc edit $mypod
  oc describe $mypod
  oc patch

  # display the status of the containers in the selected namespace.
  oc status

  # create debug pod for $mypod
  oc debug pod/$mypod
  # start a remote shell in $mypod directly
  oc rsh $mypod
  # for further actions, use exec
  oc exec $mypod -- $mycmd
Find out more in troubleshooting documentation.

node - cluster

Node Logs - Debug

  oc adm node-logs master01
  oc adm node-logs master01 -u kubelet --tail 3
  # debug node
  oc debug node/master01
    # possible actions on node
      chroot /host
      systemctl status kubelet
      systemctl is-active crio

  # gather cluster debugg logs
  oc adm must-gather --dest-dir /home/student/must-gather

  # gather kube-apiserver logs
  oc adm inspect clusteroperator/kube-apiserver  --dest-dir /home/student/inspect --since 5m
Use product documentation
Take a look in the RHOCP official documentation.

Deploy Applications
Deploy applications from resource manifests
App

  # with oc new-app
    ## With local source
    oc new-app /<path to source code>

    oc new-app --file=./example/my-app.yaml

    ## With remote source
    oc new-app https://github.com/sclorg/cakephp-ex

    oc new-app https://github.com/openshift/ruby-hello-world --name=myapp

    oc new-app https://github.com/sclorg/s2i-ruby-container.git \
    --context-dir=3.0/test/puma-test-app

    ## From image
    oc new-app mysql

    oc new-app openshift/postgresql-92-centos7 \
    -e POSTGRESQL_USER=user \
    -e POSTGRESQL_DATABASE=db \
    -e POSTGRESQL_PASSWORD=password

    ## from template
    oc new-app --template=cache-service

  # with usual create/apply
  oc create -f my_manifest.yaml
  oc create -f application_temlate.json

  oc apply -f my_manifest.yaml
Use Kustomize overlays to modify application configurations
Kustomize is a configuration management tool to make declarative changes to application configurations and components and preserve the original base YAML files.

You group in a directory the Kubernetes resources that constitute your application and then use kustomize to copy and adapt these resource files to your environments and clusters.

Overlays

Kustomize overlays declarative YAML artifacts, or patches, that override the general settings without modifying the original files. The overlay directory contains a kustomization.yaml file.

To get find out more, check

Declarative Management of Kubernetes Objects Using Kustomize documentation
Kustomize Github repo
kustomize

  ## render the files/temples
  oc kustomize overlays/staging
      apiVersion: v1
      data:
      enable: "true"
      msg: Welcome!
      kind: ConfigMap
      metadata:
      name: hello-app-configmap-9tcmf95d77
      namespace: hello-stage
      ---
      apiVersion: apps/v1
      kind: Deployment
      ...output omitted...

  ## Apply files directly with -k option
  oc  apply -k overlays/staging
Deploy applications from images, OpenShift templates, and Helm charts
From images

Deploy App

  # with oc new-app
  oc new-app --name db-image -l team=blue --image registry.ocp4.example.com:8443/rhel9/mysql-80:1 \
    -e MYSQL_USER=developer \
    -e MYSQL_PASSWORD=developer \
    -e MYSQL_ROOT_PASSWORD=redhat

  # with oc run
  oc run example-pod \
  --image=registry.access.redhat.com/ubi8/httpd-24 \
  --env GREETING='Hello from the awesome container' \
  --port 8080

  # with oc create
  oc create deployment my-db  --image registry.ocp4.example.com:8443/rhel9/mysql-80:1
From templates

Deploy and update applications from resource manifests that are packaged as OpenShift templates.

A template is a Kubernetes custom resource that describes a set of Kubernetes resource configurations.

Templates can have parameters. You can create a set of related Kubernetes resources from a template by processing the template, and providing values for the parameters.

Find out more in templates documentation.

oc template/new-app

  oc get templates -n openshift
  oc describe template cache-service -n openshift

  # process template
  oc process my-cache-service -o yaml
  oc process my-cache-service -p TOTAL_CONTAINER_MEM=1024 -p APPLICATION_USER='cache-user' -o yaml
  ## with exisiting template file
  oc process -f my-cache-service-template.yaml 
  ## Use  --parameters option to view only the parameters that a template uses
  oc process --parameters cache-service -n openshift

  # create App from template with new-app
  oc new-app --template=cache-service
  oc new-app -l team=red --template mysql-persistent -p MYSQL_USER=developer -p MYSQL_PASSWORD=developer

  # create/upload new template from files
  oc create -f  /path/to/template/manifests.yml
helm Charts

Use usual helm commands. Find out more in Helm documentation

helm

  # list repo
  helm repo list
      Error: no repositories to show

  # add repo
  helm repo add do280-repo http://helm.ocp4.example.com/charts

  # list all charts version in the repo do280-repo
  helm search repo --version
  NAME                     CHART VERSION  APP VERSION  ...
  do280-repo/etherpad      0.0.7          latest       ...
  do280-repo/etherpad      0.0.6          latest       ...
  ...output omitted..

  # install do280-repo/etherpad      0.0.6
  ## examine the chart variables
  helm show values do280-repo/etherpad --version 0.0.6

  ## create values.yml with variables
  ## install the release
  helm install example-app do280-repo/etherpad -f values.yaml --version 0.0.6

  ## upgrade to the 0.0.7
  helm upgrade example-app do280-repo/etherpad -f values.yaml --version 0.0.7

  # view the history of release
  helm history example-app
Deploy jobs to perform one-time tasks
Jobs

  oc create job date-job --image registry.access.redhat.com/ubi8/ubi -- /bin/bash -c "date"

  oc create job date-loop --image registry.ocp4.example.com:8443/ubi9/ubi \
  -- /bin/bash -c "for i in {1..30}; do date; done"

  oc get jobs
Manage application deployments
Deployment

  # with oc create
  oc create deployment my-db  --image registry.ocp4.example.com:8443/rhel9/mysql-80:1

  oc get deploy,po
Work with replica sets
Use oc scale deployment/example --replicas=3

Work with labels and selectors
Label - Selector Filtering


  Use usual `-l`, `-L` to filter the resources
Configure services
Services

  # create a service
  oc create service -h
  oc create service clusterip mysvc --tcp=8080:8080 --dry-run=server -o yaml

  # expose an existing workload
      ## create workload
  oc create deployment sakila-app --image registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
  oc expose deployment sakila-app --name sakila-svc --port 8080 --target-port 8080
      service/sakila-svc exposed

  oc create deployment satir-app --image registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
  oc expose deployment satir-app --name satir-svc --port 8080 --target-port 8080
  oc get svc,ep
Expose both HTTP and non-HTTP applications to external access
Route and Ingress are the main resources for handling ingress traffic.

When you create an ingress object, an asscociated route(with no assigned port) is also created

Expose

  # create route by exposing  an existing service
  oc expose service satir-svc --name satir
      route.route.openshift.io/satir exposed
      # if you omit the hostname, the default is: <route-name>-<project-name>.<default-domain>.
  oc get routes
      NAME    HOST/PORT                                    ... SERVICES    PORT   ...
      satir   satir-web-applications.apps.ocp4.example.com ... satir-svc   8080   ...

  # create an ingress object
  oc create ingress ingr-sakila --rule "ingr-sakila.apps.ocp4.example.com/*=sakila-svc:8080"
      ingress.networking.k8s.io/ingr-sakila created

  oc get ingress
      NAME        ... HOSTS                              ADDRESS       PORTS ...
      ingr-sakila ... ingr-sakila.apps.ocp4.example.com  router...com  80    ...

  oc get routes
      NAME           HOST/PORT                                    ... SERVICES   PORT
      ingr-sakila... ingr-sakila.apps.ocp4.example.com            ... sakila-svc <all>
      satir          satir-web-applications.apps.ocp4.example.com ... satir-svc  8080

  # create a route
  oc create route -h
  oc create route edge myroute --service=mysvc_name --port 8080
Annotate route & ingress to add features like sticky sessions

Routes Annotations

  oc annotate ingress ingr-sakila ingress.kubernetes.io/affinity=cookie
      ingress.networking.k8s.io/ingr-sakila annotated

  oc annotate route satir router.openshift.io/cookie_name="hello"
      route.route.openshift.io/satir annotated
Expose non-HTTP/SNI Applications
You can use service types LoadBalancer or NodePort to expose your non-HTTP application

Expose - Loadbalancer

  # expose a deployment as Loadbalancer service type
  oc expose deployment/virtual-rtsp-1 --target-port=8554 --type=LoadBalancer
  oc get services
    NAME            TYPE          CLUSTER-IP   EXTERNAL-IP    PORT(S)         AGE
    virtual-rtsp-1  LoadBalancer  172.30.4.18  192.168.50.20  8554:32170/TCP  59
Work with operators such as MetalLB and Multus
MetalLB
MetalLB is a load balancer component that provides a load balancing service for clusters that do not run on a cloud provider, such as a bare metal cluster,
or clusters that run on hypervisors. MetalLB operates in two modes: layer 2 and Border Gateway Protocol (BGP).

MetalLB is an operator that you can install with the Operator Lifecycle Manager. After installing the operator, you must configure MetalLB through
its custom resource definitions. In most situations, you must provide MetalLB with an IP address range.

More information on MetalLB in the documentation

Multus Secondary Networks
Expose applications to external access by using a secondary network.

The Multus CNI (container network interface) plug-in helps to attach pods to custom networks.
These custom networks can be either existing networks outside the cluster, or custom networks that are internal to the cluster.

Configuring Secondary Networks

To use existing custom networks, first you must make available the network on cluster nodes.

You can use operators, such as the Kubernetes NMState operator or the SR-IOV (Single Root I/O Virtualization) network operator, to customize node network configuration.
With these operators, you define custom resources to describe the intended network configuration, and the operator applies the configuration.

The SR-IOV network operator configures SR-IOV network devices for improved bandwidth and latency on certain platforms and devices.

Attaching Secondary Network

To configure secondary networks, create a NetworkAttachmentDefinition resource. Alternatively, update the configuration of the cluster network operator to add a secondary network

NetworkAttachmentDefinition

  apiVersion: k8s.cni.cncf.io/v1
  kind: NetworkAttachmentDefinition
  metadata:
    name: example
  spec:
    config: |-
      {
        "cniVersion": "0.3.1",
        "name": "example",
        "type": "host-device",
        "device": "ens4",
        "ipam": {
          "type": "static",
          "addresses": [
            {"address": "192.168.51.10/24"}
          ]
        }
      }
Pod Annotations:

Network attachment resources are namespaced, and are available only to pods in their namespace.
When the cluster has additional networks, you can add the k8s.v1.cni.cncf.io/networks: your-secondary-network-name annotation to the pod's template to use one of the additional networks.

Network Annotations

  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: example
    namespace: example
  spec:
    selector:
      matchLabels:
        app: example
        name: example
    template:
      metadata:
        annotations:
          k8s.v1.cni.cncf.io/networks: example  ### add annotation in template section
        labels:
          app: example
          name: example
      spec:
  ...output omitted...
Show status

  oc get pod example \
    -o jsonpath='{.metadata.annotations.k8s\.v1\.cni\.cncf\.io/networks-status}'
  [{
      "name": "ovn-kubernetes",
      "interface": "eth0",
      "ips": [
          "10.8.0.59"
      ],
      "mac": "0a:58:0a:08:00:3b",
      "default": true,
      "dns": {}
  },{
      "name": "non-http-multus/example",  # custom network
      "interface": "net1", 
      "ips": [
          "1.2.3.4"
      ],
      "mac": "52:54:00:01:33:0a",
      "dns": {}
  }]
Take a look in Multiple Networks documentation.

Manage Storage for Application Configuration and Data
Create and use secrets
Create the secrets like you used to do with kubectl

oc create secret

  # from literal
  oc create secret generic mysec  --from-literal key1=secret1 --from-literal key2=secret2
  # from env file
  oc create secret generic mysec  --from-env-file /path/to/file.env
  # ...

  # tls secret
  oc create secret tls mysec-tls --cert /path-to-certificate --key /path-to-key
Use Secrets in imperative mode with existing deployment

  # mount the secret as volume in existing deployment
  oc set volume deployment/demo --add --type secret --secret-name mysec --mount-path /app-secrets
  ## with args shortname and volume name
  oc set volume deployment/demo --add -t secret --secret-name mysec -m  /app-secrets  --name added-name

  # remove volume 
  oc set volume deployment/demo --remove --name=added-name --containers=c1

  # inject secret as  env variables in existing deployment
  oc set env deployment/demo  --from secret/mysec --prefix MYSQL_
Update Secrets

As well as oc edit secret/xx you can use oc extract and oc set data secret/xx to update your secret

Extract - Set Data

  # extract data to /tmp/demo 
  oc extract secret/demo-secrets -n demo --to /tmp/demo --confirm
  ls /tmp/demo/
  user  root_password

  # update root_password
  echo xxx > /tmp/demo/root_password

  # apply the password change
  oc set data secret/demo-secrets -n demo --from-file /tmp/demo/root_password
Create and use configuration maps
Create the configmaps like you used to do with kubectl

oc create configmap

  # from literal
  kubectl create configmap myconfig --from-literal key1=config1 --from-literal key2=config2

  # from env file
  oc create configmap myconfig-env --from-env-file /path/to/file.env

  # from file
  oc create configmap myconfig_f --from-file /path/to/config-files/httpd.conf
Use configmap in imperative mode with existing deployment

  # mount the configmap in existing deployment
  oc set volume deployment/demo --add --type configmap --configmap-name demo-map --mount-path /app-secrets --name myvol

  # To confirm that the volume is attached to the deployment
  oc set volume deployment/demo
      demo
      configMap/demo-map as myvol
          mounted at /app-secrets

  # remove the volume
  oc set volume deployment/demo --remove --name myvol
You can add this annotation configmap.reloader.stakater.com/reload: <configmap_name> in your deployment,
so that the controller can roll out deployments automaticall when the config-app configuration map changes.

Provision Persistent Storage volumes for block and file-based data
To add a pvc/pv volume to an deployment, use the oc set volumes as well

Volumes

  oc set volumes deployment/example-application \
  --add \
  --name example-pv-storage \     # volume name
  --type persistentVolumeClaim \  # -t pvc
  --claim-mode rwo \
  --claim-size 15Gi \
  --claim-name example-pv-claim   # PVC name
  --mount-path /var/lib/example-app \ 7

  # mount existing pvc
  oc set volume deployment/existing-pvc 
  --add \
  --name exisiting-pvc-vol
  --claim-name my-exisiting-pvc
  --mount-path /var/tmp

  # claim mode
  # rwo : ReadWriteOnce
  # ROX: readOnlyMany
  # RWX: ReadWriteMany
Here is the Persistent Storage documentation.

Use storage classes
Add the option --claim-class when you mount a pvc with oc set volumes.

Volumes - StorageClass

  oc set volumes deployment/db-pod \
    --add --name odf-lvm-storage --type pvc \
    --claim-mode rwo --claim-size 1Gi --mount-path /var/lib/mysql \
    --claim-class lvms-vg1 \  # storageclass
    --claim-name db-pod-odf-pv
Manage non-shared storage with StatefulSets
Add a volumeClaimTemplates block in your statefulset manifest

volumeClaimTemplate

    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: dbserver
    spec:
      selector:
        matchLabels:
          app: database
      replicas: 2
      template:
        metadata:
          labels:
            app: database
        spec:
          terminationGracePeriodSeconds: 10
          containers:
          - name: dbserver
            image: registry.ocp4.example.com:8443/redhattraining/mysql-app:v1
            ports:
            - name: database
              containerPort: 3306
            env:
            - name: MYSQL_USER
              value: "redhat"
            - name: MYSQL_PASSWORD
              value: "redhat123"
            - name: MYSQL_DATABASE
              value: "sakila"
            volumeMounts:    # mount volume
            - name: data
              mountPath: /var/lib/mysql
      volumeClaimTemplates:  # add volumeclaimtemplate
      - metadata:
          name: data
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "lvms-vg1"
          resources:
            requests:
              storage: 1Gi
Configure Applications for Reliability
Explore how the restartPolicy attribute affects crashing pods.
Deployment scalability with replicas or oc scale
Configure and use health probes
Health probes are an important part of maintaining a robust cluster. Probes enable the cluster to determine the status of an application by repeatedly probing it for a response.

A set of health probes affect a cluster's ability to do the following tasks:

Crash mitigation by automatically attempting to restart failing pods
Failover and load balancing by sending requests only to healthy pods
Monitoring by determining whether and when pods are failing
Scaling by determining when a new replica is ready to receive requests
Probe Types

A readiness probe determines whether the application is ready to serve requests.

If the readiness probe fails, then Kubernetes prevents client traffic from reaching the application by removing the pod's IP address from the service resource.
A liveness probe is called throughout the lifetime of the application.

Liveness probes determine whether the application container is in a healthy state.
Types of Tests: HTTP GET, Conatiner Command, TCP socket

Probes

  # add probes in existing deployment
  ## -c/--containers flag can be added in command below to specify a container

  ## HTTP GET
  oc set probe deploy/long-load --readiness --get-url http://:3000/health --failure-threshold 1 --period-seconds 3

  ## Container Command
  oc set probe deploy/long-load --liveness -- ping -c3 localhost

  ### TCP socket
  oc set probe  deploy/db --readiness --open-tcp=3306

  # remove liveness & readiness
  oc set probe deploy/myapp --remove --readiness --liveness
Reserve and limit application compute capacity
Set resources requests/limits
Verify nodes, pods resources usage
Resources - Requests/Limits

  ## add resources requests/limits
  oc set resources deployment hello-world-nginx --requests cpu=10m,memory=1gi --limits cpu=300m,memory=2gi

  ## verify resources usage
  oc adm top pods -A --sum
  oc adm top node mynode

  ## view totoal memory/cpu requets for a node
  oc describe node master01
Scale applications to meet increased demand
Configure a horizontal pod autoscaler: HPA

HPA

  oc autoscale deployment/hello  --name hello-hpa --min 1 --max 10 --cpu-percent 80
  oc get hpa

  # configure hpa based on memory
  ## oc autoscale don't provide option to create hpa based on memory usage
  ## you should create a manifest
  oc explain hpa.spec
  oc explain hpa.spec.metrics.resource

  ##e.g scales up when the average memory usage is above 60% of the memory requests value, and scales down when the usage is below this percentage.
  ## min 1 and max 3
      apiVersion: autoscaling/v2
      kind: HorizontalPodAutoscaler
      metadata:
      name: myapp-hpa
      labels:
          app: myapp
      spec:
      maxReplicas: 3
      minReplicas: 1
      scaleTargetRef:
          apiVersion: apps/v1
          kind: Deployment
          name: myapp
      metrics:
      - type: Resource
        resource:
          name: memory
          target:
              type: Utilization
              averageUtilization: 60
Manage Application Updates
Identify images using tags and digests
Use oc image info & skopeo command to list image tags and identify the image digest.
See previous section: Locate and examine container images

On the cluster node, you can use crictl images, crictl images --digests to list the locally available images.

Roll back failed deployments
Use oc rollout command.

Rollout

  # rollback to the preceding version
  oc rollout undo deployment/myapp
  oc rollout status deployment/myapp

  # list available revisions
  oc rollout history deployment/myapp

  # rollback to a specific revision
  oc rollout undo deployment/myapp2 --to-revision $revision_number

  # for imperatives changes, you can pause the deplyment, do the changes and then resume
  # to avoid having multiple revisions or failed pods
  ## pause
  oc rollout pause deployment/mydb

  ## perform the changes
  oc set env deployment/mydb MYSQL_PASSWORD=redhat123
  oc set image deployment/mydb mysql-80=registry.ocp4.example.com:8443/rhel9/mysql-80:1-228

  ## resume
  oc rollout resume deployment/mydb
Manage image streams
Image streams are one of the main differentiators between OpenShift and upstream Kubernetes. Kubernetes resources reference container images directly,
but OpenShift resources, such as deployment configurations and build configurations, reference image streams.

Image streams provide a stable, short name to reference a container image that is independent of any registry server and container runtime configuration.

Image Stream Tags

An image stream represents one or more sets of container images. Each set, or stream, is identified by an image stream tag.

Unlike container images in a registry server, which have multiple tags from the same image repository (or user or organization),
an image stream can have multiple image stream tags that reference container images from different registry servers and from different image repositories.

imageStream - Istag

  # list openshift namespace imagestream (is)
  oc get is -n openshift -o name
  ...output omitted...
  imagestream.image.openshift.io/nodejs
  imagestream.image.openshift.io/perl
  imagestream.image.openshift.io/php  #
  ...output omitted...

  # list imagestreamtag(istag) of is php
  oc get istag -n openshift | grep php
  8.0-ubi9      image-registry ...    6 days ago
  8.0-ubi8      image-registry ...    6 days ago
  7.4-ubi8      image-registry ...    6 days ago
  7.3-ubi7      image-registry ...    6 days ago

  # create an imagestream
  oc create is keycloak

  # create an imagestreamtag
  oc create istag keycloak:20.0  --from-image quay.io/keycloak/keycloak:20.0.2
  oc create istag keycloak:19.0 --from-image quay.io/keycloak/keycloak:19.0
  ## oc create istag will create the imagestream(is) if it doesn't exist yet

  # list the istag
  oc get istag
  # oc get istag display the target image SHA ID not its tag, you can get the tag from the istag attributes
  oc get istag keycloak:19.0 -o jsonpath='{.tag.from.name}{"\n"}'

  # create or update an imagestreamtag to point to new image/tag with oc tag
  ##  # will create the istag keycloak:20.0 if it doesn't exist or update it
  oc tag quay.io/keycloak/keycloak:20.0.3 keycloak:20.0

  ## oc tag has:
  ## --scheduled : to periodically sync the image SHA ID  between the istag and the image source
  ## --reference-policy local : to locally cache the image after the first pull

  # create an alias of istag with oc tag
  ## # keycloak:20 is an alias of keycloak:20.0.2 (source)
  oc tag --alias keycloak:20.0.2 keycloak:20

  # you can also use oc import-image to create or update an istag
  oc import-image keycloak:20.0.2 --from quay.io/keycloak/keycloak:20.0.2 --confirm

  #  like oc create istag, both oc tag and oc import-image will also create the the imagestream(is) if it doesn't exist yet
Using Image Streams in Deployments

When you create a Deployment object, you can specify an image stream instead of a container image from a registry.
Using an image stream in Kubernetes workload resources, such as deployments, requires preparation:

Create the image stream object in the same project as the Deployment object.
Enable the local lookup policy in the image stream object.
In the Deployment object, reference the image stream tag by its name, such as keycloak:20.0, and not by the full image name from the source registry.
When you use an image stream in a Deployment object, OpenShift looks for that image stream in the current project.
However, OpenShift searches only the image streams that you enabled the local lookup policy for

Use the oc set image-lookupcommand to enable the local lookup policy for an image stream

Image-lookup

  # enable local lookup
  oc set image-lookup keycloak

  oc describe is keycloak
      Name:             keycloak
      Namespace:        myproject
      Created:          3 hours ago
      Labels:           <none>
      Annotations:      openshift.io/image.dockerRepositoryCheck=2023-01-31T11:12:44Z
      Image Repository: image-registry.openshift-image-registry.svc:5000/.../keycloak
      Image Lookup:     local=true # local lookup enabled
      Unique Images:    3
      Tags:             2
      ...output omitted...

  # disable local lookup
  oc set image-lookup keycloak --enabled=false

  # list the local lookup status of all is
  oc set image-lookup
      NAME          LOCAL
      keycloak      true
      zabbix-agent  false
      nagios        false

  # use the is as normal image to deploy a workload
  oc create deployment mykeycloak --image keycloak:20.0
More Information on Image Streams in the documentation.

Use triggers to manage images
Automatic Image Updates with OpenShift Image Change Triggers

Image stream tags record the SHA ID of the source container image. Thus, an image stream tag always points to an immutable image.

If a new version of the source image becomes available, then you can change the image stream tag to point to that new image.
However, a Deployment object that uses the image stream tag does not roll out automatically.

For an automatic rollout, you must configure the Deployment object with an image trigger with oc set triggers command.

oc set triggers

  oc get deployment mykeycloak

  # enable image trigger
  oc set triggers deployment/mykeycloak --from-image keycloak:20 --containers keycloak

  # check that a trigger is enabled
  # The true value under the AUTO column for image indicates that the trigger is enabled.
  oc set triggers deployment/mykeycloak
      NAME                    TYPE    VALUE                   AUTO
      deployments/mykeycloak  config                          true
      deployments/mykeycloak  image   keycloak:20 (keycloak)  true

  # disable the image trigger by adding the --manual flag
  oc set triggers deployment/mykeycloak --from-image keycloak:20 --containers keycloak --manual

  # re-nable the image trigger by adding --auto flag
  oc set triggers deployment/mykeycloak --from-image keycloak:20 --containers keycloak --auto

  # remove omage trigger from all containers
  oc set triggers deployment/mykeycloak --remove-all
Manage Authentication and Authorization
Authenticating with the X.509 Certificate

During installation, the OpenShift installer creates a unique kubeconfig file in the auth directory.
The kubeconfig file contains specific details and parameters for the CLI to connect a client to the correct API server, including an X.509 certificate.

The installation logs provide the location of the kubeconfig file:
INFO Run 'export KUBECONFIG=root/auth/kubeconfig' to manage the cluster with 'oc'.

You can use it to authentificate oc commands.

KUBECONFIG

  export KUBECONFIG=/home/user/auth/kubeconfig
  oc get nodes
  oc --kubeconfig /home/user/auth/kubeconfig get nodes
Authenticating with the kubeadmin Virtual User

After installation completes, OpenShift creates the kubeadmin virtual user.
The kubeadmin secret in the kube-system namespace contains the hashed password for the kubeadmin user. It has cluster administrator privileges.

The OpenShift installer dynamically generates a unique kubeadmin password for the cluster.
The installation logs provide the kubeadmin credentials to log in to the cluster.

Access Details - kubeadmin

  ...output omitted...
  INFO The cluster is ready when 'oc login -u kubeadmin -p shdU_trbi_6ucX_edbu_aqop'
  ...output omitted...
  INFO Access the OpenShift web-console here:
      https://console-openshift-console.apps.ocp4.example.com
  INFO Login to the console with user: kubeadmin, password: shdU_trbi_6ucX_edbu_aqop
Deleting the Virtual User

After you define an identity provider, create a user, and assign that user the cluster-admin role, you can remove the kubeadmin user credentials to improve cluster security.

Delete kubeadmin secret

   oc delete secret kubeadmin -n kube-system
Configure the HTPasswd identity provider for authentication
The HTPasswd identity provider validates users against a secret that contains usernames and passwords that are generated with the htpasswd command.

Configuring the OAuth Custom Resource

To use the HTPasswd identity provider, the OAuth custom resource must be edited to add an entry to the .spec.identityProviders array:

oauth

  oc get oauth cluster -o yaml

    apiVersion: config.openshift.io/v1
    kind: OAuth
    metadata:
      name: cluster
    spec:
      identityProviders:
      - name: myusers  # provider name
        mappingMethod: claim # Controls how mappings are established between provider identities and user objects
        type: HTPasswd
        htpasswd:
          fileData:
            name: htpasswd-secret # An existing secret that contains data that is generated by using the htpasswd. Examples are just below
Updating the OAuth Custom Resource

To update the OAuth custom resource, use the oc get command to export the existing OAuth cluster resource to a file, update the file with the needed changes and recreate the resource with oc replace

oauth

  # get the oauth cluster resource
  oc get oauth cluster -o yaml > oauth.yaml

  # add changes
  vim oauth.yaml

  # apply changes
  oc replace -f oauth.yaml

  # check authentification pods
  oc get all -n openshift-authentication
Find out more in the Identity provider documentation.

Create and delete users
The httpd-tools package provides the htpasswd utility, which must be installed and available on your system.

Create users

Htpasswd - Secrets

  # htpasswd: create/update a user

  ## create the htpasswd file by creating student credential
  htpasswd -c -B -b /tmp/htpasswd student redhat123  # -c only when file doesn't exist yet ... to create

  ## add or update credential
  htpasswd -b /tmp/htpasswd student redhat1234
  htpasswd -b /tmp/htpasswd student4 toto1234

  # delete user credential from htpasswd file
  htpasswd -D /tmp/htpasswd student

  # create K8S secret that contains the  htpasswd data in "openshift-config" namespace
  ## !!! IMPORTANT !!! A secret that the HTPasswd identity provider uses requires adding the htpasswd= prefix before specifying the path to the file.

  oc create secret generic htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config
Update users list

Extract - Policy

  ## use oc extract to get data from the secret, update it and then update the secret object
  oc extract secret/htpasswd-secret -n openshift-config --to /tmp/ --confirm
      /tmp/htpasswd

  ## update the extracted /tmp/htpasswd file
  htpasswd -D /tmp/htpasswd user-to-delete
  htpasswd -b /tmp/htpasswd student new-password

  ## apply changes
  oc set data secret/htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config

  ## check authentificatio pods
  oc get all -n openshift-authentication

  # give cluster admin role to the user student
  oc adm policy add-cluster-role-to-user cluster-admin student
Delete users

Delete Identity - User

  # delete the user from htpasswd file and update the secret: see "Update users list"  example section above

  # then delete user & identity  resources from the cluster
  oc get users
  NAME            UID                                   ...  IDENTITIES
  admin           6126c5a9-4d18-4cdf-95f7-b16c3d3e7f24  ...  ...
  new_admin       489c7402-d318-4805-b91d-44d786a92fc1  ...  myusers:new_admin
  new_developer   8dbae772-1dd4-4242-b2b4-955b005d9022  ...  myusers:new_developer

  ## delete new developer identity resource
  oc delete identity "myusers:new_developer"

  ## delete new_developer user
  oc delete user new_developer
Modify user passwords
See the Create and delete users step above.

Create and manage groups - remove self-provisioners
Groups - Policy

  # create group
  oc adm groups new mygroup_x

  # add user in the group
  oc adm groups add-user mygroup_x user1

  # assign role to group
  oc adm policy add-role-to-group edit mygroup_x

  # prevent users from creating projects in the cluster by removing self-provisioner roles
  ## assigned to all authenticated users by default
  ##
  oc adm policy remove-cluster-role-from-group  self-provisioners system:authenticated:oauth
  ## OR
  oc patch clusterrolebinding.rbac self-provisioners -p '{"subjects": null}'
  #
  ## edit to allow only specific groups/users
  oc edit clusterrolebinding self-provisioners
  ##
  ### then add annotation to protect ths rolebinding and make the change permanent
  ### otherwise If the API server restarts, then Kubernetes restores this cluster role binding.
  ##
  oc annotate clusterrolebinding/self-provisioners --overwrite rbac.authorization.kubernetes.io/autoupdate=false
  ##
  ### to remember the annotation name , use oc describe clusterrolebindings | grep auto

  # to give a permission to a specific project, switch to the project with oc project and then use add-role-to-group
  ## grand 'edit' role privilges on 'auth-review' project to the 'developers' group
  ##
  oc project auth-review
  oc policy add-role-to-group edit developers
Modify user and group permissions
See the examples above or use oc adm policy, add-role-to-group|add-cluster-role-to-group|remove-cluster-role-from-group to manage group permission.

Configure Network Security
Network documentation
Configure networking components
NetworkType - Cidr - ServiceNetwork

  oc get network.config.openshift.io cluster -o yaml

    apiVersion: config.openshift.io/v1
    kind: Network
    metadata:
      name: cluster
    spec:
      clusterNetwork:
      - cidr: 10.128.0.0/14
        hostPrefix: 23
      externalIP:
        policy: {}
      networkType: OVNKubernetes
      serviceNetwork:
      - 172.30.0.0/16
    status:
      clusterNetwork:
      - cidr: 10.128.0.0/14
        hostPrefix: 23
      clusterNetworkMTU: 1400
      networkType: OVNKubernetes
      serviceNetwork:
      - 172.30.0.0/16
Network Operator configuration - routingViaHost

  # To Enable `routingViaHost` , set it as `true` with
  oc edit networks.operator cluster
Troubleshoot software defined networking
Apply the usual methods, don't you think?

Create and edit external routes
Use oc create route edge and oc create route passthrough to create routes.

Find out more details in section the Secure external and internal traffic using TLS certificates below.

Control cluster network ingress
Something to add here ?

Secure external and internal traffic using TLS certificates
Securing Routes

Routes can be either secured or unsecured. Secure routes support several types of transport layer security (TLS) termination to serve certificates to the client.
A secured route specifies the TLS termination of the route. The following termination types are available:

Edge: With edge termination, TLS termination occurs at the router, before the traffic is routed to the pods. The router serves the TLS certificates, so you must configure them into the route; otherwise, OpenShift assigns its own certificate to the router for TLS termination.

Passthrough: With passthrough termination, encrypted traffic is sent straight to the destination pod without TLS termination from the router. In this mode, the application is responsible for serving certificates for the traffic.

Re-encryption: is a variation on edge termination, whereby the router terminates TLS with a certificate, and then re-encrypts its connection to the endpoint, which might have a different certificate. Find out more here

Secure Routes

  # Securing Applications with Edge Routes

  ## With Edge termination generating certificate
  oc create route edge --service api-frontend --hostname api.apps.acme.com

  ## by passing your own certificate to the Edge termination
  oc create route edge --service api-frontend --hostname api.apps.acme.com --key api.key --cert api.crt

  # With Passthrough
  ## Create a TLS secret that contains the certificate
  ## Mount the certificate as volume in your pods/deployements...
  ## And assume your app is configured to use it

  oc create secret tls todo-certs --cert certs/training.crt --key certs/training.key

  ### mount secret as volume
  oc set volume deployment/todo-https  --add --type secret -secret-name todo-certs --mount-path /usr/local/etc/ssl/certs

  ### create passthrough route
  oc create route passthrough todo-https --service todo-https --port 8443 --hostname todo-https.apps.ocp4.example.com
Find out more in Configuring Routes documentation.
To learn more about SSl certificate generation, see SSL documentation.
Secure internal traffic
Service certificate

OpenShift provides the service-ca controller to generate and sign service certificates for internal traffic. The service-ca controller creates a secret that it populates with a signed certificate and key.

A deployment can mount this secret as a volume to use the signed certificate.Additionally, client applications need to trust the service-ca controller CA.

Find out more in service serving certificates documentation.

Service Certificate Creation

To generate a certificate and key pair, apply the service.beta.openshift.io/serving-cert-secret-name=your-secret annotation to a service.
The service-ca controller creates the your-secret secret in the same namespace if it does not exist, and populates it with a signed certificate and key pair for the service.

Annotations

  oc annotate service hello \ 1
      service.beta.openshift.io/serving-cert-secret-name=hello-secret 2
  service/hello annotated
After OpenShift has generated the secret, you must mount the secret in the application deployment. The location to place the certificate and key is application-dependent.

Mount TLS Certficate as volume
Client Service Application Configuration

For a client service application to verify the validity of a certificate, the application needs the CA bundle that signed that certificate.
The service-ca controller injects the CA bundle when you apply the service.beta.openshift.io/inject-cabundle=true annotation to an object.

You can apply the annotation to configuration maps, API services, custom resource definitions (CRD), mutating webhooks, and validating webhooks.

Example

  ## apply the annotation to a configmap
  oc create configmap ca-bundle

  oc annotate configmap ca-bundle \
      service.beta.openshift.io/inject-cabundle=true
  configmap/ca-bundle annotated

  ## mount the ca-bundle configmap in the client pod/deployment
  ## e.g pod
  apiVersion: v1
  kind: Pod
  metadata:
    name: client
  spec:
    containers:
      - name: client
        image: registry.ocp4.example.com:8443/redhattraining/hello-world-nginx
        resources: {}
        volumeMounts:
          - mountPath: /etc/pki/ca-trust/extracted/pem
            name: trusted-ca
    volumes:
      - configMap:
          defaultMode: 420
          name: ca-bundle
          items:
            - key: service-ca.crt
              path: tls-ca-bundle.pem
        name: trusted-ca
Alternatives to Service Certificates

Other options can handle TLS encryption inside an OpenShift cluster, such as a service mesh or the certmanager operator.

You can use the certmanager operator to delegate the certificate signing process to a trusted external service, and also to renew a certificate.

You can also use Red Hat OpenShift Service Mesh for encrypted service-to-service communication and for other advanced features.

Configure application network policies
There are the same as Kubernetes Network Policies

Here is Openshift related documentation

To allow traffic from only router/ingress pods, use the label network.openshift.io/policy-group: ingress to match ingress namespace

NetworkPolicy

  # The following network policy applies to all pods with the deployment="product-catalog" label in the network-1 namespace.
  # The network-2 namespace has the network=network-2 label.
  # The policy allows TCP traffic over port 8080 from pods whose label is role="qa" in namespaces with the network=network-2 label.

  kind: NetworkPolicy
  apiVersion: networking.k8s.io/v1
  metadata:
    name: network-1-policy
    namespace: network-1
  spec:
    podSelector:  1
      matchLabels:
        deployment: product-catalog
    policyTypes:
      - Ingress
    ingress:  2
    - from:  3
      - namespaceSelector:
          matchLabels:
            network: network-2
        podSelector:
          matchLabels:
            role: qa
      ports:  4
      - port: 8080
        protocol: TCP
Enable Developer Self-Service
Configure project quotas
You can choose one of the following option to create resourceQuota:

Navigate to Administration → ResourceQuotas to create a resource quota from the web console
or use oc create resourcequota commands
use a manifest
Quota

  oc get resourcequota
  oc get quota

  oc create resourcequota example --hard=count/deployment=1
  oc create quota example2 --hard=requests.cpu=2
Quota Template

  apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: memory
    namespace: example
  spec:
    hard:
      limits.memory: 4Gi
      requests.memory: 2Gi
    scopes: {}
    scopeSelector: {}
Configure cluster resource quotas
For example, a group of developers manages many namespaces. Namespace quotas can limit RAM usage per namespace.
However, a cluster administrator cannot limit total RAM usage by all workloads that the group of developers manages.

OpenShift introduces cluster resource quotas for those scenarios.

Cluster resource quotas follow a similar structure to namespace resource quotas. However, cluster resource quotas use selectors to choose which namespaces the quota applies to.

Quota

  apiVersion: quota.openshift.io/v1
  kind: ClusterResourceQuota
  metadata:
    name: example
  spec:
    quota: #  contains the quota definition. This key follows the structure of the ResourceQuota specification.
      hard:
        limits.cpu: 
    selector: # defines which namespaces the cluster resource quota applies to. 
      annotations: {}
      labels:
        matchLabels:
          kubernetes.io/metadata.name: example
You can also use oc create clusterresourcequota command to create clusterresourcequota

clusterresourcequota

  # create a resourcequota that will be applied to all projects/namespaces with the label  group=dev

  oc create clusterresourcequota example --project-label-selector=group=dev --hard=requests.cpu=10
    clusterresourcequota/example created
Configure project resource requirements
See resoureQuota and clusterresoureQuota sections

Configure project limit ranges
Cluster administrators can set resource quotas on namespaces. Namespace quotas limit the resources that workloads in a namespace use. Quotas address resource management at the cluster level.

Kubernetes users might have further resource management needs within a namespace.

Users might accidentally create workloads that consume too much of the namespace quota. These unwanted workloads might prevent other workloads from running.

Users might forget to set workload limits and requests, or might find it time-consuming to configure limits and requests. When a namespace has a quota, creating workloads fails if the workload does not define values for the limits or requests in the quota.

Kubernetes introduces limit ranges to help with these issues. Limit ranges are namespaced objects that define limits for workloads within the namespace.

LimitRange

  apiVersion: v1
  kind: LimitRange
  metadata:
    name: example
    namespace: example
  spec:
    limits:
    - default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 250m
        memory: 256Mi
      max:
        cpu: "1"
        memory: 1Gi
      min:
        cpu: 125m
        memory: 128Mi
      type: Container
More information on LimitRange in the documentation.

Configure project templates
OpenShift introduces projects to improve security and users' experience of working with namespaces. The OpenShift API server adds the Project resource type.

By using a template, cluster administrators can customize namespace creation. For example, cluster administrators can ensure that new namespaces have specific permissions, resource quotas, or limit ranges.

Planning a Project Template

You can add any namespaced resource to the project template. For example, you can add resources of the following types:

Roles and role bindings: to grant specific permissions in new projects.
Resource quotas and limit ranges: to ensure that all new projects have resource limits.
If you add resource quotas, then creating workloads requires explicit resource limit declarations. Consider adding limit ranges to reduce the effort for workload creation.
Network policies: to enforce organizational network isolation requirements.
Creating a Project Template

The oc adm create-bootstrap-project-template command prints a template that you can use to create your own project template.

This template has the same behavior as the default project creation in OpenShift. The template adds a role binding that grants the admin cluster role over the new namespace to the user who requests the project.

Project templates use the same template feature as the oc new-app command.

Bootstrap Template

  oc adm create-bootstrap-project-template -o yaml > output.yml

    apiVersion: template.openshift.io/v1
    kind: Template
    metadata:
      creationTimestamp: null
      name: project-request
    objects:
    - apiVersion: project.openshift.io/v1
      kind: Project
      metadata:
        annotations:
          openshift.io/description: ${PROJECT_DESCRIPTION}
          openshift.io/display-name: ${PROJECT_DISPLAYNAME}
          openshift.io/requester: ${PROJECT_REQUESTING_USER}
        creationTimestamp: null
        name: ${PROJECT_NAME}
      spec: {}
      status: {}
    - apiVersion: rbac.authorization.k8s.io/v1
      kind: RoleBinding
      metadata:
        creationTimestamp: null
        name: admin
        namespace: ${PROJECT_NAME}
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: admin
      subjects:
      - apiGroup: rbac.authorization.k8s.io
        kind: User
        name: ${PROJECT_ADMIN_USER}
    parameters:
    - name: PROJECT_NAME
    - name: PROJECT_DISPLAYNAME
    - name: PROJECT_DESCRIPTION
    - name: PROJECT_ADMIN_USER
    - name: PROJECT_REQUESTING_USER
You can modify the object list in output.yml to add the required resources for new namespaces.

Then, use the oc createcommand to create the template resource in the openshift-config namespace:

Create template

  oc create -f output.yml -n openshift-config
  template.template.openshift.io/project-request created
Apply the new created template as the default one

Edit the resource projects.config.openshift.io/cluster

Project

  oc edit projects.config.openshift.io cluster

  # add in the spec bloc this content:
  # projectRequestTemplate:
  #    name: project-request

  apiVersion: config.openshift.io/v1
  kind: Project
  metadata:
  ...output omitted...
    name: cluster
  ...output omitted...
  spec:
    projectRequestTemplate: ## here
      name: project-request ## and there

  # to retrieve projectRequestTemplate key
  oc get projects. # then keyboard tab to get projects.config.openshift.io among the list
  oc explain projects.config.openshift.io.spec

  # notice the restart of api-server pods
  oc get pod -n openshift-apiserver -w
Manage OpenShift Operators
The Cluster Version Operator (CVO) installs and updates cluster operators as part of the OpenShift installation and update processes.
The CVO provides cluster operator status information as resources of the ClusterOperator type: oc get clusteroperator

Operator Lifecycle Manager (OLM) helps users to install and update operators in a cluster. Operators that the OLM manages are also known as add-on operators, in contrast with cluster operators that implement platform services.

Take a look in Operators documentation.

Install an operator
Installing Operators with the Web Console

The OpenShift web console provides an interface to the Operator Lifecycle Manager (OLM). The OperatorHub page lists available operators and provides an interface for installing them.

The Install Operator Wizard:

Navigate to Operators → OperatorHub to display the list of available operators. The OperatorHub page displays operators, and has filters to locate operators.
Click an operator to display further information
Click Install to begin the Install Operator wizard
You can choose installation options in the Install Operator wizard.
Find out more in Installing from OperatorHub using the web console documentation.

Install Operators with the CLI
Installing Operators

To install an operator, you must perform the following steps:

Locate the operator to install.
Review the operator and its documentation for installation options and requirements.
Decide the update channel to use.
Decide the installation mode. For most operators, you should make them available to all namespaces.
Decide to deploy the operator workload to an existing namespace or to a new namespace.
Decide whether the Operator Lifecycle Manager (OLM) applies updates automatically, or requires an administrator to approve updates.
Create an operator group if needed for the installation mode.
Create a namespace for the operator workload if needed.
Create the operator subscription.
Review and test the operator installation.
Operator Resources

The OLM uses the following resource types:

Catalog source: Each catalog source resource references an operator repository. Periodically, the OLM examines the catalog sources in the cluster and retrieves information about the operators in each source.

Package manifest: The OLM creates a package manifest for each available operator. The package manifest contains the required information to install an operator, such as the available channels.

Operator group: Operator groups define how the OLM presents operators across namespaces.

Subscription: Cluster administrators create subscriptions to install operators.

Operator: The OLM creates operator resources to store information about installed operators.

Install plan: The OLM creates install plan resources as part of the installation and update process. When requiring approvals, administrators must approve install plans.

Cluster service version (CSV): Each version of an operator has a corresponding CSV. The CSV contains the information that the OLM requires to install the operator

When installing an operator, an administrator must create only the subscription and the operator group. The OLM generates all other resources automatically.

Install an Operator

Operators

  # if the operator doesn't require a specific namespace, USE openshift-operators
    ## otherwise create a new namespace, add labels/annotations if there are required by the operators
        oc create ns myoperator-ns

  # Determine whether you need to create an operator group. Operators use the operator group in their namespace. 
    ## Operators monitor custom resources in the namespaces that the operator group targets.
    ## The openshift-operators namespace contains a "global-operators" operator group.
    ### to create a new Operatorgroup, use this manifest

      apiVersion: operators.coreos.com/v1
      kind: OperatorGroup
      metadata:
        name: name
        namespace: namespace # namespace where to create the resource
      spec:
        targetNamespaces:  # list of namespaces that the operator monitors CRD
        - namespace

    ### list all operatorGroup
    oc get  OperatorGroup -A

  #  create a subscription
    ## with the operator web-terminal as example
    ## oc get packagemanifests
    NAME                      CATALOG                            AGE
    odf-lvm-operator          do280 Operator Catalog Red Hat     5d1h
    web-terminal              do280 Operator Catalog Red Hat     5d1
    ...
    ## describe web-terminal operator
    oc describe packagemanifest web-terminal -n openshift-marketplace
    ...output omitted...

  ## create the subscription  manifest
  apiVersion: operators.coreos.com/v1alpha1
  kind: Subscription
  metadata:
    name: web-terminal
    namespace: openshift-operators # namespace for the operator workload
  spec:
    channel: fast                 # update channel from  oc describe packagemanifest web-terminal -n openshift-marketplace
    name: web-terminal            # package manifest to subscribe to
    source: do280-catalog-redhat  # source catalog from the oc describe packagemanifest 
    installPlanApproval: Manual   #  install plan approval mode, either Automatic or Manual
    sourceNamespace: openshift-marketplace

  ## Create the resource
  oc create -f subscription.yaml
Full example with file-integrity-operator operator

  oc get packagemanifests
  NAME                      CATALOG                            AGE
  web-terminal              do280 Operator Catalog Red Hat     5d1h
  file-integrity-operator   do280 Operator Catalog Red Hat     5d1

  c describe packagemanifest file-integrity-operator
  ...output omitted...

  # creating namespace with some required labels
  ## create file namespace.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      labels:
        openshift.io/cluster-monitoring: "true"
        pod-security.kubernetes.io/enforce: privileged
      name: openshift-file-integrity

  oc create -f namespace.yaml

  # Create an operator group in the operator namespace.
  ## operator-group.yaml
    apiVersion: operators.coreos.com/v1
    kind: OperatorGroup
    metadata:
      name: file-integrity-operator
      namespace: openshift-file-integrity
    spec:
      targetNamespaces:
      - openshift-file-integrity

  oc create -f operator-group.yaml

  # Create the subscription in the operator namespace
  ## create subscription.yaml
    apiVersion: operators.coreos.com/v1alpha1
    kind: Subscription
    metadata:
      name: file-integrity-operator
      namespace: openshift-file-integrity
    spec:
      channel: "stable"
      installPlanApproval: Manual
      name: file-integrity-operator
      source: do280-catalog-redhat
      sourceNamespace: openshift-marketplace

  oc create -f subscription.yaml

  # Examine the operator resource that the OLM created.

  oc describe operator file-integrity-operator
  Name:         file-integrity-operator.openshift-file-integrity
  ...output omitted...
  Status:
    Components:
      Label Selector:
        Match Expressions:
          Key:       operators.coreos.com/file-integrity-operator.openshift-file-integrity
          Operator:  Exists
      Refs:
  ...output omitted...
        Kind:                    InstallPlan
        Name:                    install-4wsq6   # installplan to approve
        Namespace:               openshift-file-integrity
        API Version:             operators.coreos.com/v1alpha1
        Conditions:
          Last Transition Time:  2023-03-22T10:38:22Z
          Message:               all available catalogsources are healthy
          Reason:                AllCatalogSourcesHealthy
          Status:                False
          Type:                  CatalogSourcesUnhealthy
          Last Transition Time:  2023-03-22T10:38:21Z
          Reason:                RequiresApproval
          Status:                True
          Type:                  InstallPlanPending   # waiting for approval because we set installPlanApproval: Manual in subscription
        Kind:                    Subscription
        Name:                    file-integrity-operator
        Namespace:               openshift-file-integrity
  Events:                        <none>

  # Approve the installplan
  ## get installplan spec

  oc get installplan -n openshift-file-integrity install-4wsq6 -o jsonpath='{.spec}'
  {"approval":"Manual","approved":false,"clusterServiceVersionNames":["file-integrity-operator.v1.0.0"],"generation":1}

  ## approve the install
  oc patch installplan install-pmh78 --type merge -p '{"spec":{"approved":true}}' -n openshift-file-integrity
    installplan.operators.coreos.com/install-pmh78 patched

  # Examine the status again
  oc describe operator file-integrity-operator
  ...output omitted...
  Status:
    Components:
      Label Selector:
        Match Expressions:
          Key:       operators.coreos.com/file-integrity-operator.openshift-file-integrity
          Operator:  Exists
      Refs:
        ...output omitted...
        Conditions:
          Last Transition Time:  2023-01-26T18:21:03Z
          Last Update Time:      2023-01-26T18:21:03Z
          Message:               install strategy completed with no errors
          Reason:                InstallSucceeded
          Status:                True
          Type:                  Succeeded
        Kind:                    ClusterServiceVersion
        Name:                    file-integrity-operator.v1.0.0
        Namespace:               openshift-file-integrity
        ...output omitted...

  # Examine the workloads in the openshift-file-integrity namespace.
  oc get all -n openshift-file-integrity

  # switch to the operator namespace/project

  oc project openshift-file-integrity
  oc get csv
Delete an operator
To remove an operator from the cluster with the CLI , delete the subscription and cluster service version objects.

Uninstall an Operator

  ## Check the current version of the subscribed operator in the currentCSV field.

  oc get sub <subscription-name> -o yaml | grep currentCSV
  currentCSV: ...output omitted...

  # Delete the subscription object. Use the value obtained from the preceding command to delete the cluster service version object.
  oc delete sub <subscription-name>
  oc delete csv <currentCSV>

  # if needed, delete operatorGroup
Configure Application Security
Create service accounts
A service account is a Kubernetes object within a project. The service account represents the identity of an application that runs in a pod

ServiceAccount SA

  oc create sa  mysvc-account
  oc create serviceaccount  mysvc-account2
To learn more about SA, check the documentation.

Configure and manage service accounts
Use SA

  # set a servicaccount (sa) in exisiting deployment
  oc set serviceaccount deployment/<deployment-name> <service-account-name>

  # add the sa in your workload manifest directly
  ...
  spec:
        serviceAccountName: <service-account-name>
        containers:
  ...

  # check sa options
  oc explain sa
Run privileged applications
See security context constraints section below.

Manage and apply permissions using security context constraints
OpenShift provides security context constraints(SCCs), a security mechanism that limits the access from a running pod in OpenShift to the host environment.
SCCs control the following host resources:

Running privileged containers
Requesting extra capabilities for a container
Using host directories as volumes
Changing the SELinux context of a container
Changing the user ID
OpenShift provides the following default SCCs (Most pods that OpenShift creates use the restricted SCC):

anyuid
hostaccess
hostmount-anyuid
hostnetwork
node-exporter
nonroot
privileged
restricted
SCC

  oc get scc

  oc describe scc anyuid
    Name:           anyuid
    Priority:         10
    Access:
      Users:          <none>
      Groups:         system:cluster-admins
    Settings:
    ...output omitted

  oc describe pod console-5df4fcbb47-67c52 -n openshift-console | grep scc
    openshift.io/scc: restricted
Review a pod securityContext requirements

Container images that are downloaded from public container registries, such as Docker Hub, might fail to run when using the restricted SCC.

A container image that requires running as a specific user ID can fail because the restricted SCC runs the container by using a random user ID.
A container image that listens on port 80/443 can fail for a related reason. The random user ID that the restricted SCC uses cannot start a service that listens on a privileged network port (port < 1024)
Use the scc-subject-review subcommand to list all the security context constraints that can overcome the limitations of a container: oc get pod podname -o yaml | oc adm policy scc-subject-review -f -

Apply a SCC

To change the container to run with a different SCC, you must create a service account that is bound to a pod.

Attach SCC

# example workload

oc new-app --name gitlab --image registry.ocp4.example.com:8443/redhattraining/gitlab-ce:8.4.3-ce.0

## the pod fails to run with error
## Chef::Exceptions::InsufficientPermissions: Cannot create directory[/etc/gitlab] at /etc/gitlab due to insufficient permissions

# create the sa
## log back with an admin user
oc create sa gitlab-sa

# associate a SCC to the sa
## oc adm policy add-scc-to-user <SCC> -z <service-account>

oc adm policy add-scc-to-user anyuid -z gitlab-sa

# update the deployment gitlab with the sa gitlab-sa
## can login back as normal user
## oc set serviceaccount deployment/<deployment-name> <service-account-name>

oc set serviceaccount deployment/gitlab gitlab-sa
Create and apply secrets to manage sensitive information
Secrets

  oc create secret -h
  oc get secret

  # extract and update secret data
  oc extract secret/htpasswd-secret  --to /tmp/ --confirm
  oc set data secret/htpasswd-secret --from-file htpasswd=/tmp/htpasswd
Configure application access to Kubernetes APIs
To grant an application access to a Kubernetes API, take these actions:

Create an application service account.
Grant the service account access to the Kubernetes API.
Assign the service account to the application pods
App Access

  # create sa
    ## creating a dedicated project
  oc new-project configmap-reloader
  oc create sa configmap-reloader

  # creating a switching in new project
  oc new-project appsec-api

  # assign the "edit" cluster role to the "configmap-reload"(created in configmap-reloader) sa in current project appsec-api
  oc adm policy add-role-to-user edit system:serviceaccount:configmap-reloader:configmap-reloader --rolebinding-name=reloader-edit
  # system:serviceaccount:configmap-reloader: used to get  the sa configmap-reloader from the project configmap-reloader

  # add the sa in the deployment manifest
  serviceAccountName: configmap-reload
Configure Kubernetes CronJobs
You can use oc create cronjob to create the jobs or generate a manifest

Cronjob

  oc create cronjob test \
    --image=registry.access.redhat.com/ubi8/ubi:8.6 \
    --schedule='0 0 * * *' \
    -- curl https://example.com \
    --dry-run=client -o yaml

      apiVersion: batch/v1
      kind: CronJob
      metadata:
        creationTimestamp: null
        name: test
      spec:
        jobTemplate:
          metadata:
            creationTimestamp: null
            name: test
          spec:
            template:
              metadata:
                creationTimestamp: null
              spec:
                containers:
                - command:
                  - curl
                  - https://example.com
                  image: registry.access.redhat.com/ubi8/ubi:8.6
                  name: test
                  resources: {}
                restartPolicy: OnFailure
        schedule: 0 0 * * *
      status: {}
Update OpenShift
Update an OpenShift cluster
Upgrade Cluster: oc adm upgrade

  # prior update add-ons operators
  ## Be sure to update all operators that are installed through the OLM to the latest version before updating the OpenShift cluster.

  # get the current version
  oc get clusterversion
  NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
  version   4.12.0    True        False         43d     Cluster version is 4.12.0

  oc get clusterversion -o jsonpath='{.items[0].spec.channel}{"\n"}'
  stable-4.12
  # if needed change the channel in the clusterversion, e.g: stable-4.13

  # View the available updates and note the version number of the update to apply.
  oc adm upgrade
  Cluster version is 4.12.0

  Updates:

  VERSION IMAGE
  4.12.1   quay.io/openshift-release-dev/ocp-release@sha256:...
  ...output omitted..

  # Upgrade
  oc adm upgrade --to-latest=true
  oc adm upgrade --to=VERSION

  # check CVO status
  oc get clusterversion

  # check cluster operator status
  oc get clusteroperators
Failed Update : AdminAckRequired

Got this error when trying to upgrade from 4.12.0 to 4.12.35
To perform the ack for AdminAckRequired, update/patch the configmap admin-acks

AdminAckRequired

  # display the configmap content
  oc get cm admin-acks -n openshift-config   -o yaml

  ### E.g: acknowledge the upgrade from 4.12 (K8S 1.25) to 4.13 (K8S 1.26)
  oc -n openshift-config patch cm admin-acks --patch '{"data":{"ack-4.12-kube-1.26-api-removals-in-4.13":"true"}}' --type=merge
Upgrading a cluster using the CLI documentation
OLM upgrading operators documentation
Openshift Container Platform LifeCycle Policy
Identify deprecated Kubernetes API usage
Deprecated Apis

  oc get apirequestcounts
  # and check column REMOVEINRELEASE

  # filter columns with awk
  oc get apirequestcounts | awk '{if(NF==4){print $0}}'

  NAME                    REMOVEDINRELEASE   REQUESTSINCURRENTHOUR   REQUESTSINLAST24H
  cronjobs.v1beta1.batch
                              1.25               15                      44
  horizontalpodautoscalers.v2beta2.autoscaling
                              1.26               6                       30
  ...output omitted...
Deprecated API Alerts in OpenShift

OpenShift includes two alerts that are triggered when a workload uses a deprecated API version:

APIRemovedInNextReleaseInUse : this alert is triggered for APIs that OpenShift Container Platform will remove in the next release.
APIRemovedInNextEUSReleaseInUse : this alert is triggered for APIs that OpenShift Container Platform Extended Update Support (EUS) will remove in the next release.
Update OpenShift Operators
Subscription - InstallPlan

  # get the new version from the operator subscription
  oc get sub  web-terminal -n openshift-operators -o yaml

    ...output omitted...
    spec:
      channel: fast
      installPlanApproval: Manual
      name: web-terminal
      source: do280-catalog-redhat
      sourceNamespace: openshift-marketplace
      startingCSV: web-terminal.v1.5.1
    status:
    ...output omitted...
      conditions:
    ...output omitted...
      - lastTransitionTime: "2022-11-24T13:46:21Z"
        reason: RequiresApproval
        status: "True"
        type: InstallPlanPending
      currentCSV: web-terminal.v1.6.0   # the latest available version in the channel.
      installPlanGeneration:
      installPlanRef:
        apiVersion: operators.coreos.com/v1alpha1
        kind: InstallPlan
        name: install-72vnw   # installPlanRef for the latest version
        namespace: openshift-operators
        resourceVersion: "194989"
        uid: 8dc979fe-936f-475a-8977-36d210c4da98
      installedCSV: web-terminal.v1.5.1  # current version
    ...output omitted...
      state: UpgradePending

  # check the installplan content
  oc get installplan -n openshift-operators install-72vnw -o yaml

    apiVersion: operators.coreos.com/v1alpha1
    kind: InstallPlan
    ...output omitted...
    spec:
      approval: Manual
      approved: false
      clusterServiceVersionNames:
      - web-terminal.v1.6.0
      generation: 2
    status:
    ...output omitted...

  # Update the operator by patching the installplan
  oc patch installplan install-72vnw --type merge \
    --patch '{"spec":{"approved":true}}'

  installplan.operators.coreos.com/install-72vnw patched

<pre>
