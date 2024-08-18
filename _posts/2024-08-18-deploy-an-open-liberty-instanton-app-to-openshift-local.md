# How to deploy an Open Liberty InstantOn app to OpenShift Local

This guide steps you through the process of deploying a basic [Open Liberty InstantOn](https://openliberty.io/docs/latest/instanton.html) app to [OpenShift Local](https://developers.redhat.com/products/openshift-local/overview) running on your PC.

## Audience

The audience of this guide is anyone having an interest to deploy an Open Liberty InstantOn app to OpenShift.

While this guide installs to OpenShift Local on your PC, the same process can be followed when using a full installation of OpenShift.

The guide has been written with an admin in mind who may have little or no development experience. Me, I fit somewhere in-between.

## Pre-requisites

For this exercise you need to:

- [Install OpenShift Local](https://thechalkboards.com/install-openshift-local-to-my-windows-pc/) to your PC.

- While not mandatory, it is recommended to download **git** and **github desktop**. You can happily complete this guide by ignoring this step.

- Ensure your firewall is opened to icr.io.

- Ensure the pre-requisites as outlined in [Open Liberty InstantOn](https://openliberty.io/docs/latest/instanton.html) doc are met. This is a great read on Open Liberty InstantOn.

- While not mandatory, this post assumes you have followed the [Deploy an Open Liberty app to OpenShift Local](../_posts/2024-07-10-deploy-an-open-liberty-app-to-openshift-local.md) post. Some output and steps have been removed from this post that exist in the former post.

## Environment

This guide was completed using the following environment.


| Category | Version  | URL |
| :--- | :--- | :--- |
| OpenShift Local | **crc version**<br>CRC version: 2.33.0+c43b17<br>OpenShift version: 4.14.12<br>Podman version: 4.4.4 | [https://seanbyd.github.io/2024/03/13/installOpenShiftLocal.html](https://seanbyd.github.io/2024/03/13/installOpenShiftLocal.html) |
| Open Liberty  | 24.0.0.2<br>All GA features:<br>openliberty-24.0.0.2.zip | [https://openliberty.io/start/](https://openliberty.io/start/) |
| Java  | IBM Semeru Java 17<br>ibm-semeru-open-jdk_x64_windows_17.0.8.1_1_openj9-0.40.0.zip | [https://developer.ibm.com/languages/java/semeru-runtimes/downloads/](https://developer.ibm.com/languages/java/semeru-runtimes/downloads/) |

## Sample app

For this exercise, I wrote a [basic servlet](https://github.com/seanbyd/LibertyToOpenShiftInstantOn.git) to deploy to OpenShift. The goal being to learn about deploying apps to OpenShift, rather than developing apps for running in OpenShift.

To complete this guide you need to download this sample app.

### Download the sample project

Download the [sample project](https://github.com/seanbyd/LibertyToOpenShiftInstantOn) from github.

You can [clone the project](https://github.com/seanbyd/LibertyToOpenShiftInstantOn.git) using git or github desktop.

Alternatively you can download the [zip file](https://github.com/seanbyd/LibertyToOpenShiftInstantOn/archive/refs/heads/main.zip). Unzip to a location of your choice. For the non-developers, or those not experienced with git and github, I have followed this approach.

### Unzip the sample project

Unzip the downloaded file *LibertyToOpenShiftInstantOn-main.zip* to c:/ocp/LibertyToOpenShiftInstantOn.

Open a command window and change to the project directory.

````cmd
cd /d c:\
mkdir c:\ocp
mkdir c:\ocp\LibertyToOpenShiftInstantOn
cd /d c:\ocp\LibertyToOpenShiftInstantOn
unzip LibertyToOpenShiftInstantOn-main.zip using a tool of your choice
````

The directory should be similar to the below.

    c:\ocp\LibertyToOpenShiftInstantOn>dir

    Directory of c:\ocp\LibertyToOpenShiftInstantOn

    08/18/2024  09:52 PM    <DIR>          .
    08/18/2024  09:52 PM    <DIR>          ..
    08/18/2024  09:52 PM               896 .classpath
    08/18/2024  09:52 PM               494 .factorypath
    08/18/2024  09:52 PM               896 .project
    08/18/2024  09:52 PM    <DIR>          .settings
    08/18/2024  09:52 PM               751 Containerfile.olp.slim.java17
    08/18/2024  09:52 PM             1,275 deploy.yaml
    08/18/2024  09:52 PM             1,709 deploy_incorrectly_placed_in_spec.yaml
    08/18/2024  09:52 PM             1,530 deploy_incorrectly_placed_SA_in_spec.yaml
    08/18/2024  09:52 PM               958 scc-cap-cr-full-details-dont-use-1.yaml
    08/18/2024  09:52 PM             1,059 scc-cap-cr-full-details-dont-use-2.yaml
    08/18/2024  09:52 PM               541 scc-cap-cr-minmal.yaml
    08/18/2024  09:52 PM    <DIR>          src
    08/18/2024  09:52 PM    <DIR>          target

## Deployment

### Start OpenShift Local

````cmd
crc start
````

    C:\ocp/LibertyToOpenShift>crc start
    WARN A new version (2.38.0) has been published on https://developers.redhat.com/content-gateway/file/pub/openshift-v4/clients/crc/2.38.0/crc-windows-installer.zip
    INFO Using bundle path C:\Users\sean\.crc\cache\crc_hyperv_4.14.12_amd64.crcbundle
    INFO Checking minimum RAM requirements
    INFO Checking if running in a shell with administrator rights
    INFO Checking Windows release
    INFO Checking Windows edition
    INFO Checking if Hyper-V is installed and operational
    INFO Checking if Hyper-V service is enabled
    INFO Checking if crc-users group exists
    INFO Checking if current user is in crc-users and Hyper-V admins group
    INFO Checking if vsock is correctly configured
    INFO Checking if the win32 background launcher is installed
    INFO Checking if the daemon task is installed
    INFO Checking if the daemon task is running
    INFO Checking admin helper service is running
    INFO Checking SSH port availability
    INFO Loading bundle: crc_hyperv_4.14.12_amd64...
    INFO Starting CRC VM for openshift 4.14.12...
    INFO CRC instance is running with IP 127.0.0.1
    INFO CRC VM is running
    INFO Updating authorized keys...
    INFO Check internal and public DNS query...
    INFO Check DNS query from host...
    INFO Verifying validity of the kubelet certificates...
    INFO Starting kubelet service
    INFO Waiting for kube-apiserver availability... [takes around 2min]
    INFO Waiting until the user's pull secret is written to the instance disk...
    INFO Enabling cluster monitoring operator...
    INFO Starting openshift instance... [waiting for the cluster to stabilize]
    INFO Operator network is progressing
    INFO 4 operators are progressing: console, image-registry, ingress, network
    INFO All operators are available. Ensuring stability...
    INFO Operators are stable (2/3)...
    INFO Operators are stable (3/3)...
    INFO Adding crc-admin and crc-developer contexts to kubeconfig...
    Started the OpenShift cluster.

    The server is accessible via web console at:
    https://console-openshift-console.apps-crc.testing

    Log in as administrator:
    Username: kubeadmin
    Password: 

    Log in as user:
    Username: developer
    Password: developer

    Use the 'oc' command line interface:
    > @FOR /f "tokens=*" %i IN ('crc oc-env') DO @call %i
    > oc login -u developer https://api.crc.testing:6443

### Prepare OC CLI and podman

To use the OC CLI you'll need to set some variables. If you miss this step, you'll receive errors later on.

The "crc start" command mentions to use "crc oc-env". To avoid problems later on I use "crc podman-env".

````cmd
crc podman-env
@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
````

    c:\ocp>cd LibertyToOpenShiftInstantOn

    c:\ocp\LibertyToOpenShiftInstantOn>crc podman-env
    SET PATH=C:\Users\bdsae\.crc\bin\oc;%PATH%
    SET CONTAINER_SSHKEY=C:\Users\bdsae\.crc\machines\crc\id_ecdsa
    SET CONTAINER_HOST=ssh://core@127.0.0.1:2222/run/user/1000/podman/podman.sock
    SET DOCKER_HOST=npipe:////./pipe/crc-podman
    REM Run this command to configure your shell:
    REM @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

    c:\ocp\LibertyToOpenShiftInstantOn>@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

### (Optionally) Remove old podman images

Optionally delete any old podman images you no longer required.

| Command| Help |
| :--- | :--- |
| podman images | Display all the podman images  |
| podman rmi IMAGE_ID | Delete the podman image with image id IMAGE_ID |
| podman rmi REPOSITORY:TAG | Delete the podman image matching the combination of REPOSITORY name and TAG |
| podman rmi IMAGE_ID --force | This deletes all images including any tagged images |

````cmd
podman images
podman rmi 385d889567ef
podman rmi icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
podman rmi dea88bde62d4 --force
````

    c:\ocp\LibertyToOpenShiftInstantOn>podman images
    REPOSITORY                                                                                                             TAG                            IMAGE ID      CREATED       SIZE
    default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  5 days ago    798 MB
    localhost/liberty-to-openshift-instanton                                                                               olp-java17-1.0                 dea88bde62d4  5 days ago    798 MB
    icr.io/appcafe/open-liberty                                                                                            kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago  688 MB

    c:\ocp\LibertyToOpenShiftInstantOn>podman rmi 385d889567ef
    Untagged: icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
    Deleted: 385d889567ef799ddf8daa3e3c071462c598a96aeb5f59cf8f57d0fed9179b14

    c:\ocp\LibertyToOpenShiftInstantOn>podman images
    REPOSITORY                                                                                                             TAG             IMAGE ID      CREATED     SIZE
    default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton  olp-java17-1.0  dea88bde62d4  5 days ago  798 MB
    localhost/liberty-to-openshift-instanton                                                                               olp-java17-1.0  dea88bde62d4  5 days ago  798 MB

    c:\ocp\LibertyToOpenShiftInstantOn>podman rmi dea88bde62d4 --force
    Untagged: default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
    Untagged: localhost/liberty-to-openshift-instanton:olp-java17-1.0
    Deleted: dea88bde62d4daa1a8fdfa5a25ebf5564cc2961aac258f21e75ab974c22cc3b2

    c:\ocp\LibertyToOpenShiftInstantOn>podman images
    REPOSITORY  TAG         IMAGE ID    CREATED     SIZE

### Build the image on Linux

Use podman to build the image.

If the image *icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi* has not already been downloaded, it will be automatically downloaded for you.

Ensure you are in the correct directory containing the file *Containerfile.olp.slim.java17*.

It is not possible to build the Open Liberty InstantOn image on Windows. It needs access to the [criu](https://github.com/checkpoint-restore/criu) (Checkpoint/Restore in Userspace) feature found in Linux.

To build my test image I used a RHEL 9.2 installation running using VMWARE Player 16. These can be downloaded and used for free from the following URLs.

| Software | URL |
| :--- | :--- |
| RHEL | [RHEL 9.2](https://developers.redhat.com/products/rhel/download#publicandprivatecloudreadyrhelimages7522) |
| VMware | [VMware player 16](https://www.vmware.com/info/workstation-player/evaluation) |

Of course, if you are using Linux rather than Windows, you can proceed without needing to use VMware.

The Open Liberty InstantOn image needs to be built using the **root** account.

I tend to do as much of the work as possible using a non-root account. Unless otherwise specified, all work should be done with a non-root account.

Log into Linux with the non-root user and create the following in your home directory.

````cmd
mkdir LibertyToOpenShiftInstantOn
````

FTP LibertyToOpenShiftInstantOn-main.zip to the $HOME/LibertyToOpenShiftInstantOn directory

Unzip the file.

````cmd
cd $HOME/LibertyToOpenShiftInstantOn
unzip LibertyToOpenShiftInstantOn-main.zip using a tool of your choice
````

You should see the following structure.

    [sean@localhost ~]$ cd $HOME/LibertyToOpenShiftInstantOn
    [sean@localhost LibertyToOpenShiftInstantOn]$ ls -lR
    .:
    total 12
    -rw-r--r--. 1 sean sean  773 Aug 12 21:43 Containerfile.olp.slim.java17
    -rw-r--r--. 1 sean sean 1376 Aug 12 21:57 deploy.yaml
    -rw-r--r--. 1 sean sean  567 Aug 12 21:54 scc-cap-cr-minmal.yaml
    drwxr-xr-x. 2 sean sean   63 Aug 13 20:57 target

    ./target:
    total 8
    -rw-r--r--. 1 sean sean 3062 Aug 12 21:10 LibertyToOpenShiftInstantOn.war
    -rw-r--r--. 1 sean sean  918 Aug 12 21:11 server.xml

Now switch to the root account.

````cmd
sudo -i
````

Change to the directory you created. In my case, my HOME directory is /home/sean. Use your HOME directory.

````cmd
cd /home/sean/LibertyToOpenShiftInstantOn
````

Cleanup any old images.

````cmd
podman images
````

    [root@localhost ~]# podman images
    REPOSITORY                             TAG                            IMAGE ID      CREATED       SIZE
    localhost/libertytoopenshift           1.0-SNAPSHOT                   015675eb1c03  4 months ago  741 MB
    localhost/libertytoopenshiftinstanton  1.0-SNAPSHOT                   64879db0d28e  4 months ago  798 MB
    icr.io/appcafe/open-liberty            kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago  688 MB

Delete an old InstantOn image.

````cmd
podman rmi libertytoopenshiftinstanton:1.0-SNAPSHOT
````

    [root@localhost ~]# podman rmi libertytoopenshiftinstanton:1.0-SNAPSHOT
    Untagged: localhost/libertytoopenshiftinstanton:1.0-SNAPSHOT
    Deleted: 64879db0d28e743724f7ce5c8ccc9558502a54aa87352196f055920ab23758d1
    Deleted: 338bc1e5601e6679e06d7ff0c1ef25613739e1554560b9c42633ade185d6b79c
    Deleted: 6998a04fb8f637cd8e16a0483a7d4adb719ecbba1307299bf44bf4f73111433d
    Deleted: 46f0db66cae664004948a09d75d2d9bdaa74cc2aa02dbfbb2fd17b9f9082bb20
    Deleted: 9e91a3072d7018e79a559894d3743875bd4151f24138fceeef36fa0a7edd3c28
    Deleted: 00239816e1a71de0130fbb2a2440bc808c7c101efadb0a9e1cb383c94fc103a1
    [root@localhost ~]# podman images
    REPOSITORY                    TAG                            IMAGE ID      CREATED       SIZE
    localhost/libertytoopenshift  1.0-SNAPSHOT                   015675eb1c03  4 months ago  741 MB
    icr.io/appcafe/open-liberty   kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago  688 MB

Build the Open Liberty InstantOn image.

Take note of the parameters.

    --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined

These are required to build an Open Liberty InstantOn image. Refer to [Building an InstantOn image with Podman](https://openliberty.io/docs/latest/instanton.html#build) for more details.

````cmd
podman build -t liberty-to-openshift-instanton:olp-java17-1.0 --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
````

The command output should be similar to the following.

    [root@localhost LibertyToOpenShiftInstantOn]# podman build -t liberty-to-openshift-instanton:olp-java17-1.0 --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
    STEP 1/9: FROM icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
    STEP 2/9: ARG VERSION=1.0
    --> Using cache f4c74d7b98f42412a079fd9660c81ceadedf1cfec726ae57dd6c65cf79b2d5d5
    --> f4c74d7b98f
    STEP 3/9: ARG REVISION=SNAPSHOT
    --> Using cache cf717c4202b51903b9cbbe5c7d980b220c84cf3b328b41033fd4b14afd05ceaf
    --> cf717c4202b
    STEP 4/9: LABEL   org.opencontainers.image.authors="Sean Boyd"   org.opencontainers.image.url="local"   org.opencontainers.image.version="$VERSION"   org.opencontainers.image.revision="$REVISION"   name="LibertyToOpenShift"   version="$VERSION-$REVISION"   summary="Sample Open Liberty InstantOn app for deploying to OpenShift"   description="This servlet has been written to help test running an Open Liberty app in OpenShift"
    --> 150747e9c88
    STEP 5/9: COPY --chown=1001:0 target/server.xml /config/
    --> eeb3078ca8f
    STEP 6/9: RUN features.sh
    + SNIPPETS_SOURCE=/opt/ol/helpers/build/configuration_snippets
    + SNIPPETS_TARGET=/config/configDropins/overrides
    + SNIPPETS_TARGET_DEFAULTS=/config/configDropins/defaults
    + mkdir -p /config/configDropins/overrides
    + mkdir -p /config/configDropins/defaults
    + '[' -n '' ']'
    + '[' '' == client ']'
    + '[' '' == embedded ']'
    + [[ -n '' ]]
    + '[' '' == true ']'
    + '[' '' == true ']'
    + featureUtility installServerFeatures --acceptLicense defaultServer --noCache
    + find /opt/ol/wlp/lib /opt/ol/wlp/bin '!' -perm -g=rw -print0
    + xargs -0 -r chmod g+rw
    --> ab39c9b6e6f
    STEP 7/9: COPY --chown=1001:0 target/LibertyToOpenShiftInstantOn.war /config/apps/
    --> 66b86522b37
    STEP 8/9: RUN configure.sh
    --> 8e0a124ce02
    STEP 9/9: RUN checkpoint.sh afterAppStart
    Performing checkpoint --at=afterAppStart

    Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.668 seconds.
    [AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
    COMMIT liberty-to-openshift-instanton:olp-java17-1.0
    --> fdeb66c9055
    Successfully tagged localhost/liberty-to-openshift-instanton:olp-java17-1.0
    fdeb66c9055741fdf705ef1a89d3d04671cd694ce402fba5eaf87c45e191f286

Notice the additional step towards the bottom of the build.

    STEP 9/9: RUN checkpoint.sh afterAppStart
    Performing checkpoint --at=afterAppStart

    Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.668 seconds.
    [AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
    COMMIT liberty-to-openshift-instanton:olp-java17-1.0
    --> fdeb66c9055

For a full description of InstantOn refer to the [Faster startup for containerized applications with Open Liberty InstantOn](https://openliberty.io/docs/latest/instanton.html) doc. In particular, refer to the beforeAppStart and afterAppStart options.

If you receive the following error:

````
Error (criu/proc_parse.c:694): Can't open 1023's mapfile link 55a65bfe8000: Operation not permitted
Error (criu/cr-dump.c:1558): Collect mappings (pid: 1023) failed with -1
Error (criu/cr-dump.c:2093): Dumping FAILED.
CWWKE0963E: The server checkpoint request failed because netlink system calls were unsuccessful. If SELinux is enabled in enforcing mode, netlink system calls might be blocked by the SELinux "virt_sandbox_use_netlink" policy setting. Either disable SELinux or enable the netlink system calls with the "setsebool virt_sandbox_use_netlink 1" command.
Error: building at STEP "RUN checkpoint.sh afterAppStart": while running runtime: exit status 74
````

Issue the following command (under root) before building the image.

````cmd
setsebool virt_sandbox_use_netlink 1
````

If you receive the following error and you are not building the image using the root account, switch to the root account and re-try.

````
Error (criu/proc_parse.c:379): Failed to resolve mapping 558982528000 filename
Error (criu/proc_parse.c:694): Can't open 1023's mapfile link 558982528000: Operation not permitted
Error (criu/cr-dump.c:1558): Collect mappings (pid: 1023) failed with -1
Error (criu/cr-dump.c:2093): Dumping FAILED.
Error: building at STEP "RUN checkpoint.sh afterAppStart": while running runtime: exit status 74
````

Refer later in this post for 'problems encountered' for more details on these errors.

### Verify that the image has been created

Use podman to display the images.

Look for the image you just created named *localhost/liberty-to-openshift-instanton:olp-java17-1.0*.

If you didn't already download the image *icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi* you'll notice that it also exists.

````cmd
podman images
````

    [root@localhost LibertyToOpenShiftInstantOn]# podman images
    REPOSITORY                                TAG                            IMAGE ID      CREATED        SIZE
    localhost/liberty-to-openshift-instanton  olp-java17-1.0                 fdeb66c90557  9 minutes ago  800 MB
    localhost/libertytoopenshift              1.0-SNAPSHOT                   015675eb1c03  4 months ago   741 MB
    icr.io/appcafe/open-liberty               kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago   688 MB

### (Optionally) Test the image

Optionally test the image using podman before you deploy to OpenShift.

#### Start the podman container with missing parameters

````cmd
podman run -d --name A_NAME_OF_YOUR_CHOICE --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined  -p PORT_YOU_WILL_ACCESS:INTERNAL_PORT_IN_SERVER_XML REPOSITORY:TAG
````

The below command is missing mandatory parameters: --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined.

For some reason I often forget to add these parameters. Try this out yourself so you can see the errors.

````cmd
podman run -d --name liberty-to-openshift-instanton -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
podman ps
````

The command output should be similar to the following.

    [root@localhost LibertyToOpenShiftInstantOn]# podman run -d --name liberty-to-openshift-instanton -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
    0f8d146c7403082222e82589cc4e10b97aa7a0a1988f0de999091923bd594429

    [root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE                                                    COMMAND               CREATED        STATUS        PORTS                   NAMES
    0f8d146c7403  localhost/liberty-to-openshift-instanton:olp-java17-1.0  /opt/ol/wlp/bin/s...  9 seconds ago  Up 8 seconds  0.0.0.0:9080->9080/tcp  liberty-to-openshift-instanton

#### Check the logs and fix the errors

To check whether the container is running fine, check the logs.

| Command| Help |
| :--- | :--- |
| podman logs NAME | Display the container logs |

````cmd
podman logs liberty-to-openshift-instanton
````

If you check the logs you will see the checkpoint failed with 'Operation not permitted'.

    [root@localhost LibertyToOpenShiftInstantOn]# podman logs liberty-to-openshift-instanton

    /opt/ol/wlp/bin/server: line 1351: /opt/criu/criu: Operation not permitted
    CWWKE0961I: Restoring the checkpoint server process failed. Check the /logs/checkpoint/restore.log log to determine why the checkpoint process was not restored. Launching the server without using the checkpoint image.
    Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://0f8d146c7403:9080/
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.754 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 4.501 seconds.

Connect to the running pod and double check the build process, specifically the checkpoint.

````cmd
podman exec -ti liberty-to-openshift-instanton /bin/bash
cd logs/checkpoint
cat checkpoint.log
````

    bash-4.4$ cd logs
    bash-4.4$ ls -l
    total 16
    drwxrwx---. 2 default root   46 Aug 13 18:10 checkpoint
    -rw-rw----. 1 default root  757 Aug 13 18:10 checkpoint-console.log
    -rw-r-----. 1 default root    0 Aug 13 18:22 console.log
    -rw-rw----. 1 default root 3233 Aug 13 18:10 messages_24.08.13_18.22.05.0.log
    -rw-r-----. 1 default root 4637 Aug 13 18:22 messages.log
    bash-4.4$ cd checkpoint
    bash-4.4$ ls -l
    total 8
    -rw-rw----. 1 default root 1489 Aug 13 18:10 checkpoint.log
    -rw-rw----. 1 default root   53 Aug 13 18:10 stats-dump
    bash-4.4$ cat checkpoint.log
    Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
    Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
    Warn  (compel/src/lib/infect.c:133): Unable to interrupt task: 1088 (Operation not permitted)
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1044 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1054 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1057 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1058 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1062 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1063 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1064 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1065 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1067 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1069 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1070 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1071 with interrupted system call
    Warn  (compel/arch/x86/src/lib/infect.c:356): Will restore 1072 with interrupted system call

You will see the error 'Unable to interrupt task: 1088 (Operation not permitted)'. This was caused as the requirement parameters were not set when running podman.

Exit from the container.

````cmd
exit
````

Stop the running pod.

````
podman ps
podman stop liberty-to-openshift-instanton
podman ps
````

    [root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE                                                    COMMAND               CREATED         STATUS         PORTS                   NAMES
    0f8d146c7403  localhost/liberty-to-openshift-instanton:olp-java17-1.0  /opt/ol/wlp/bin/s...  32 minutes ago  Up 32 minutes  0.0.0.0:9080->9080/tcp  liberty-to-openshift-instanton

    [root@localhost LibertyToOpenShiftInstantOn]# podman stop liberty-to-openshift-instanton
    0f8d146c7403

    [root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

#### Start the podman container with the mandatory parameters

Run the command with the required parameters.

````cmd
podman run -d --name liberty-to-openshift-instanton --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined  -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
podman ps
````

If you get at error that the container is already in use, remove it. Note that you use 'podman rm' and not 'podman rmi'.

````cmd
podman ps
podman run -d --name liberty-to-openshift-instanton --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined  -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
podman ps -a
podman rm f1d25285c018
````

    root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

    [root@localhost LibertyToOpenShiftInstantOn]# podman run -d --name liberty-to-openshift-instanton --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined  -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
    Error: creating container storage: the container name "liberty-to-openshift-instanton" is already in use by f1d25285c01854d6a4374ab3077b1e035581c43a5fa5f6da9407c94669190219. You have to remove that container to be able to reuse that name: that name is already in use
    
    [root@localhost LibertyToOpenShiftInstantOn]# podman ps -a
    CONTAINER ID  IMAGE                                                    COMMAND               CREATED             STATUS                           PORTS                   NAMES
    f1d25285c018  localhost/liberty-to-openshift-instanton:olp-java17-1.0  /opt/ol/wlp/bin/s...  About a minute ago  Exited (130) About a minute ago  0.0.0.0:9080->9080/tcp  liberty-to-openshift-instanton
    [root@localhost LibertyToOpenShiftInstantOn]# podman rm f1d25285c018
    f1d25285c018
    [root@localhost LibertyToOpenShiftInstantOn]# podman ps -a
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

Try again.

````cmd
podman run -d --name liberty-to-openshift-instanton --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined  -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
podman ps
````

    [root@localhost LibertyToOpenShiftInstantOn]# podman run -d --name liberty-to-openshift-instanton --cap-add=CHECKPOINT_RESTORE --cap-add=SETPCAP --security-opt seccomp=unconfined  -p 9080:9080 liberty-to-openshift-instanton:olp-java17-1.0
    85a664df916ba7fd4711cd43b6e2476f146db5fec1a21a5ebead9956a230868c
    [root@localhost LibertyToOpenShiftInstantOn]#
    [root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE                                                    COMMAND               CREATED        STATUS        PORTS                   NAMES
    85a664df916b  localhost/liberty-to-openshift-instanton:olp-java17-1.0  /opt/ol/wlp/bin/s...  2 minutes ago  Up 2 minutes  0.0.0.0:9080->9080/tcp  liberty-to-openshift-instanton

#### Check the logs

````cmd
podman logs liberty-to-openshift-instanton
````

If all goes well, you'll see a succesful start using a checkpoint restore. Look for message 'resumed operation from a checkpoint'.

    [root@localhost LibertyToOpenShiftInstantOn]# podman logs liberty-to-openshift-instanton

    [AUDIT   ] Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://85a664df916b:9080/
    [AUDIT   ] CWWKC0452I: The Liberty server process resumed operation from a checkpoint in 0.436 seconds.
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.440 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.654 seconds.

You can optionally attach to the running container and check the logs.

````cmd
podman exec -ti liberty-to-openshift-instanton /bin/bash
````

#### Use curl to access the app

Test the application using curl.

````cmd
curl -vk http://localhost:9080
````

    [root@localhost LibertyToOpenShiftInstantOn]# curl -vk http://localhost:9080
    *   Trying ::1:9080...
    * connect to ::1 port 9080 failed: Connection refused
    *   Trying 127.0.0.1:9080...
    * Connected to localhost (127.0.0.1) port 9080 (#0)
    > GET / HTTP/1.1
    > Host: localhost:9080
    > User-Agent: curl/7.76.1
    > Accept: */*
    >
    * Mark bundle as not supporting multiuse
    < HTTP/1.1 200 OK
    < X-Powered-By: Servlet/4.0
    < Content-Type: text/html;charset=UTF-8
    < Content-Language: en-US
    < Transfer-Encoding: chunked
    < Date: Tue, 13 Aug 2024 19:23:29 GMT
    <
    <html>
    <head><title>Servlet Settings</title></head>
    <body>
    <h1>LibertyToOpenShift Testing - InstantOn</h1>
    <table border='1'>
    <tr><th>Name</th><th>Value</th></tr>
    <tr>
    <td>Request URL</td>
    <td>http://localhost:9080/</td>
    </tr>
    <tr>
    <td>Local Name/td>
    <td>ed8ec972d72a</td>
    </tr>
    <tr>
    <td>Remote Host/td>
    <td>host.containers.internal</td>
    </tr>
    <tr>
    <td>Server Name/td>
    <td>localhost</td>
    </tr>
    <tr>
    <td>Local addr/td>
    <td>10.88.0.24</td>
    </tr>
    <tr>
    <td>Remote addr/td>
    <td>10.88.0.1</td>
    </tr>
    <tr>
    <td>Local port/td>
    <td>9080</td>
    </tr>
    <tr>
    <td>Remote port/td>
    <td>36778</td>
    </tr>
    <tr>
    <td>Server port/td>
    <td>9080</td>
    </tr>
    </table>
    </body>
    </html>
    * Connection #0 to host localhost left intact
    [root@localhost LibertyToOpenShiftInstantOn]#

#### Browse the app using a browser

Test the application using a browser.

Find the IP of your Linux server. Let's say the IP is 10.50.12.35, issue the following in your browser.

You can try:

````cmd
ifconfig -a
ip addr
````

[http://10.50.12.35:9080](http://10.50.12.35:9080)

### Stop the running container

| Command| Help |
| :--- | :--- |
| podman ps | Display the running containers |
| podman stop CONTAINER_ID |  Stop the container CONTAINER_ID |
| podman ps -a | Display the running containers, including the stopped containers |
| podman rm CONTAINER_ID | Delete a container |

````cmd
podman ps
podman stop liberty-to-openshift-instanton
podman ps
podman ps -a
podman rm liberty-to-openshift-instanton
````

    [root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE                                                    COMMAND               CREATED        STATUS        PORTS                   NAMES
    ed8ec972d72a  localhost/liberty-to-openshift-instanton:olp-java17-1.0  /opt/ol/wlp/bin/s...  4 minutes ago  Up 4 minutes  0.0.0.0:9080->9080/tcp  liberty-to-openshift-instanton

    [root@localhost LibertyToOpenShiftInstantOn]# podman stop liberty-to-openshift-instanton
    liberty-to-openshift-instanton
    
    [root@localhost LibertyToOpenShiftInstantOn]# podman ps
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
    
    [root@localhost LibertyToOpenShiftInstantOn]# podman ps -a
    CONTAINER ID  IMAGE                                                    COMMAND               CREATED        STATUS                    PORTS                   NAMES
    ed8ec972d72a  localhost/liberty-to-openshift-instanton:olp-java17-1.0  /opt/ol/wlp/bin/s...  4 minutes ago  Exited (0) 9 seconds ago  0.0.0.0:9080->9080/tcp  liberty-to-openshift-instanton
    
    [root@localhost LibertyToOpenShiftInstantOn]# podman rm liberty-to-openshift-instanton
    liberty-to-openshift-instanton
    
    [root@localhost LibertyToOpenShiftInstantOn]# podman ps -a
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

### Push the image to OpenShift Local

Push the image to the OpenShift internal registry.

#### When using VMware on Windows

In my test lab I am running OpenShift Local on my Windows PC while building the Open Liberty InstantOn image on RHEL 9 running under VMware.

To push to OpenShift Local I need to save the podman image, FTP from the VM to my PC, and load the image using podman on Windows.

If you are using Linux and OpenShift, you can skip this step.

Display the images.

````cmd
podman images
````

    [root@localhost LibertyToOpenShiftInstantOn]# podman images
    REPOSITORY                                TAG                            IMAGE ID      CREATED         SIZE
    localhost/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  23 minutes ago  798 MB
    localhost/libertytoopenshift              1.0-SNAPSHOT                   015675eb1c03  4 months ago    741 MB
    icr.io/appcafe/open-liberty               kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago    688 MB

Save the image.

````cmd
cd /home/sean
mkdir images
cd /home/sean/images
podman save --output liberty-to-openshift-instanton_olp-java17-1.0.tar dea88bde62d4
ls -l
````

    [root@localhost LibertyToOpenShiftInstantOn]# podman images
    REPOSITORY                                TAG                            IMAGE ID      CREATED         SIZE
    localhost/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  23 minutes ago  798 MB
    localhost/libertytoopenshift              1.0-SNAPSHOT                   015675eb1c03  4 months ago    741 MB
    icr.io/appcafe/open-liberty               kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago    688 MB
    [root@localhost LibertyToOpenShiftInstantOn]# pwd
    /home/sean/LibertyToOpenShiftInstantOn
    [root@localhost LibertyToOpenShiftInstantOn]#
    [root@localhost LibertyToOpenShiftInstantOn]#
    [root@localhost LibertyToOpenShiftInstantOn]#
    [root@localhost LibertyToOpenShiftInstantOn]# cd /home/sean
    [root@localhost sean]# mkdir images
    [root@localhost sean]# cd images/
    [root@localhost images]# podman save --output liberty-to-openshift-instanton_olp-java17-1.0.tar dea88bde62d4
    Copying blob ecf6a89969f5 done
    Copying blob 38fbe8c222cd done
    Copying blob 28514f1c36df done
    Copying blob a0395d30313e done
    Copying blob 5072c9ed8ec7 done
    Copying blob 3fe103da66fa done
    Copying blob b81c520caa50 done
    Copying blob 9d2028e22db0 done
    Copying blob 74a1e6f67ecb done
    Copying blob 78d820be1a24 done
    Copying blob a037803dd8e9 done
    Copying blob 89d6feb548fc done
    Copying blob 715790d78398 done
    Copying blob 2c39fedc4e1e done
    Copying blob b90f8f8be268 done
    Copying blob c7430cd06acc done
    Copying blob 663d6c6c1533 done
    Copying blob c65df4ffdbfd done
    Copying blob 2db68336d1b7 done
    Copying blob 1d4516f97732 done
    Copying blob a5a133c1a5fc done
    Copying blob 56e3ad214bcd done
    Copying config dea88bde62 done
    Writing manifest to image destination
    Storing signatures
    [root@localhost images]# ls -l
    total 779708
    -rw-r--r--. 1 root root 798417408 Aug 13 22:35 liberty-to-openshift-instanton_olp-java17-1.0.tar

FTP the image to your PC in the directory of your choice.

Open a Windows command window

````
cd /d c:/ocp/
mkdir images
cd /d c:\ocp\images
````

    c:\ocp\LibertyToOpenShiftInstantOn>cd /d c:/ocp/

    c:\ocp>mkdir images

    c:\ocp>cd /d c:\ocp\images

    c:\ocp\images>dir
    Volume in drive C is Windows

    Directory of c:\ocp\images

    08/13/2024  10:38 PM    <DIR>          .
    08/13/2024  10:37 PM    <DIR>          ..
    08/13/2024  10:35 PM       798,417,408 liberty-to-openshift-instanton_olp-java17-1.0.tar

Check the images

````cmd
podman images
````

    c:\ocp\images>podman images
    REPOSITORY                   TAG                            IMAGE ID      CREATED       SIZE

Load the image from your Windows command window.

````cmd
podman load --input liberty-to-openshift-instanton_olp-java17-1.0.tar
podman images
````

    c:\ocp\images>podman load --input liberty-to-openshift-instanton_olp-java17-1.0.tar
    Loaded image: sha256:dea88bde62d4daa1a8fdfa5a25ebf5564cc2961aac258f21e75ab974c22cc3b2

    c:\ocp\images>podman images
    REPOSITORY                   TAG                            IMAGE ID      CREATED         SIZE
    <none>                       <none>                         dea88bde62d4  38 minutes ago  798 MB

Tag the image.

````cmd
podman tag dea88bde62d4 liberty-to-openshift-instanton:olp-java17-1.0
podman images
````

    c:\ocp\images>podman tag dea88bde62d4 liberty-to-openshift-instanton:olp-java17-1.0

    c:\ocp\images>podman images
    REPOSITORY                                TAG                            IMAGE ID      CREATED         SIZE
    localhost/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  42 minutes ago  798 MB

Now you are ready to push the image to OpenShift Local.

#### Set OC CLI variables

Before proceeding, ensure you have set the OC CLI variables. You don't know how many times I missed this only to encounter frustrating problems.

````cmd
crc podman-env
@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
````

    c:\ocp\images>crc podman-env
    SET PATH=C:\Users\bdsae\.crc\bin\oc;%PATH%
    SET CONTAINER_SSHKEY=C:\Users\bdsae\.crc\machines\crc\id_ecdsa
    SET CONTAINER_HOST=ssh://core@127.0.0.1:2222/run/user/1000/podman/podman.sock
    SET DOCKER_HOST=npipe:////./pipe/crc-podman
    REM Run this command to configure your shell:
    REM @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

    c:\ocp\images>@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

#### Login to OpenShift

Change back to the InstantOn directory.

````cmd
cd /d C:\ocp\LibertyToOpenShiftInstantOn
````

Log into OpenShift using the developer user. The password is developer.

````cmd
oc login -u developer https://api.crc.testing:6443
````

    c:\ocp\images>oc login -u developer https://api.crc.testing:6443
    Logged into "https://api.crc.testing:6443" as "developer" using existing credentials.

    You have access to the following projects and can switch between them with 'oc project <projectname>':

        guide
    * liberty-to-openshift

    Using project "liberty-to-openshift".

#### Check the project you are connected to

After logging in, you may have a project already selected.

| Command| Help |
| :--- | :--- |
| oc projects | List all projects you have access to, including the project you are connected to |
| oc project | Display the project you are connected to |
| oc project PROJECT_NAME | Select the project named PROJECT_NAME |

````cmd
oc projects
oc project
oc project liberty-to-openshift-instanton
````
    c:\ocp\LibertyToOpenShiftInstantOn>oc projects
    You have access to the following projects and can switch between them with ' project <projectname>':

        guide
    * liberty-to-openshift

    Using project "liberty-to-openshift" on server "https://api.crc.testing:6443".

#### Create the project

If the project does not exist, create it as follows.

| Command| Help |
| :--- | :--- |
| oc new-project NEW_PROJECT_NAME | Create a new project named NEW_PROJECT_NAME |
| oc project | Display the project you are connected to |
| oc project PROJECT_NAME | Switch to the project named PROJECT_NAME |
| oc projects | Display all projects you have access to |

````cmd
oc new-project liberty-to-openshift-instanton
````

    c:\ocp\images>oc new-project liberty-to-openshift-instanton
    Now using project "liberty-to-openshift-instanton" on server "https://api.crc.testing:6443".

    You can add applications to this project with the 'new-app' command. For example, try:

        oc new-app rails-postgresql-example

    to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

        kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

Ensure you are on the selected project. if not, enter the command to switch to the correct project.

````cmd
oc project liberty-to-openshift-instanton
oc projects
````

    c:\ocp\images>oc project liberty-to-openshift-instanton
    Already on project "liberty-to-openshift-instanton" on server "https://api.crc.testing:6443".

    c:\ocp\images>oc projects
    You have access to the following projects and can switch between them with ' project <projectname>':

        guide
        liberty-to-openshift
    * liberty-to-openshift-instanton

    Using project "liberty-to-openshift-instanton" on server "https://api.crc.testing:6443".

#### (Optional read) Problems if you don't set the crc podman-env now

*Optional read*

You can optionally read or skip this section. Your choice. This section covers a problem I've faced on numerous occasions while I was learning - this led to much frustration.

It is important that you set the "crc podman-env" properly before continuing. Not doing so may lead to problems when pushing the image to the OpenShift Local internal registry.

This below shows what happens before and after the environment is set. You don't need to issue any of these commands for this guide, but feel free to try it out for yourself if you like.

List the current images.

````cmd
podman images
````

    c:\ocp\LibertyToOpenShiftInstantOn>podman images
    REPOSITORY                                                                                                             TAG                            IMAGE ID      CREATED        SIZE
    localhost/liberty-to-openshift-instanton                                                                               olp-java17-1.0                 dea88bde62d4  2 days ago     798 MB
    default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  2 days ago     798 MB
    <none>                                                                                                                 <none>                         3bb93545eaac  3 days ago     741 MB
    localhost/liberty-to-openshift                                                                                         olp-java17-1.0                 e03048676c90  5 weeks ago    740 MB
    icr.io/appcafe/open-liberty                                                                                            kernel-slim-java17-openj9-ubi  b5cc5094dd1f  7 weeks ago    690 MB
    icr.io/ibm-messaging/mq                                                                                                latest                         9e0011332935  11 months ago  1.62 GB

As can be seen, I have a number of images from previous testing, including an image tagged to the OpenShift Local internal registry.

Ensure you are logged into the registry.

````cmd
oc registry login --insecure=true
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc registry login --insecure=true
    info: Using registry public hostname default-route-openshift-image-registry.apps-crc.testing
    Saved credentials for default-route-openshift-image-registry.apps-crc.testing into C:\Users\sean\.config\containers\auth.json

Now try to push the image.

````cmd
podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
````

    c:\ocp\LibertyToOpenShiftInstantOn>podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
    Getting image source signatures
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:28514f1c36dfaab17b64bafe50b81674eb26d1bd4d3a77b3e41fa845aaa6de47
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:b81c520caa506161480e54309c78d231316ef7d8d9f678f9156dffb6a29dc6f2
    Copying blob sha256:9d2028e22db09545171d5e599b1783a20bffb0f111951ad7c3b28fd406a99c93
    Copying blob sha256:74a1e6f67ecb6df52be3a1bb0171c02a3ae877ece96fc8fa29569bcfafd2a7eb
    Copying blob sha256:78d820be1a244529c6168f3aceaf86bfff3b7b7b70298cfc38e1035cb3ccfcb6
    Copying blob sha256:a037803dd8e9658a1dde19a4912df955f04c5cbff5edfc6a00a72897c54a9f32
    Copying blob sha256:89d6feb548fc2d8a0944d2520aa8b4dde3133e600f2b83791972a29f898727c8
    Copying blob sha256:715790d78398c7eeddcc5ba1a63a81a2afbcbf56e1f56ebeef159a9614a17fe3
    Copying blob sha256:2c39fedc4e1e5004c6f279d7a08ac1d9df8d1e3a873af9fd89f0ddd70d15ad9c
    Copying blob sha256:b90f8f8be268b24e0d9551aacf17025a9e5efa2283c23f5e741edbf1cf06fd37
    Copying blob sha256:c7430cd06acc6fd0bb2e2e07079ad589b82f8c6d4921d1a05d4c7a97723b211e
    Copying blob sha256:663d6c6c1533ec21d318030d64f347237b5f4c94b8fdc09d8e87fa1ce77961fb
    Copying blob sha256:c65df4ffdbfd663bef1dee404651465c96a78e04516a06320d207577a7e4ac56
    Copying blob sha256:2db68336d1b7d07f8e00774fe81a89d6c91327c43c141c4af5e119869c41d41f
    Copying blob sha256:1d4516f977329d6cd15482f25c26c4079d35404b6a20d34f532b579e0ff2642b
    Copying blob sha256:a5a133c1a5fc6e3f4c73c2857ddcb7b52cb3857134667b6d839046aed76c8f51
    Copying blob sha256:56e3ad214bcdca7ebc7d8c62d8d2926b45546ba3a85b74ae47d4682f3baeb1f7
    Getting image source signatures
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:28514f1c36dfaab17b64bafe50b81674eb26d1bd4d3a77b3e41fa845aaa6de47
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:b81c520caa506161480e54309c78d231316ef7d8d9f678f9156dffb6a29dc6f2
    Copying blob sha256:9d2028e22db09545171d5e599b1783a20bffb0f111951ad7c3b28fd406a99c93
    Copying blob sha256:89d6feb548fc2d8a0944d2520aa8b4dde3133e600f2b83791972a29f898727c8
    Copying blob sha256:a037803dd8e9658a1dde19a4912df955f04c5cbff5edfc6a00a72897c54a9f32
    Copying blob sha256:715790d78398c7eeddcc5ba1a63a81a2afbcbf56e1f56ebeef159a9614a17fe3
    Copying blob sha256:78d820be1a244529c6168f3aceaf86bfff3b7b7b70298cfc38e1035cb3ccfcb6
    Copying blob sha256:74a1e6f67ecb6df52be3a1bb0171c02a3ae877ece96fc8fa29569bcfafd2a7eb
    Copying blob sha256:2c39fedc4e1e5004c6f279d7a08ac1d9df8d1e3a873af9fd89f0ddd70d15ad9c
    Copying blob sha256:b90f8f8be268b24e0d9551aacf17025a9e5efa2283c23f5e741edbf1cf06fd37
    Copying blob sha256:c7430cd06acc6fd0bb2e2e07079ad589b82f8c6d4921d1a05d4c7a97723b211e
    Copying blob sha256:663d6c6c1533ec21d318030d64f347237b5f4c94b8fdc09d8e87fa1ce77961fb
    Copying blob sha256:2db68336d1b7d07f8e00774fe81a89d6c91327c43c141c4af5e119869c41d41f
    Copying blob sha256:c65df4ffdbfd663bef1dee404651465c96a78e04516a06320d207577a7e4ac56
    Copying blob sha256:1d4516f977329d6cd15482f25c26c4079d35404b6a20d34f532b579e0ff2642b
    Copying blob sha256:a5a133c1a5fc6e3f4c73c2857ddcb7b52cb3857134667b6d839046aed76c8f51
    Copying blob sha256:56e3ad214bcdca7ebc7d8c62d8d2926b45546ba3a85b74ae47d4682f3baeb1f7
    Getting image source signatures
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:28514f1c36dfaab17b64bafe50b81674eb26d1bd4d3a77b3e41fa845aaa6de47
    Copying blob sha256:b81c520caa506161480e54309c78d231316ef7d8d9f678f9156dffb6a29dc6f2
    Copying blob sha256:9d2028e22db09545171d5e599b1783a20bffb0f111951ad7c3b28fd406a99c93
    Copying blob sha256:78d820be1a244529c6168f3aceaf86bfff3b7b7b70298cfc38e1035cb3ccfcb6
    Copying blob sha256:74a1e6f67ecb6df52be3a1bb0171c02a3ae877ece96fc8fa29569bcfafd2a7eb
    Copying blob sha256:715790d78398c7eeddcc5ba1a63a81a2afbcbf56e1f56ebeef159a9614a17fe3
    Copying blob sha256:2c39fedc4e1e5004c6f279d7a08ac1d9df8d1e3a873af9fd89f0ddd70d15ad9c
    Copying blob sha256:a037803dd8e9658a1dde19a4912df955f04c5cbff5edfc6a00a72897c54a9f32
    Copying blob sha256:89d6feb548fc2d8a0944d2520aa8b4dde3133e600f2b83791972a29f898727c8
    Copying blob sha256:c65df4ffdbfd663bef1dee404651465c96a78e04516a06320d207577a7e4ac56
    Copying blob sha256:663d6c6c1533ec21d318030d64f347237b5f4c94b8fdc09d8e87fa1ce77961fb
    Copying blob sha256:2db68336d1b7d07f8e00774fe81a89d6c91327c43c141c4af5e119869c41d41f
    Copying blob sha256:b90f8f8be268b24e0d9551aacf17025a9e5efa2283c23f5e741edbf1cf06fd37
    Copying blob sha256:1d4516f977329d6cd15482f25c26c4079d35404b6a20d34f532b579e0ff2642b
    Copying blob sha256:c7430cd06acc6fd0bb2e2e07079ad589b82f8c6d4921d1a05d4c7a97723b211e
    Copying blob sha256:a5a133c1a5fc6e3f4c73c2857ddcb7b52cb3857134667b6d839046aed76c8f51
    Copying blob sha256:56e3ad214bcdca7ebc7d8c62d8d2926b45546ba3a85b74ae47d4682f3baeb1f7
    Getting image source signatures
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:28514f1c36dfaab17b64bafe50b81674eb26d1bd4d3a77b3e41fa845aaa6de47
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:b81c520caa506161480e54309c78d231316ef7d8d9f678f9156dffb6a29dc6f2
    Copying blob sha256:9d2028e22db09545171d5e599b1783a20bffb0f111951ad7c3b28fd406a99c93
    Copying blob sha256:a037803dd8e9658a1dde19a4912df955f04c5cbff5edfc6a00a72897c54a9f32
    Copying blob sha256:89d6feb548fc2d8a0944d2520aa8b4dde3133e600f2b83791972a29f898727c8
    Copying blob sha256:74a1e6f67ecb6df52be3a1bb0171c02a3ae877ece96fc8fa29569bcfafd2a7eb
    Copying blob sha256:78d820be1a244529c6168f3aceaf86bfff3b7b7b70298cfc38e1035cb3ccfcb6
    Copying blob sha256:715790d78398c7eeddcc5ba1a63a81a2afbcbf56e1f56ebeef159a9614a17fe3
    Copying blob sha256:2c39fedc4e1e5004c6f279d7a08ac1d9df8d1e3a873af9fd89f0ddd70d15ad9c
    Copying blob sha256:b90f8f8be268b24e0d9551aacf17025a9e5efa2283c23f5e741edbf1cf06fd37
    Copying blob sha256:c7430cd06acc6fd0bb2e2e07079ad589b82f8c6d4921d1a05d4c7a97723b211e
    Copying blob sha256:663d6c6c1533ec21d318030d64f347237b5f4c94b8fdc09d8e87fa1ce77961fb
    Copying blob sha256:c65df4ffdbfd663bef1dee404651465c96a78e04516a06320d207577a7e4ac56
    Copying blob sha256:2db68336d1b7d07f8e00774fe81a89d6c91327c43c141c4af5e119869c41d41f
    Copying blob sha256:1d4516f977329d6cd15482f25c26c4079d35404b6a20d34f532b579e0ff2642b
    Copying blob sha256:a5a133c1a5fc6e3f4c73c2857ddcb7b52cb3857134667b6d839046aed76c8f51
    Copying blob sha256:56e3ad214bcdca7ebc7d8c62d8d2926b45546ba3a85b74ae47d4682f3baeb1f7
    Error: trying to reuse blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91 at destination: pinging container registry default-route-openshift-image-registry.apps-crc.testing: Get "https://default-route-openshift-image-registry.apps-crc.testing/v2/": dial tcp 127.0.0.1:443: connect: connection refused

You will receive the error "dial tcp 127.0.0.1:443: connect: connection refused".

If you don't receive the error continue with the following section as you are all setup. If you want to simulate this issue, open a fresh command window and try this process.

Set the crc podman-env settings.

````cmd
crc podman-env
@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
````

    c:\ocp\LibertyToOpenShiftInstantOn>crc podman-env
    SET PATH=C:\Users\sean\.crc\bin\oc;%PATH%
    SET CONTAINER_SSHKEY=C:\Users\sean\.crc\machines\crc\id_ecdsa
    SET CONTAINER_HOST=ssh://core@127.0.0.1:2222/run/user/1000/podman/podman.sock
    SET DOCKER_HOST=npipe:////./pipe/crc-podman
    REM Run this command to configure your shell:
    REM @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

    c:\ocp\LibertyToOpenShiftInstantOn>@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

Check the images. You will notice the list is now different.

I've had many frustrating hours due to this situation.

````cmd
podman images
````

    c:\ocp\LibertyToOpenShiftInstantOn>podman images
    REPOSITORY                                TAG                            IMAGE ID      CREATED       SIZE
    localhost/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  2 days ago    798 MB
    icr.io/appcafe/open-liberty               kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago  688 MB

Tag the image to the OpenShift Local internal registry, and check the image list.

You should see a new tagged image pointing to OpenShift Local internal registry.

````cmd
oc registry info
podman tag liberty-to-openshift-instanton:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
podman images
````

    c:\ocp\images>oc registry info
    default-route-openshift-image-registry.apps-crc.testing

    c:\ocp\LibertyToOpenShiftInstantOn>podman tag liberty-to-openshift-instanton:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0

    c:\ocp\LibertyToOpenShiftInstantOn>podman images
    REPOSITORY                                                                                                             TAG                            IMAGE ID      CREATED       SIZE
    localhost/liberty-to-openshift-instanton                                                                               olp-java17-1.0                 dea88bde62d4  2 days ago    798 MB
    default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  2 days ago    798 MB
    icr.io/appcafe/open-liberty                                                                                            kernel-slim-java17-openj9-ubi  385d889567ef  5 months ago  688 MB

Push the image. If all goes well, you should have successfully pushed the image.

````cmd
podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
oc get imagestream
````

    c:\ocp\LibertyToOpenShiftInstantOn>podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
    Getting image source signatures
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:28514f1c36dfaab17b64bafe50b81674eb26d1bd4d3a77b3e41fa845aaa6de47
    Copying blob sha256:b81c520caa506161480e54309c78d231316ef7d8d9f678f9156dffb6a29dc6f2
    Copying blob sha256:9d2028e22db09545171d5e599b1783a20bffb0f111951ad7c3b28fd406a99c93
    Copying blob sha256:74a1e6f67ecb6df52be3a1bb0171c02a3ae877ece96fc8fa29569bcfafd2a7eb
    Copying blob sha256:78d820be1a244529c6168f3aceaf86bfff3b7b7b70298cfc38e1035cb3ccfcb6
    Copying blob sha256:a037803dd8e9658a1dde19a4912df955f04c5cbff5edfc6a00a72897c54a9f32
    Copying blob sha256:89d6feb548fc2d8a0944d2520aa8b4dde3133e600f2b83791972a29f898727c8
    Copying blob sha256:715790d78398c7eeddcc5ba1a63a81a2afbcbf56e1f56ebeef159a9614a17fe3
    Copying blob sha256:2c39fedc4e1e5004c6f279d7a08ac1d9df8d1e3a873af9fd89f0ddd70d15ad9c
    Copying blob sha256:b90f8f8be268b24e0d9551aacf17025a9e5efa2283c23f5e741edbf1cf06fd37
    Copying blob sha256:c7430cd06acc6fd0bb2e2e07079ad589b82f8c6d4921d1a05d4c7a97723b211e
    Copying blob sha256:663d6c6c1533ec21d318030d64f347237b5f4c94b8fdc09d8e87fa1ce77961fb
    Copying blob sha256:c65df4ffdbfd663bef1dee404651465c96a78e04516a06320d207577a7e4ac56
    Copying blob sha256:2db68336d1b7d07f8e00774fe81a89d6c91327c43c141c4af5e119869c41d41f
    Copying blob sha256:1d4516f977329d6cd15482f25c26c4079d35404b6a20d34f532b579e0ff2642b
    Copying blob sha256:a5a133c1a5fc6e3f4c73c2857ddcb7b52cb3857134667b6d839046aed76c8f51
    Copying blob sha256:56e3ad214bcdca7ebc7d8c62d8d2926b45546ba3a85b74ae47d4682f3baeb1f7
    Copying config sha256:dea88bde62d4daa1a8fdfa5a25ebf5564cc2961aac258f21e75ab974c22cc3b2
    Writing manifest to image destination
    Storing signatures

    c:\ocp\LibertyToOpenShiftInstantOn>oc get imagestream
    NAME                             IMAGE REPOSITORY                                                                                                        TAGS             UPDATED
    liberty-to-openshift-instanton   default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton   olp-java17-1.0   9 seconds ago

If you don't like too much typing, you can use the shortened version to get the OpenShift Local ImageStream.

````cmd
oc get is
````

#### Log into the OpenShift registry

Ensure you are logged into the registry.

````cmd
oc registry login --insecure=true
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc registry login --insecure=true
    info: Using registry public hostname default-route-openshift-image-registry.apps-crc.testing
    Saved credentials for default-route-openshift-image-registry.apps-crc.testing into C:\Users\sean\.config\containers\auth.json

#### Set the crc podman-env

Now continue with your process.

If not already done, set the crc podman-env settings.

````cmd
crc podman-env
@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
````

    c:\ocp\LibertyToOpenShiftInstantOn>crc podman-env
    SET PATH=C:\Users\sean\.crc\bin\oc;%PATH%
    SET CONTAINER_SSHKEY=C:\Users\sean\.crc\machines\crc\id_ecdsa
    SET CONTAINER_HOST=ssh://core@127.0.0.1:2222/run/user/1000/podman/podman.sock
    SET DOCKER_HOST=npipe:////./pipe/crc-podman
    REM Run this command to configure your shell:
    REM @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

    c:\ocp\LibertyToOpenShiftInstantOn>@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

#### Tag the image to the OpenShift Local internal registry

| Command| Help |
| :--- | :--- |
| oc registry info | Retrieve the OpenShift registry details where the image will be pushed |
| podman tag REPOSITORY:TAG OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Tag the image to the OpenShift registry |

Identify the registry where the image will be pushed.

````cmd
oc registry info
````

    c:\ocp\images>oc registry info
    default-route-openshift-image-registry.apps-crc.testing

Tag the image pointing to the registry.

````cmd
podman tag liberty-to-openshift-instanton:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
````

View the tagged image.

````cmd
podman images
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc registry info
    default-route-openshift-image-registry.apps-crc.testing

    c:\ocp\images>podman images
    REPOSITORY                                TAG                            IMAGE ID      CREATED        SIZE
    localhost/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  2 days ago     798 MB
    localhost/liberty-to-openshift            olp-java17-1.0                 e03048676c90  5 weeks ago    740 MB
    icr.io/appcafe/open-liberty               kernel-slim-java17-openj9-ubi  b5cc5094dd1f  7 weeks ago    690 MB

    c:\ocp\images>podman tag liberty-to-openshift-instanton:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0

    c:\ocp\images>podman images
    REPOSITORY                                                                                                             TAG                            IMAGE ID      CREATED        SIZE
    localhost/liberty-to-openshift-instanton                                                                               olp-java17-1.0                 dea88bde62d4  2 days ago     798 MB
    default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton  olp-java17-1.0                 dea88bde62d4  2 days ago     798 MB
    localhost/liberty-to-openshift                                                                                         olp-java17-1.0                 e03048676c90  5 weeks ago    740 MB
    icr.io/appcafe/open-liberty                                                                                            kernel-slim-java17-openj9-ubi  b5cc5094dd1f  7 weeks ago    690 MB

Note that there are two images with the same IMAGE ID: the original image build using podman; plus the newly tagged image specifying the OpenShift registry details.

#### Push the image to OpenShift

Issue the following command to push the image to the OpenShift Local internal registry.

| Command| Help |
| :--- | :--- |
| podman push OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Push the image to the OpenShift internal image stream |

````cmd
podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
````

    C:\Users\bdsae>podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
    Getting image source signatures
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:28514f1c36dfaab17b64bafe50b81674eb26d1bd4d3a77b3e41fa845aaa6de47
    Copying blob sha256:b81c520caa506161480e54309c78d231316ef7d8d9f678f9156dffb6a29dc6f2
    Copying blob sha256:9d2028e22db09545171d5e599b1783a20bffb0f111951ad7c3b28fd406a99c93
    Copying blob sha256:74a1e6f67ecb6df52be3a1bb0171c02a3ae877ece96fc8fa29569bcfafd2a7eb
    Copying blob sha256:78d820be1a244529c6168f3aceaf86bfff3b7b7b70298cfc38e1035cb3ccfcb6
    Copying blob sha256:a037803dd8e9658a1dde19a4912df955f04c5cbff5edfc6a00a72897c54a9f32
    Copying blob sha256:89d6feb548fc2d8a0944d2520aa8b4dde3133e600f2b83791972a29f898727c8
    Copying blob sha256:715790d78398c7eeddcc5ba1a63a81a2afbcbf56e1f56ebeef159a9614a17fe3
    Copying blob sha256:2c39fedc4e1e5004c6f279d7a08ac1d9df8d1e3a873af9fd89f0ddd70d15ad9c
    Copying blob sha256:b90f8f8be268b24e0d9551aacf17025a9e5efa2283c23f5e741edbf1cf06fd37
    Copying blob sha256:c7430cd06acc6fd0bb2e2e07079ad589b82f8c6d4921d1a05d4c7a97723b211e
    Copying blob sha256:663d6c6c1533ec21d318030d64f347237b5f4c94b8fdc09d8e87fa1ce77961fb
    Copying blob sha256:c65df4ffdbfd663bef1dee404651465c96a78e04516a06320d207577a7e4ac56
    Copying blob sha256:2db68336d1b7d07f8e00774fe81a89d6c91327c43c141c4af5e119869c41d41f
    Copying blob sha256:1d4516f977329d6cd15482f25c26c4079d35404b6a20d34f532b579e0ff2642b
    Copying blob sha256:a5a133c1a5fc6e3f4c73c2857ddcb7b52cb3857134667b6d839046aed76c8f51
    Copying blob sha256:56e3ad214bcdca7ebc7d8c62d8d2926b45546ba3a85b74ae47d4682f3baeb1f7
    Copying config sha256:dea88bde62d4daa1a8fdfa5a25ebf5564cc2961aac258f21e75ab974c22cc3b2
    Writing manifest to image destination
    Storing signatures

Possible problems:

- **Error: trying to reuse blob** sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91 at destination: unable to retrieve auth token: invalid username/password: authentication required

    Issue the following to resolve the problem

    ````cmd
    oc registry login --insecure=true
    ````

    ````
    c:\ocp\images>oc registry login --insecure=true
    info: Using registry public hostname default-route-openshift-image-registry.apps-crc.testing
    Saved credentials for default-route-openshift-image-registry.apps-crc.testing into C:\Users\bdsae\.config\containers\auth.json
    ````

- **Error: trying to reuse blob** sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91 at destination: pinging container registry default-route-openshift-image-registry.apps-crc.testing: Get "https://default-route-openshift-image-registry.apps-crc.testing/v2/": dial tcp 127.0.0.1:443: connect: connection refused

    Set the crc podman-env settings and re-try.

    ````cmd
    crc podman-env
    @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
    ````

        c:\ocp\LibertyToOpenShiftInstantOn>crc podman-env
        SET PATH=C:\Users\sean\.crc\bin\oc;%PATH%
        SET CONTAINER_SSHKEY=C:\Users\sean\.crc\machines\crc\id_ecdsa
        SET CONTAINER_HOST=ssh://core@127.0.0.1:2222/run/user/1000/podman/podman.sock
        SET DOCKER_HOST=npipe:////./pipe/crc-podman
        REM Run this command to configure your shell:
        REM @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

        c:\ocp\LibertyToOpenShiftInstantOn>@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

### Check the image in OpenShift

#### Find the pushed image

Either issue the following OC CLI command or log into OpenShift to verify the image has been pushed successfully.

````cmd
oc get imagestream
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get imagestream
    NAME                             IMAGE REPOSITORY                                                                                                        TAGS             UPDATED
    liberty-to-openshift-instanton   default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton   olp-java17-1.0   9 seconds ago

-- OR --

````cmd
oc get is
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get is
    NAME                             IMAGE REPOSITORY                                                                                                        TAGS             UPDATED
    liberty-to-openshift-instanton   default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton   olp-java17-1.0   9 seconds ago

-- OR --

Login into the OpenShift console using the developer user.

- Select the appropriate project
- Switch from the Developer role to the Administrator role
- Expand Builds
- Select ImageStreams - you should see your image here

#### (Optionally) Pull the image from OpenShift Local to podman

Optionally, to verify the image was successfully pushed to the Open Shift Local internal registry, pull it from OpenShift.

| Command| Help |
| :--- | :--- |
| podman pull OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Pull the image from OpenShift to podman |

````cmd
podman pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
````

    c:\ocp\LibertyToOpenShift>podman pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
    Trying to pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0...
    Getting image source signatures
    Copying blob sha256:19580bd631ab6ee8ed1f47416e094b849c1d0c1260c8fd2f1cde47c892669129
    Copying blob sha256:77590358b45b84caafe5ecd361bd9337ac3319cb0fcfaa581e1c7d3efdad75ce
    Copying blob sha256:c24702af7a76136c090fc91245eb7ce805e53cc786f5810a57e3d9b65385dc5d
    Copying blob sha256:8f935604158f23174be2299afb9463d20940039de707c2466dcc0734bc152fa1
    Copying blob sha256:d66a972c08964d487e0b43062210ac00ce3dca3fe569ac665c4324ea6522507d
    Copying blob sha256:77aa5edef061d0cc9eb8868e9c94fa1f415dc6d5f6f6852fd0b163e684f2bc83
    Copying blob sha256:94a481cb41b6c8ef49caf6ee7311a625463557f65600e92e42b578b033928830
    Copying blob sha256:092bf0984951efbeae6fe45d1e6c72c0ffd3d3ec5ccb124c0677f6d75696f826
    Copying blob sha256:412d8924673992313cf26622c40c3a21bad3ce0056b48eb555081febd9fd9d49
    Copying blob sha256:6d15ab97294d6e59539f04b37d4754e82d4768c73c445c5a55451d7cf93df782
    Copying blob sha256:92a53db242f82ce48443a4fd55c5132e2aa02338a68508048f8c9b2f5de0ae2d
    Copying blob sha256:3dd1916c1c6b125aed548cd0de43b94237349a5aa2c6113cf2abeb24d3e67b4b
    Copying blob sha256:255c33f808df935686dd1116328b89b8b15453951d3cc4d259861a2cbd7e16f3
    Copying blob sha256:8eff0be5f172d635d552914290d1234800129f9b7e976c6d07e14a36ee73bac6
    Copying blob sha256:f8ac83ba3f93f24cdbd7f5fb89c66a47b22f4896a407860077ad4427162c2094
    Copying blob sha256:15b973b7e658692708aa4705857afc18bcf7bfcc3da6abc8b4731bc0fdc12640
    Copying blob sha256:afa615b17d2f5174b31afe1510f40c07186be05fece770df91180286691ba87a
    Copying blob sha256:26f544582b0deefe9b360c5802261c87f5192528403c566f1c89d92af2b2117c
    Copying blob sha256:8e3000e019338cb0e036021761086f515f7ce8f5ef9412b060c10dddc9b184dd
    Copying blob sha256:fd1b7871dd73094afe0bd2f2bc0ef69d8fff444644d4fe8beab1879f1e4ac356
    Copying blob sha256:2bac988494112099c3836fbd246aa7707f4d1c7d428fcac4f8c35293ccd7efc9
    Copying blob sha256:db759ac130a181c129c5331f54376ae1e4a9f1f5c2f3f3e2d9c0bee6e0092c82
    Copying config sha256:dea88bde62d4daa1a8fdfa5a25ebf5564cc2961aac258f21e75ab974c22cc3b2
    Writing manifest to image destination
    Storing signatures
    dea88bde62d4daa1a8fdfa5a25ebf5564cc2961aac258f21e75ab974c22cc3b2

## Deploy an application

### Create a service account

Create a service account using the "developer" user. This service account will be mapped to the SCC in the following section.

The service account name must match the "serviceAccountName" field in the deploy.yaml file.

First, ensure you are in the correct project. If you are not on the correct project, switch to it.

````cmd
oc project
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc project
    Using project "liberty-to-openshift-instanton" on server "https://api.crc.testing:6443".

Create the service account.

````cmd
oc create serviceaccount liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc create serviceaccount liberty-to-openshift-instanton
    serviceaccount/liberty-to-openshift-instanton created

Check that it has been created.

````cmd
oc get sa liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get sa liberty-to-openshift-instanton
    NAME                             SECRETS   AGE
    liberty-to-openshift-instanton   1         15s

    c:\ocp\LibertyToOpenShiftInstantOn>oc get sa
    NAME                             SECRETS   AGE
    builder                          1         2d21h
    default                          1         2d21h
    deployer                         1         2d21h
    liberty-to-openshift-instanton   1         6s

Optionally display the service account details. There is nothing you need to check here other than to practice the command.

````cmd
oc describe sa liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc describe sa liberty-to-openshift-instanton
    Name:                liberty-to-openshift-instanton
    Namespace:           liberty-to-openshift-instanton
    Labels:              <none>
    Annotations:         <none>
    Image pull secrets:  liberty-to-openshift-instanton-dockercfg-wjdhg
    Mountable secrets:   liberty-to-openshift-instanton-dockercfg-wjdhg
    Tokens:              liberty-to-openshift-instanton-token-bljpd
    Events:              <none>

### Create the Security Context Contraints (SCC)

To run and Open liberty InstantOn app in OpenShift (Local or full) a Security Context Contraints (SCC) needs to be created. The service account will be mapped to the SCC giving the service account the necessary privileges to start an OpenLiberty InstantOn app.

This needs to be created with a privileged account such as a cluster admin account of on the case of OpenShidt Local with the "kubeadmin" account.

If you issue the command with the "developer" account you'll get the following error.

````cmd
oc apply -f scc-cap-cr-minmal.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc apply -f scc-cap-cr-minmal.yaml
    Error from server (Forbidden): error when retrieving current configuration of:
    Resource: "security.openshift.io/v1, Resource=securitycontextconstraints", GroupVersionKind: "security.openshift.io/v1, Kind=SecurityContextConstraints"
    Name: "liberty-to-openshift-instanton-scc", Namespace: ""
    from server for: "scc-cap-cr-minmal.yaml": securitycontextconstraints.security.openshift.io "liberty-to-openshift-instanton-scc" is forbidden: User "developer" cannot get resource "securitycontextconstraints" in API group "security.openshift.io" at the cluster scope

Switch to the kubeadmin user.

````cmd
oc login -u kubeadmin https://api.crc.testing:6443
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc login -u kubeadmin https://api.crc.testing:6443
    Logged into "https://api.crc.testing:6443" as "kubeadmin" using existing credentials.

    You have access to 71 projects, the list has been suppressed. You can list all projects with 'oc projects'

    Using project "liberty-to-openshift-instanton".

Now create the SCC.

````cmd
oc apply -f scc-cap-cr-minmal.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc apply -f scc-cap-cr-minmal.yaml
    securitycontextconstraints.security.openshift.io/liberty-to-openshift-instanton created

Verify the SCC was created.

````cmd
oc get scc liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get scc liberty-to-openshift-instanton
    NAME                             PRIV    CAPS         SELINUX     RUNASUSER          FSGROUP    SUPGROUP   PRIORITY     READONLYROOTFS   VOLUMES
    liberty-to-openshift-instanton   false   <no value>   MustRunAs   MustRunAsNonRoot   RunAsAny   RunAsAny   <no value>   false            ["awsElasticBlockStore","azureDisk","azureFile","cephFS","cinder","configMap","csi","downwardAPI","emptyDir","ephemeral","fc","flexVolume","flocker","gcePersistentDisk","gitRepo","glusterfs","iscsi","nfs","persistentVolumeClaim","photonPersistentDisk","portworxVolume","projected","quobyte","rbd","scaleIO","secret","storageOS","vsphere"]

Display all the SCC details. Take particular not of:

    Default Add Capabilities:                     CHECKPOINT_RESTORE,SETPCAP
    Required Drop Capabilities:                   KILL,MKNOD,SETUID,SETGID

````cmd
oc describe scc liberty-to-openshift-instanton
````
    c:\ocp\LibertyToOpenShiftInstantOn>oc describe scc liberty-to-openshift-instanton
    Name:                                           liberty-to-openshift-instanton
    Priority:                                       <none>
    Access:
    Users:                                        <none>
    Groups:                                       <none>
    Settings:
    Allow Privileged:                             false
    Allow Privilege Escalation:                   true
    Default Add Capabilities:                     CHECKPOINT_RESTORE,SETPCAP
    Required Drop Capabilities:                   KILL,MKNOD,SETUID,SETGID
    Allowed Capabilities:                         <none>
    Allowed Seccomp Profiles:                     <none>
    Allowed Volume Types:                         awsElasticBlockStore,azureDisk,azureFile,cephFS,cinder,configMap,csi,downwardAPI,emptyDir,ephemeral,fc,flexVolume,flocker,gcePersistentDisk,gitRepo,glusterfs,iscsi,nfs,persistentVolumeClaim,photonPersistentDisk,portworxVolume,projected,quobyte,rbd,scaleIO,secret,storageOS,vsphere
    Allowed Flexvolumes:                          <all>
    Allowed Unsafe Sysctls:                       <none>
    Forbidden Sysctls:                            <none>
    Allow Host Network:                           false
    Allow Host Ports:                             false
    Allow Host PID:                               false
    Allow Host IPC:                               false
    Read Only Root Filesystem:                    false
    Run As User Strategy: MustRunAsNonRoot
        UID:                                        1001
        UID Range Min:                              <none>
        UID Range Max:                              <none>
    SELinux Context Strategy: MustRunAs
        User:                                       <none>
        Role:                                       <none>
        Type:                                       <none>
        Level:                                      <none>
    FSGroup Strategy: RunAsAny
        Ranges:                                     <none>
    Supplemental Groups Strategy: RunAsAny
        Ranges:                                     <none>

Create the role based mapping between the SCC and the service account.

| Command| Help | Example |
| :--- | :--- | :--- |
| oc adm policy add-scc-to-user SCC_NAME -z SERVICE_ACCOUNT_NAME | SCC_NAME is the SCC created using the kubeadmin account<br>SERVICE_ACCOUNT_NAME is the service account name created using the develper account| oc adm policy add-scc-to-user liberty-to-openshift-instanton -z liberty-to-openshift-instanton |

````cmd
oc adm policy add-scc-to-user liberty-to-openshift-instanton -z liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc adm policy add-scc-to-user liberty-to-openshift-instanton -z liberty-to-openshift-instanton
    clusterrole.rbac.authorization.k8s.io/system:openshift:scc:liberty-to-openshift-instanton added: "liberty-to-openshift-instanton"

If you are interested in more details on this command enter.

````cmd
oc adm policy --help
oc adm policy add-scc-to-user --help
````

If you want to read more details on SCC, you can read the blod [Managing SCCs in OpenShift]([https://www.redhat.com/en/blog/managing-sccs-in-openshift).

Switch back to the developer account. All remaining steps should be created under this account.

````cmd
oc login -u developer https://api.crc.testing:6443
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc login -u developer https://api.crc.testing:6443
    Logged into "https://api.crc.testing:6443" as "developer" using existing credentials.

    You have access to the following projects and can switch between them with 'oc project <projectname>':

        guide
        liberty-to-openshift
    * liberty-to-openshift-instanton

    Using project "liberty-to-openshift-instanton".

Ensure you are still on the correct project.

### (Optional read) Problems when placing serviceAccountName or securityContext in the wrong place.

*Optional read*

You can optionally read or skip this section. Your choice. This section covers a challenging problem I faced when trying to deploy my first Open liberty InstantOn app to OpenShift Local and OpenShift on Linux-X86.

The first problem I faced was placing the following in the "spec" section of the OpenShift deploy yaml file. The correct location is in the "containers" section of the yaml file.

    spec:
      # Add InstantOn - this is in the wrong spot, it should be under container
      serviceAccountName: libertytoopenshiftinstanton-sa
      securityContext:
        runAsNonRoot: true
        privileged: false
        allowPrivilegeEscalation: true
        capabilities:
          add:
            - CHECKPOINT_RESTORE
            - SETPCAP
          drop:
            - ALL
      # Add InstantOn

You can try this out yourself.

````cmd
oc apply -f deploy_incorrectly_placed_in_spec.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc apply -f deploy_incorrectly_placed_in_spec.yaml
    deployment.apps/liberty-to-openshift-instanton created
    service/liberty-to-openshift-instanton created
    route.route.openshift.io/liberty-to-openshift-instanton created

From the output it looks like all was ok.

Check the running pod. The pod is running fine.

````cmd
oc get po
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get po
    NAME                                             READY   STATUS    RESTARTS   AGE
    liberty-to-openshift-instanton-9fc5d9dc5-d9xnp   1/1     Running   0          4s

Now check the logs.

| Command| Help |
| :--- | :--- |
| oc logs NAME | Display the logs of the running container. Get the NAME from the "oc get po" command. |

````cmd
oc logs liberty-to-openshift-instanton-9fc5d9dc5-d9xnp
````

You will see an error "line 1351: /opt/criu/criu: Operation not permitted". The checkpoint restore failed.

    c:\ocp\LibertyToOpenShiftInstantOn>oc logs liberty-to-openshift-instanton-9fc5d9dc5-d9xnp

    /opt/ol/wlp/bin/server: line 1351: /opt/criu/criu: Operation not permitted
    CWWKE0961I: Restoring the checkpoint server process failed. Check the /logs/checkpoint/restore.log log to determine why the checkpoint process was not restored. Launching the server without using the checkpoint image.
    Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://liberty-to-openshift-instanton-9fc5d9dc5-d9xnp:9080/
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.134 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.908 seconds.

You can connect to the running container and check the logs in more detail.

| Command| Help |
| :--- | :--- |
| oc rsh NAME | Connect to the running container. This is similar to the podman "podman exec -ti" command. Get the NAME from the "oc get po" command. |

````cmd
oc rsh liberty-to-openshift-instanton-9fc5d9dc5-d9xnp
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc rsh liberty-to-openshift-instanton-9fc5d9dc5-d9xnp
    sh-4.4$ pwd
    /
    sh-4.4$ cd logs
    sh-4.4$ ls -l
    total 16
    drwxrwx---. 2 default    root   46 Aug 13 19:09 checkpoint
    -rw-rw----. 1 default    root  757 Aug 13 19:09 checkpoint-console.log
    -rw-r-----. 1 1000690000 root    0 Aug 18 17:28 console.log
    -rw-rw----. 1 default    root 3233 Aug 13 19:09 messages_24.08.18_17.28.55.0.log
    -rw-r-----. 1 1000690000 root 4706 Aug 18 17:28 messages.log
    sh-4.4$ ls -l checkpoint
    total 8
    -rw-rw----. 1 default root 1489 Aug 13 19:09 checkpoint.log
    -rw-rw----. 1 default root   53 Aug 13 19:09 stats-dump
    sh-4.4$ exit
    exit

There are no additional details to help.

Check the events.

| Command| Help |
| :--- | :--- |
| oc get events | Get the OpenShift events |

````cmd
oc get events
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get events
    LAST SEEN   TYPE      REASON                   OBJECT                                                       MESSAGE
    12m         Normal    Scheduled                pod/liberty-to-openshift-instanton-85fbc954dd-zx4rb          Successfully assigned liberty-to-openshift-instanton/liberty-to-openshift-instanton-85fbc954dd-zx4rb to crc-vlf7c-master-0
    12m         Normal    AddedInterface           pod/liberty-to-openshift-instanton-85fbc954dd-zx4rb          Add eth0 [10.217.0.92/23] from openshift-sdn
    12m         Normal    Pulled                   pod/liberty-to-openshift-instanton-85fbc954dd-zx4rb          Container image "image-registry.openshift-image-registry.svc:5000/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0" already present on machine
    12m         Normal    Created                  pod/liberty-to-openshift-instanton-85fbc954dd-zx4rb          Created container liberty-to-openshift-instanton
    12m         Normal    Started                  pod/liberty-to-openshift-instanton-85fbc954dd-zx4rb          Started container liberty-to-openshift-instanton
    11m         Normal    Killing                  pod/liberty-to-openshift-instanton-85fbc954dd-zx4rb          Stopping container liberty-to-openshift-instanton
    12m         Normal    SuccessfulCreate         replicaset/liberty-to-openshift-instanton-85fbc954dd         Created pod: liberty-to-openshift-instanton-85fbc954dd-zx4rb
    8m25s       Normal    Scheduled                pod/liberty-to-openshift-instanton-9fc5d9dc5-d9xnp           Successfully assigned liberty-to-openshift-instanton/liberty-to-openshift-instanton-9fc5d9dc5-d9xnp to crc-vlf7c-master-0
    8m24s       Normal    AddedInterface           pod/liberty-to-openshift-instanton-9fc5d9dc5-d9xnp           Add eth0 [10.217.0.93/23] from openshift-sdn
    8m24s       Normal    Pulled                   pod/liberty-to-openshift-instanton-9fc5d9dc5-d9xnp           Container image "image-registry.openshift-image-registry.svc:5000/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0" already present on machine
    8m23s       Normal    Created                  pod/liberty-to-openshift-instanton-9fc5d9dc5-d9xnp           Created container libertytoopenshiftinstanton-container
    8m23s       Normal    Started                  pod/liberty-to-openshift-instanton-9fc5d9dc5-d9xnp           Started container libertytoopenshiftinstanton-container
    25m         Warning   FailedCreatePodSandBox   pod/liberty-to-openshift-instanton-9fc5d9dc5-krdkt           Failed to create pod sandbox: rpc error: code = Unknown desc = failed to create pod network sandbox k8s_liberty-to-openshift-instanton-9fc5d9dc5-krdkt_liberty-to-openshift-instanton_be2f46d0-a84a-4b4b-a88f-4f45193b8d57_0(a2e2c386c036ff3a30e4fff6c57ebf233bc1200289cc6857064e56d2f5d87611): error adding pod liberty-to-openshift-instanton_liberty-to-openshift-instanton-9fc5d9dc5-krdkt to CNI network "multus-cni-network": plugin type="multus-shim" name="multus-cni-network" failed (add): CmdAdd (shim): failed to send CNI request: Post "http://dummy/cni": dial unix /run/multus/socket/multus.sock: connect: no such file or directory
    24m         Normal    AddedInterface           pod/liberty-to-openshift-instanton-9fc5d9dc5-krdkt           Add eth0 [10.217.0.79/23] from openshift-sdn
    24m         Normal    Pulled                   pod/liberty-to-openshift-instanton-9fc5d9dc5-krdkt           Container image "image-registry.openshift-image-registry.svc:5000/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0" already present on machine
    24m         Normal    Created                  pod/liberty-to-openshift-instanton-9fc5d9dc5-krdkt           Created container libertytoopenshiftinstanton-container
    24m         Normal    Started                  pod/liberty-to-openshift-instanton-9fc5d9dc5-krdkt           Started container libertytoopenshiftinstanton-container
    12m         Normal    Killing                  pod/liberty-to-openshift-instanton-9fc5d9dc5-krdkt           Stopping container libertytoopenshiftinstanton-container
    8m26s       Normal    SuccessfulCreate         replicaset/liberty-to-openshift-instanton-9fc5d9dc5          Created pod: liberty-to-openshift-instanton-9fc5d9dc5-d9xnp
    12m         Normal    ScalingReplicaSet        deployment/liberty-to-openshift-instanton                    Scaled up replica set liberty-to-openshift-instanton-85fbc954dd to 1
    8m26s       Normal    ScalingReplicaSet        deployment/liberty-to-openshift-instanton                    Scaled up replica set liberty-to-openshift-instanton-9fc5d9dc5 to 1
    5m11s       Warning   FailedCreate             replicaset/libertytoopenshiftinstanton-deployment-84c48874   Error creating: pods "libertytoopenshiftinstanton-deployment-84c48874-" is forbidden: error looking up service account liberty-to-openshift-instanton/libertytoopenshiftinstanton-sa: serviceaccount "libertytoopenshiftinstanton-sa" not found

Towards the bottom you'll see the error:

    is forbidden: error looking up service account liberty-to-openshift-instanton/libertytoopenshiftinstanton-sa: serviceaccount "libertytoopenshiftinstanton-sa" not found

This suggests something wrong with the service account.

You can delete your deployment.

````cmd
oc delete -f deploy_incorrectly_placed_in_spec.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc delete -f deploy_incorrectly_placed_in_spec.yaml
    deployment.apps "liberty-to-openshift-instanton" deleted
    service "liberty-to-openshift-instanton" deleted
    route.route.openshift.io "liberty-to-openshift-instanton" deleted

The next mistake I made was to move the securityContext to the "containers" section but not move the serviceAccountName. Don't ask why? I have no idea!

Deploy the app again, but still with one error. I couldn't work this one out until someone pointed it out to me.

````cmd
oc apply -f deploy_incorrectly_placed_SA_in_spec.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc apply -f deploy_incorrectly_placed_SA_in_spec.yaml
    deployment.apps/liberty-to-openshift-instanton created
    service/liberty-to-openshift-instanton created
    route.route.openshift.io/liberty-to-openshift-instanton created

From the output it looks like all was ok.

Check the running pod. The pod is running fine.

````cmd
oc get po
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get po
    NAME                                             READY   STATUS    RESTARTS   AGE
    liberty-to-openshift-instanton-d4b676bbd-2mbp9   1/1     Running   0          36s

Now check the logs.

````cmd
oc logs liberty-to-openshift-instanton-d4b676bbd-2mbp9
````

This is a little unexpected. On my home PC using OpenShift Local, the checkpiont restore worked fine. But when using a full installation of OpenShift I got the following error.

Delete the deployment so you can continue cleanly in the next section.

````cmd
oc delete -f deploy_incorrectly_placed_SA_in_spec.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc delete -f deploy_incorrectly_placed_SA_in_spec.yaml
    deployment.apps "liberty-to-openshift-instanton" deleted
    service "liberty-to-openshift-instanton" deleted
    route.route.openshift.io "liberty-to-openshift-instanton" deleted

### Deploy an application to OpenShift using the pushed image

| Command| Help |
| :--- | :--- |
| oc apply -f file DEPLOY_YAML | Deploy the app using the contents of the DEPLOY_YAML file |

````cmd
cd /d c:\ocp\LibertyToOpenShiftInstantOn
oc apply -f deploy.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc apply -f deploy.yaml
    deployment.apps/liberty-to-openshift-instanton created
    service/liberty-to-openshift-instanton created
    route.route.openshift.io/liberty-to-openshift-instanton created

### Check whether the pod is running

````cmd
oc get pods
oc get po
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get po
    NAME                                              READY   STATUS    RESTARTS   AGE
    liberty-to-openshift-instanton-85fbc954dd-zmsfs   1/1     Running   0          31s

### Check the logs

| Command| Help |
| :--- | :--- |
| oc logs NAME | Display the logs for the NAME from the "oc get po" command |

````cmd
oc logs liberty-to-openshift-instanton-85fbc954dd-zmsfs
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc logs liberty-to-openshift-instanton-85fbc954dd-zmsfs

    [AUDIT   ] Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://liberty-to-openshift-instanton-85fbc954dd-zmsfs:9080/
    [AUDIT   ] CWWKC0452I: The Liberty server process resumed operation from a checkpoint in 0.053 seconds.
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.053 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.124 seconds.

You will see the message "The Liberty server process resumed operation from a checkpoint". And you should see the server start considerably faster.

You can optionally connect to the running container and check the log details.

| Command| Help |
| :--- | :--- |
| oc rsh NAME | Connect to the running container. This is similar to the podman "podman exec -ti" command. |

````cmd
oc rsh liberty-to-openshift-instanton-85fbc954dd-zmsfs
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc rsh liberty-to-openshift-instanton-85fbc954dd-zmsfs
    sh-4.4$ cd logs
    sh-4.4$ ls -l
    total 16
    drwxrwx---. 1 default root   25 Aug 18 17:58 checkpoint
    -rw-rw----. 1 default root  757 Aug 13 19:09 checkpoint-console.log
    -rw-r-----. 1 default root  710 Aug 18 17:58 console.log
    -rw-rw----. 1 default root 3233 Aug 13 19:09 messages_24.08.18_17.58.42.0.log
    -rw-r-----. 1 default root 2272 Aug 18 17:58 messages.log
    sh-4.4$ cat console.log
    [AUDIT   ] Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://liberty-to-openshift-instanton-85fbc954dd-zmsfs:9080/
    [AUDIT   ] CWWKC0452I: The Liberty server process resumed operation from a checkpoint in 0.053 seconds.
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.053 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.124 seconds.
    sh-4.4$ cd checkpoint
    sh-4.4$ ls -l
    total 12
    -rw-rw----. 1 default root 1489 Aug 13 19:09 checkpoint.log
    -rw-------. 1 default root  528 Aug 18 17:58 restore.log
    -rw-rw----. 1 default root   53 Aug 13 19:09 stats-dump
    sh-4.4$ cat restore.log
    Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
    sh-4.4$ exit
    exit

### Use curl to access the app - wrong way

If you try the same URL as used with podman, you'll see the below error.

````cmd
curl -vk http://localhost:9080
````

    c:\ocp\LibertyToOpenShiftInstantOn>curl -vk http://localhost:9080
    * Host localhost:9080 was resolved.
    * IPv6: ::1
    * IPv4: 127.0.0.1
    *   Trying [::1]:9080...
    *   Trying 127.0.0.1:9080...
    * connect to ::1 port 9080 from :: port 55624 failed: Connection refused
    * connect to 127.0.0.1 port 9080 from 0.0.0.0 port 55625 failed: Connection refused
    * Failed to connect to localhost port 9080 after 2268 ms: Couldn't connect to server
    * Closing connection
    curl: (7) Failed to connect to localhost port 9080 after 2268 ms: Couldn't connect to server

### Use curl to access the app - correct way

For OpenShift you need to access the URL using the OpenShift route. 

The command "oc apply -f deploy.yaml" creates the OpenShift route and service. These are used to access the application.

Find the route to the running pod.

````cmd
oc get routes
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get routes
    NAME                             HOST/PORT                                                                        PATH   SERVICES                         PORT    TERMINATION   WILDCARD
    liberty-to-openshift-instanton   liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing          liberty-to-openshift-instanton   <all>                 None

Now curl to the route.

````cmd
curl -vk liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing
````

    c:\ocp\LibertyToOpenShiftInstantOn>curl -vk liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing
    * Host liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing:80 was resolved.
    * IPv6: (none)
    * IPv4: 127.0.0.1
    *   Trying 127.0.0.1:80...
    * Connected to liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing (127.0.0.1) port 80
    > GET / HTTP/1.1
    > Host: liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing
    > User-Agent: curl/8.8.0
    > Accept: */*
    >
    * Request completely sent off
    < HTTP/1.1 200 OK
    < x-powered-by: Servlet/4.0
    < content-type: text/html;charset=UTF-8
    < content-language: en-US
    < transfer-encoding: chunked
    < date: Sun, 18 Aug 2024 18:05:37 GMT
    < set-cookie: 9770bdb7ac95d5c38cd6578dd5259673=cb927f2ae7b71bfc109d527360f594ca; path=/; HttpOnly
    <
    <html>
    <head><title>Servlet Settings</title></head>
    <body>
    <h1>LibertyToOpenShift Testing - InstantOn</h1>
    <table border='1'>
    <tr><th>Name</th><th>Value</th></tr>
    <tr>
    <td>Request URL</td>
    <td>http://liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing/</td>
    </tr>
    <tr>
    <td>Local Name/td>
    <td>liberty-to-openshift-instanton-85fbc954dd-zmsfs</td>
    </tr>
    <tr>
    <td>Remote Host/td>
    :
    :
    * Connection #0 to host liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing left intact

### Browse the app using a browser

Open a browser of your choice and paste the below URL.

[http://liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing/](http://liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing/)

or if you have enables HTTPS from a later step, use

[https://liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing/](https://liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing/)

### Access the app from OpenShift

A third option is to access the URL from within the OpenShift console.

Login into the OpenShift console.

- Select the appropriate project
- Switch from the Developer role to the Administrator role
- Expand Networking
- Select Routes
- Click on the Location URL to access the app

### (Optionally) Enable HTTPS

To enable HTTPS,

Login into the OpenShift console.

- Select the appropriate project
- Switch from the Developer role to the Administrator role
- Expand Networking
- Select Routes
- Click on the traffic light (three dots) on the right hand side of the route and select "Edit Route"
- Select the "Target port"
- Click on "Secure route"
- Select the "TLS termination" as "Edge"
- SAVE

Now click on the route and you'll see HTTPS enabled.

## Additional commands

Some other useful OC CLI commands.

| Command| Help |
| :--- | :--- |
| oc get routes | List the OpenShift routes |
| oc describe route ROUTE_NAME | Display the details of the route named ROUTE_NAME |
| oc get services | List the OpenShift services |
| oc describe service SERVICE_NAME | Display the details of the service named SERVICE_NAME |
| oc get po | List the running pods |
| oc describe po POD_NAME | Display the details of the running pod named POD_NAME |

````cmd
oc get routes
oc describe route liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get routes
    NAME                             HOST/PORT                                                                        PATH   SERVICES                         PORT   TERMINATION   WILDCARD
    liberty-to-openshift-instanton   liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing          liberty-to-openshift-instanton   9080   edge          None

    c:\ocp\LibertyToOpenShiftInstantOn>oc describe route liberty-to-openshift-instanton
    Name:                   liberty-to-openshift-instanton
    Namespace:              liberty-to-openshift-instanton
    Created:                10 minutes ago
    Labels:                 <none>
    Annotations:            kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"route.openshift.io/v1","kind":"Route","metadata":{"annotations":{},"name":"liberty-to-openshift-instanton","namespace":"liberty-to-openshift-instanton"},"spec":{"to":{"kind":"Service","name":"liberty-to-openshift-instanton"}}}

                            openshift.io/host.generated=true
    Requested Host:         liberty-to-openshift-instanton-liberty-to-openshift-instanton.apps-crc.testing
                            exposed on router default (host router-default.apps-crc.testing) 10 minutes ago
    Path:                   <none>
    TLS Termination:        edge
    Insecure Policy:        <none>
    Endpoint Port:          9080

    Service:        liberty-to-openshift-instanton
    Weight:         100 (100%)
    Endpoints:      10.217.0.107:9080

````cmd
oc get services
oc describe service liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get services
    NAME                                  TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    liberty-to-openshift-instanton        ClusterIP   10.217.4.60   <none>        9080/TCP   12m

    c:\ocp\LibertyToOpenShiftInstantOn>oc describe service liberty-to-openshift-instanton
    Name:              liberty-to-openshift-instanton
    Namespace:         liberty-to-openshift-instanton
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=liberty-to-openshift-instanton
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.217.4.60
    IPs:               10.217.4.60
    Port:              <unset>  9080/TCP
    TargetPort:        9080/TCP
    Endpoints:         10.217.0.107:9080
    Session Affinity:  None
    Events:            <none>

````cmd
oc get po
oc describe po liberty-to-openshift-instanton-85fbc954dd-zmsfs
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get po
    NAME                                              READY   STATUS    RESTARTS   AGE
    liberty-to-openshift-instanton-85fbc954dd-zmsfs   1/1     Running   0          14m

    c:\ocp\LibertyToOpenShiftInstantOn>oc describe po liberty-to-openshift-instanton-85fbc954dd-zmsfs
    Name:             liberty-to-openshift-instanton-85fbc954dd-zmsfs
    Namespace:        liberty-to-openshift-instanton
    Priority:         0
    Service Account:  default
    Node:             crc-vlf7c-master-0/192.168.126.11
    Start Time:       Sun, 18 Aug 2024 20:58:40 +0300
    Labels:           app=liberty-to-openshift-instanton
                    pod-template-hash=85fbc954dd
    Annotations:      k8s.v1.cni.cncf.io/network-status:
                        [{
                            "name": "openshift-sdn",
                            "interface": "eth0",
                            "ips": [
                                "10.217.0.107"
                            ],
                            "default": true,
                            "dns": {}
                        }]
                    openshift.io/scc: libertytoopenshiftinstanton-cap-cr-scc
    Status:           Running
    IP:               10.217.0.107
    IPs:
    IP:           10.217.0.107
    Controlled By:  ReplicaSet/liberty-to-openshift-instanton-85fbc954dd
    Containers:
    liberty-to-openshift-instanton:
        Container ID:   cri-o://d0932c5d74ebffc4f42e8b3c61c83f4ae9ac8fcadca90474afb91f53c2e1c279
        Image:          image-registry.openshift-image-registry.svc:5000/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0
        Image ID:       image-registry.openshift-image-registry.svc:5000/liberty-to-openshift-instanton/liberty-to-openshift-instanton@sha256:dd641bc9b3c4393f5a7d652e93a6d07f373fe9f15154ce8f6c580e54aa3e3d93
        Port:           9080/TCP
        Host Port:      0/TCP
        State:          Running
        Started:      Sun, 18 Aug 2024 20:58:42 +0300
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xq6hb (ro)
    Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
    Volumes:
    kube-api-access-xq6hb:
        Type:                    Projected (a volume that contains injected data from multiple sources)
        TokenExpirationSeconds:  3607
        ConfigMapName:           kube-root-ca.crt
        ConfigMapOptional:       <nil>
        DownwardAPI:             true
        ConfigMapName:           openshift-service-ca.crt
        ConfigMapOptional:       <nil>
    QoS Class:                   BestEffort
    Node-Selectors:              <none>
    Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
    Events:
    Type    Reason          Age   From               Message
    ----    ------          ----  ----               -------
    Normal  Scheduled       14m   default-scheduler  Successfully assigned liberty-to-openshift-instanton/liberty-to-openshift-instanton-85fbc954dd-zmsfs to crc-vlf7c-master-0
    Normal  AddedInterface  14m   multus             Add eth0 [10.217.0.107/23] from openshift-sdn
    Normal  Pulled          14m   kubelet            Container image "image-registry.openshift-image-registry.svc:5000/liberty-to-openshift-instanton/liberty-to-openshift-instanton:olp-java17-1.0" already present on machine
    Normal  Created         14m   kubelet            Created container liberty-to-openshift-instanton
    Normal  Started         14m   kubelet            Started container liberty-to-openshift-instanton

## Tear down the environment

To delete the environment, issue the following commands.

### Select the project

````cmd
oc projects
oc project liberty-to-openshift-instanton
````

### Delete the deployment including the service and route

````cmd
cd /d c:\ocp\LibertyToOpenShiftInstantOn
oc delete -f deploy.yaml
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc delete -f deploy.yaml
    deployment.apps "liberty-to-openshift-instanton" deleted
    service "liberty-to-openshift-instanton" deleted
    route.route.openshift.io "liberty-to-openshift-instanton" deleted

### Delete the pushed image

````cmd
oc get imagestreams
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc get imagestreams
    NAME                             IMAGE REPOSITORY                                                                                                        TAGS             UPDATED
    liberty-to-openshift-instanton   default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift-instanton/liberty-to-openshift-instanton   olp-java17-1.0   2 days ago

````cmd
oc delete imagestream/liberty-to-openshift-instanton
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc delete imagestream/liberty-to-openshift-instanton
    imagestream.image.openshift.io "liberty-to-openshift-instanton" deleted

### Delete the project

````cmd
oc delete project liberty-to-openshift-instanton
oc projects
````

    c:\ocp\LibertyToOpenShiftInstantOn>oc delete project liberty-to-openshift-instanton
    project.project.openshift.io "liberty-to-openshift-instanton" deleted

    c:\ocp\LibertyToOpenShiftInstantOn>oc projects
    You have access to the following projects and can switch between them with ' project <projectname>':

    guide
    liberty-to-openshift

## References

| Reference  | URL |
| :--- | :--- |
| Faster startup with ... Open Liberty InstantOn | https://openliberty.io/docs/latest/instanton.html |
| Linux capabilities. Great read to understan the Linux capabilities.  | https://man7.org/linux/man-pages/man7/capabilities.7.html |
| Hands-on Open Liberty InstantOn. A lab I found early on that helped me during this process. | https://github.com/Emily-Jiang/techxchange-instanton-lab?search=1 |

## Problems encountered building images

During my early attempts to create an InstantOn image I encountered a few problems. This was mainly due to my lack of understanding and experience in building images, in particular Open Liberty InstantOn images.

### Missing options to the build command

The below command is not valid when building an InstantOn image. For some reason, I often forgot to add the additional options.

````cmd
podman build -t liberty-to-openshift-instanton:olp-java17-1.0 -f Containerfile.olp.slim.java17
````

Issuing the above command produces the following  error:

````
[AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
Can't exec criu swrk: Operation not permitted
Can't read request: Connection reset by peer
Can't receive response: Invalid argument
[ERROR   ] CWWKC0453E: The server checkpoint request failed with the following message: Could not dump the JVM processes, err=-70
[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.794 seconds.
[AUDIT   ] CWWKE0084I: The server defaultServer is stopping because thread Checkpoint failed, exiting... (00000061) called the method java.lang.System.exit:
        at java.base/java.lang.System.exit(System.java:519)
        at io.openliberty.checkpoint.internal.CheckpointImpl.lambda$checkpointOrExitOnFailure$5(CheckpointImpl.java:318)
        at java.base/java.lang.Thread.run(Thread.java:857)
````

The correct command is:

````cmd
podman build -t liberty-to-openshift-instanton:olp-java17-1.0 --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
````

The full output from the command will be similar to the following:

````
D:\Sean>podman build -t liberty-to-openshift-instanton:olp-java17-1.0 -f Containerfile.olp.slim.java17
STEP 1/9: FROM icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
STEP 2/9: ARG VERSION=1.0
--> Using cache 03f674df8cedde83c6c01d0bd68032033c696e6ec9d3a14fde3e9524d044d982
--> 03f674df8ce
STEP 3/9: ARG REVISION=SNAPSHOT
--> Using cache 395d58791e35877d227085a2aae7f03e654ac79d88dba56ae83f94baefad1863
--> 395d58791e3
STEP 4/9: LABEL   org.opencontainers.image.authors="Sean Boyd"   org.opencontainers.image.url="local"   org.opencontainers.image.version="$VERSION"   org.opencontainers.image.revision="$REVISION"   name="LibertyToOpenShift"   version="$VERSION-$REVISION"   summary="Sample Open Liberty InstantOn app for deploying to OpenShift"   description="This servlet has been written to help test running an Open Liberty app in OpenShift"
--> bfb0a4c14b7
STEP 5/9: COPY --chown=1001:0 target/server.xml /config/
--> 3b8135ed937
STEP 6/9: RUN features.sh
+ SNIPPETS_SOURCE=/opt/ol/helpers/build/configuration_snippets
+ SNIPPETS_TARGET=/config/configDropins/overrides
+ SNIPPETS_TARGET_DEFAULTS=/config/configDropins/defaults
+ mkdir -p /config/configDropins/overrides
+ mkdir -p /config/configDropins/defaults
+ '[' -n '' ']'
+ '[' '' == client ']'
+ '[' '' == embedded ']'
+ [[ -n '' ]]
+ '[' '' == true ']'
+ '[' '' == true ']'
+ featureUtility installServerFeatures --acceptLicense defaultServer --noCache
+ find /opt/ol/wlp/lib /opt/ol/wlp/bin '!' -perm -g=rw -print0
+ xargs -0 -r chmod g+rw
--> 37584c81c4e
STEP 7/9: COPY --chown=1001:0 target/LibertyToOpenShiftInstantOn.war /config/apps/
--> 5e1b75f11e1
STEP 8/9: RUN configure.sh
+ main
+ WLP_INSTALL_DIR=/opt/ol/wlp
+ SHARED_CONFIG_DIR=/opt/ol/wlp/usr/shared/config
+ SHARED_RESOURCE_DIR=/opt/ol/wlp/usr/shared/resources
+ SNIPPETS_SOURCE=/opt/ol/helpers/build/configuration_snippets
+ SNIPPETS_TARGET=/config/configDropins/overrides
+ SNIPPETS_TARGET_DEFAULTS=/config/configDropins/defaults
+ mkdir -p /config/configDropins/overrides
+ mkdir -p /config/configDropins/defaults
+ [[ -n '' ]]
+ '[' '' == client ']'
+ '[' '' == embedded ']'
+ keystorePath=/config/configDropins/defaults/keystore.xml
+ '[' '' '!=' false ']'
+ '[' '' '!=' false ']'
+ '[' '!' -e /config/configDropins/defaults/keystore.xml ']'
++ openssl rand -base64 32
+ export KEYSTOREPWD=BXwJbKXb6VfabbLp94GJday/keFlJ/1QO7G8ltMsFe8=
+ KEYSTOREPWD=BXwJbKXb6VfabbLp94GJday/keFlJ/1QO7G8ltMsFe8=
+ sed 's|REPLACE|BXwJbKXb6VfabbLp94GJday/keFlJ/1QO7G8ltMsFe8=|g' /opt/ol/helpers/build/configuration_snippets/keystore.xml
+ chmod g+w /config/configDropins/defaults/keystore.xml
+ [[ -n '' ]]
+ find /opt/ol/fixes -type f -name '*.jar' -print0
+ sort -z
+ xargs -0 -n 1 -r -I '{}' java -jar '{}' --installLocation /opt/ol/wlp
+ touch /config/server.xml
+ '[' true == true ']'
+ cmd='populate_scc.sh -i 1'
+ '[' '' == false ']'
+ '[' '!' '' = '' ']'
+ '[' '' = false ']'
+ '[' '!' '' = '' ']'
+ '[' '' = false ']'
+ '[' '!' '' = '' ']'
+ eval populate_scc.sh -i 1
++ populate_scc.sh -i 1
+ SCC_SIZE=80m
+ ITERATIONS=2
+ TRIM_SCC=yes
+ WARM_ENDPOINT=true
+ WARM_ENDPOINT_URL=localhost:9080/
+ WARM_OPENAPI_ENDPOINT=true
+ WARM_OPENAPI_ENDPOINT_URL=localhost:9080/openapi
+ [[ -d /opt/java/.scc ]]
++ stat -L -c %a /opt/java/.scc
++ cut -c 1,2
+ [[ 77 == \7\7 ]]
+ SCC=-Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc
+ export 'OPENJ9_JAVA_OPTIONS=-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc'
+ OPENJ9_JAVA_OPTIONS='-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc'
+ export 'IBM_JAVA_OPTIONS=-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc'
+ IBM_JAVA_OPTIONS='-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc'
+ CREATE_LAYER='-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,createLayer,groupAccess'
+ DESTROY_LAYER='-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,destroy'
+ PRINT_LAYER_STATS='-XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,printTopLayerStats'
+ getopts :i:s:u:o:tdhwcml OPT
+ case "$OPT" in
+ ITERATIONS=1
+ getopts :i:s:u:o:tdhwcml OPT
++ umask
+ OLD_UMASK=0022
+ umask 002
+ java -XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,createLayer,groupAccess -Xscmx80m -version
openjdk version "17.0.11" 2024-04-16
IBM Semeru Runtime Open Edition 17.0.11.0 (build 17.0.11+9)
Eclipse OpenJ9 VM 17.0.11.0 (build openj9-0.44.0, JRE 17 Linux amd64-64-Bit Compressed References 20240416_760 (JIT enabled, AOT enabled)
OpenJ9   - b0699311c7
OMR      - 254af5a04
JCL      - 5d7d758b682 based on jdk-17.0.11+9)
+ '[' yes == yes ']'
+ echo 'Calculating SCC layer upper bound, starting with initial size 80m.'
+ /opt/ol/wlp/bin/server start
+ '[' true == true ']'
+ curl --silent --output /dev/null --show-error --fail --max-time 5 localhost:9080/
+ '[' true == true ']'
+ curl --silent --output /dev/null --show-error --fail --max-time 5 localhost:9080/openapi
+ /opt/ol/wlp/bin/server stop
++ awk '/^Cache is [0-9.]*% .*full/ {print substr($3, 1, length($3)-1)}'
+ FULL=25
+ echo 'SCC layer is 25% full. Destroying layer.'
+ java -XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,destroy
JVMSHRC806I Compressed references persistent shared cache "openj9_system_scc" has been destroyed. Use option -Xnocompressedrefs if you want to destroy a non-compressed references cache.
+ true
+ SCC_SIZE=80
++ awk 'BEGIN {print int(80 * 25 / 100.0 + 0.5)}'
+ SCC_SIZE=20
+ '[' 20 -eq 0 ']'
+ SCC_SIZE=20m
+ echo 'Re-creating layer with size 20m.'
+ java -XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,createLayer,groupAccess -Xscmx20m -version
openjdk version "17.0.11" 2024-04-16
IBM Semeru Runtime Open Edition 17.0.11.0 (build 17.0.11+9)
Eclipse OpenJ9 VM 17.0.11.0 (build openj9-0.44.0, JRE 17 Linux amd64-64-Bit Compressed References 20240416_760 (JIT enabled, AOT enabled)
OpenJ9   - b0699311c7
OMR      - 254af5a04
JCL      - 5d7d758b682 based on jdk-17.0.11+9)
+ (( i=0 ))
+ (( i<1 ))
+ /opt/ol/wlp/bin/server start
+ '[' true == true ']'
+ curl --silent --output /dev/null --show-error --fail --max-time 5 localhost:9080/
+ '[' true == true ']'
+ curl --silent --output /dev/null --show-error --fail --max-time 5 localhost:9080/openapi
+ /opt/ol/wlp/bin/server stop
+ (( i++ ))
+ (( i<1 ))
+ umask 0022
+ rm -rf /output/messaging /logs/console.log /logs/messages_24.08.12_19.04.11.0.log /logs/messages.log /logs/verbosegc.001.log /logs/verbosegc.002.log /opt/ol/wlp/output/.classCache
+ chmod -R g+rwx /output/workarea
+ [[ -d /output/resources ]]
++ awk '/^Cache is [0-9.]*% .*full/ {print substr($3, 1, length($3)-1)}'
+ FULL=54
+ echo 'SCC layer is 54% full.'
--> 3bb93545eaa
STEP 9/9: RUN checkpoint.sh afterAppStart
Performing checkpoint --at=afterAppStart

Launching defaultServer (Open Liberty 24.0.0.6/wlp-1.0.90.cl240620240603-2001) on Eclipse OpenJ9 VM, version 17.0.11+9 (en_US)
[AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
[AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.167 seconds.
[AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
Can't exec criu swrk: Operation not permitted
Can't read request: Connection reset by peer
Can't receive response: Invalid argument
[ERROR   ] CWWKC0453E: The server checkpoint request failed with the following message: Could not dump the JVM processes, err=-70
[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.794 seconds.
[AUDIT   ] CWWKE0084I: The server defaultServer is stopping because thread Checkpoint failed, exiting... (00000061) called the method java.lang.System.exit:
        at java.base/java.lang.System.exit(System.java:519)
        at io.openliberty.checkpoint.internal.CheckpointImpl.lambda$checkpointOrExitOnFailure$5(CheckpointImpl.java:318)
        at java.base/java.lang.Thread.run(Thread.java:857)

[AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
[AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
[AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.826 seconds.
Error: error building at STEP "RUN checkpoint.sh afterAppStart": error while running runtime: exit status 74
````

### Build on Windows and not UNIX

Not understanding the importance of building Open Liberty InstantOn images on the correct platform, I issued the correct command but on Windows and not UNIX.

````cmd
podman build -t liberty-to-openshift-instanton:olp-java17-1.0 --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
````

The following error appeared:

````
[AUDIT   ] CWWKE0084I: The server defaultServer is stopping because thread Checkpoint failed, exiting... (00000061) called the method java.lang.System.exit:
        at java.base/java.lang.System.exit(System.java:519)
        at io.openliberty.checkpoint.internal.CheckpointImpl.lambda$checkpointOrExitOnFailure$5(CheckpointImpl.java:318)
        at java.base/java.lang.Thread.run(Thread.java:857)

[AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
[AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
[AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.925 seconds.
CWWKE0962E: The server checkpoint request failed. The following output is from the CRIU /logs/checkpoint/checkpoint.log file that contains details on why the checkpoint failed.
Warn  (criu/kerndat.c:1153): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
Warn  (criu/kerndat.c:1153): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
Warn  (compel/src/lib/infect.c:133): Unable to interrupt task: 1098 (Operation not permitted)
Error (criu/proc_parse.c:379): Failed to resolve mapping 5593b80a5000 filename
Error (criu/proc_parse.c:693): Can't open 1021's mapfile link 5593b80a5000: Operation not permitted
Error (criu/cr-dump.c:1563): Collect mappings (pid: 1021) failed with -1
Error (criu/cr-dump.c:2098): Dumping FAILED.
Error: error building at STEP "RUN checkpoint.sh afterAppStart": error while running runtime: exit status 74
````

The full output from the command will be similar to:

````
D:\Sean>podman build -t liberty-to-openshift-instanton:olp-java17-1.0 --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
STEP 1/9: FROM icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
STEP 2/9: ARG VERSION=1.0
--> Using cache 03f674df8cedde83c6c01d0bd68032033c696e6ec9d3a14fde3e9524d044d982
--> 03f674df8ce
STEP 3/9: ARG REVISION=SNAPSHOT
--> Using cache 395d58791e35877d227085a2aae7f03e654ac79d88dba56ae83f94baefad1863
--> 395d58791e3
STEP 4/9: LABEL   org.opencontainers.image.authors="Sean Boyd"   org.opencontainers.image.url="local"   org.opencontainers.image.version="$VERSION"   org.opencontainers.image.revision="$REVISION"   name="LibertyToOpenShift"   version="$VERSION-$REVISION"   summary="Sample Open Liberty InstantOn app for deploying to OpenShift"   description="This servlet has been written to help test running an Open Liberty app in OpenShift"
--> Using cache bfb0a4c14b7863e6bd275834c394a3d258273930f65c60b173e5bc152590d62e
--> bfb0a4c14b7
STEP 5/9: COPY --chown=1001:0 target/server.xml /config/
--> Using cache 3b8135ed937bb83e8617681abe6329b48abfa4ed32ff5ddbc6f67e30f94bc4d2
--> 3b8135ed937
STEP 6/9: RUN features.sh
--> Using cache 37584c81c4edf0b3c0c75c3b969df080399157218ab4e758e3435494e4052d05
--> 37584c81c4e
STEP 7/9: COPY --chown=1001:0 target/LibertyToOpenShiftInstantOn.war /config/apps/
--> Using cache 5e1b75f11e19be5b6dd3a1bdcb8c71d086b25f78343e7a4a49626ed015584458
--> 5e1b75f11e1
STEP 8/9: RUN configure.sh
--> Using cache 3bb93545eaac5cadbfa21f61591e239d7a0640a993a76a45c016f3113407c1d9
--> 3bb93545eaa
STEP 9/9: RUN checkpoint.sh afterAppStart
Performing checkpoint --at=afterAppStart

Launching defaultServer (Open Liberty 24.0.0.6/wlp-1.0.90.cl240620240603-2001) on Eclipse OpenJ9 VM, version 17.0.11+9 (en_US)
[AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
[AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.190 seconds.
[AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
[ERROR   ] CWWKC0453E: The server checkpoint request failed with the following message: Could not dump the JVM processes, err=-52
[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.896 seconds.
[AUDIT   ] CWWKE0084I: The server defaultServer is stopping because thread Checkpoint failed, exiting... (00000061) called the method java.lang.System.exit:
        at java.base/java.lang.System.exit(System.java:519)
        at io.openliberty.checkpoint.internal.CheckpointImpl.lambda$checkpointOrExitOnFailure$5(CheckpointImpl.java:318)
        at java.base/java.lang.Thread.run(Thread.java:857)

[AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
[AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
[AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.925 seconds.
CWWKE0962E: The server checkpoint request failed. The following output is from the CRIU /logs/checkpoint/checkpoint.log file that contains details on why the checkpoint failed.
Warn  (criu/kerndat.c:1153): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
Warn  (criu/kerndat.c:1153): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
Warn  (compel/src/lib/infect.c:133): Unable to interrupt task: 1098 (Operation not permitted)
Error (criu/proc_parse.c:379): Failed to resolve mapping 5593b80a5000 filename
Error (criu/proc_parse.c:693): Can't open 1021's mapfile link 5593b80a5000: Operation not permitted
Error (criu/cr-dump.c:1563): Collect mappings (pid: 1021) failed with -1
Error (criu/cr-dump.c:2098): Dumping FAILED.
Error: error building at STEP "RUN checkpoint.sh afterAppStart": error while running runtime: exit status 74
````

### setsebool virt_sandbox_use_netlink 1

When building an image you maye see the below error.

````cmd
podman build -t libertytoopenshiftinstanton:1.0-SNAPSHOT --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
````

    Error (criu/proc_parse.c:379): Failed to resolve mapping 55a65bfe8000 filename
    Error (criu/proc_parse.c:694): Can't open 1023's mapfile link 55a65bfe8000: Operation not permitted
    Error (criu/cr-dump.c:1558): Collect mappings (pid: 1023) failed with -1
    Error (criu/cr-dump.c:2093): Dumping FAILED.
    CWWKE0963E: The server checkpoint request failed because netlink system calls were unsuccessful. If SELinux is enabled in enforcing mode, netlink system calls might be blocked by the SELinux "virt_sandbox_use_netlink" policy setting. Either disable SELinux or enable the netlink system calls with the "setsebool virt_sandbox_use_netlink 1" command.
    Error: building at STEP "RUN checkpoint.sh afterAppStart": while running runtime: exit status 74

To fix this, issue the following command under root.

````cmd
setsebool virt_sandbox_use_netlink 1
````

The below contains the full output. You may notice that I issued the build command ujsing my user id 'sean' rather than 'root'. After you fix this immediate issue you'll face a further issue due to the build not using the root account.

    [sean@localhost LibertyToOpenShiftInstantOn]$ podman build -t libertytoopenshiftinstanton:1.0-SNAPSHOT --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
    STEP 1/9: FROM icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
    Trying to pull icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi...
    Getting image source signatures
    Checking if image destination supports signatures
    Copying blob 8f22e9941898 done
    Copying blob 3a5826b65e10 done
    Copying blob d90ec51f9710 done
    Copying blob 54c06979847e done
    Copying blob daa19010dafa done
    Copying blob 4e9a7c1fc5cd done
    Copying blob 7d7147923259 done
    Copying blob 8aebe929fca6 done
    Copying blob 357b19ceed78 done
    Copying blob 577f53465011 done
    Copying blob 24b59405bacd done
    Copying blob 76df7b4cb448 done
    Copying blob 7e244e59eaca done
    Copying blob c109aab1e70f done
    Copying blob 4a8886590402 done
    Copying blob 9a2d37d61430 done
    Copying blob 6957698accc1 done
    Copying config 385d889567 done
    Writing manifest to image destination
    Storing signatures
    STEP 2/9: ARG VERSION=1.0
    --> 4d4d9639717
    STEP 3/9: ARG REVISION=SNAPSHOT
    --> 67d515c6621
    STEP 4/9: LABEL   org.opencontainers.image.authors="Sean Boyd"   org.opencontainers.image.url="local"   org.opencontainers.image.version="$VERSION"   org.opencontainers.image.revision="$REVISION"   name="LibertyToOpenShift"   version="$VERSION-$REVISION"   summary="Quick guide to go from Liberty to Liberty on OpenShift"   description="This image has been tested in Podman and OpenShift Local."
    --> 737da430adc
    STEP 5/9: COPY --chown=1001:0 target/server.xml /config/
    --> 740d285de0b
    STEP 6/9: RUN features.sh
    + SNIPPETS_SOURCE=/opt/ol/helpers/build/configuration_snippets
    + SNIPPETS_TARGET=/config/configDropins/overrides
    + SNIPPETS_TARGET_DEFAULTS=/config/configDropins/defaults
    + mkdir -p /config/configDropins/overrides
    + mkdir -p /config/configDropins/defaults
    + '[' -n '' ']'
    + '[' '' == client ']'
    + '[' '' == embedded ']'
    + [[ -n '' ]]
    + '[' '' == true ']'
    + '[' '' == true ']'
    + featureUtility installServerFeatures --acceptLicense defaultServer --noCache
    + find /opt/ol/wlp/lib /opt/ol/wlp/bin '!' -perm -g=rw -print0
    + xargs -0 -r chmod g+rw
    --> b9d125bbd00
    STEP 7/9: COPY --chown=1001:0 target/LibertyToOpenShiftInstantOn.war /config/apps/
    --> 75daa47a5dc
    STEP 8/9: RUN configure.sh
    --> 247528a7524
    STEP 9/9: RUN checkpoint.sh afterAppStart
    Performing checkpoint --at=afterAppStart

    Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.760 seconds.
    [AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
    [ERROR   ] CWWKC0453E: The server checkpoint request failed with the following message: Could not dump the JVM processes, err=-52
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 5.579 seconds.
    [AUDIT   ] CWWKE0084I: The server defaultServer is stopping because thread Checkpoint failed, exiting... (00000045) called the method java.lang.System.exit:
            at java.base/java.lang.System.exit(System.java:517)
            at io.openliberty.checkpoint.internal.CheckpointImpl.lambda$checkpointOrExitOnFailure$6(CheckpointImpl.java:334)
            at java.base/java.lang.Thread.run(Thread.java:857)

    [AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 5.744 seconds.
    [AUDIT   ] CWWKZ0009I: The application LibertyToOpenShiftInstantOn has stopped successfully.
    [AUDIT   ] CWWKE0036I: The server defaultServer stopped after 6.179 seconds.
    CWWKE0962E: The server checkpoint request failed. The following output is from the CRIU /logs/checkpoint/checkpoint.log file that contains details on why the checkpoint failed.
    Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
    Warn  (compel/src/lib/infect.c:133): Unable to interrupt task: 1088 (Operation not permitted)
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Warn  (criu/libnetlink.c:84): Can't send request message
    Error (criu/proc_parse.c:379): Failed to resolve mapping 55a65bfe8000 filename
    Error (criu/proc_parse.c:694): Can't open 1023's mapfile link 55a65bfe8000: Operation not permitted
    Error (criu/cr-dump.c:1558): Collect mappings (pid: 1023) failed with -1
    Error (criu/cr-dump.c:2093): Dumping FAILED.
    CWWKE0963E: The server checkpoint request failed because netlink system calls were unsuccessful. If SELinux is enabled in enforcing mode, netlink system calls might be blocked by the SELinux "virt_sandbox_use_netlink" policy setting. Either disable SELinux or enable the netlink system calls with the "setsebool virt_sandbox_use_netlink 1" command.
    Error: building at STEP "RUN checkpoint.sh afterAppStart": while running runtime: exit status 74

### (criu/proc_parse.c:694): Can't open 1023's mapfile link 558982528000: Operation not permitted

If after setting the following

````cmd
sudo setsebool virt_sandbox_use_netlink 1
````

    [sean@localhost LibertyToOpenShiftInstantOn]$ sudo setsebool virt_sandbox_use_netlink 1
    [sudo] password for sean:

And you get the below error when building the image, check whether you are logged into your user id or root. In my case, I was logged into my user id.

````cmd
podman build -t libertytoopenshiftinstanton:1.0-SNAPSHOT --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
````

	[sean@localhost LibertyToOpenShiftInstantOn]$ podman build -t libertytoopenshiftinstanton:1.0-SNAPSHOT --cap-add=CHECKPOINT_RESTORE --cap-add=SYS_PTRACE --cap-add=SETPCAP --security-opt seccomp=unconfined -f Containerfile.olp.slim.java17
	STEP 1/9: FROM icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
	STEP 2/9: ARG VERSION=1.0
	--> Using cache 4d4d96397170ce8da95d31e600d47724e21b9a18e15b9a22bb58d8f1a851ed1f
	--> 4d4d9639717
	STEP 3/9: ARG REVISION=SNAPSHOT
	--> Using cache 67d515c6621976c2e1facbd983721cb7d5cad72075d3cfe8e31b12062b7ed7d5
	--> 67d515c6621
	STEP 4/9: LABEL   org.opencontainers.image.authors="Sean Boyd"   org.opencontainers.image.url="local"   org.opencontainers.image.version="$VERSION"   org.opencontainers.image.revision="$REVISION"   name="LibertyToOpenShift"   version="$VERSION-$REVISION"   summary="Quick guide to go from Liberty to Liberty on OpenShift"   description="This image has been tested in Podman and OpenShift Local."
	--> Using cache 737da430adcb79a38e3d43d9fdfbb74f4b554dc62a6c7407532570bfd2839dbe
	--> 737da430adc
	STEP 5/9: COPY --chown=1001:0 target/server.xml /config/
	--> Using cache 740d285de0b3f3865a4cd50b0a28f47c0022134c05555ad3cfff797ee436f97f
	--> 740d285de0b
	STEP 6/9: RUN features.sh
	--> Using cache b9d125bbd00f527b20eeeec0d35ed487cd0df7e2d037dba1f79fc578717d2044
	--> b9d125bbd00
	STEP 7/9: COPY --chown=1001:0 target/LibertyToOpenShiftInstantOn.war /config/apps/
	--> Using cache 75daa47a5dc51631bcf821fe01656681dc94abb538cc09304abb3b1322a90412
	--> 75daa47a5dc
	STEP 8/9: RUN configure.sh
	--> Using cache 247528a75246e4e823d07e62af592ac9702df99335e43b5ee3e9ee5f92910ad5
	--> 247528a7524
	STEP 9/9: RUN checkpoint.sh afterAppStart
	Performing checkpoint --at=afterAppStart

	Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
	[AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
	[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
	[AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
	[AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
	[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 0.768 seconds.
	[AUDIT   ] CWWKC0451I: A server checkpoint "afterAppStart" was requested. When the checkpoint completes, the server stops.
	[ERROR   ] CWWKC0453E: The server checkpoint request failed with the following message: Could not dump the JVM processes, err=-52
	[AUDIT   ] CWWKZ0001I: Application LibertyToOpenShiftInstantOn started in 5.973 seconds.
	[AUDIT   ] CWWKE0084I: The server defaultServer is stopping because thread Checkpoint failed, exiting... (00000045) called the method java.lang.System.exit:
			at java.base/java.lang.System.exit(System.java:517)
			at io.openliberty.checkpoint.internal.CheckpointImpl.lambda$checkpointOrExitOnFailure$6(CheckpointImpl.java:334)
			at java.base/java.lang.Thread.run(Thread.java:857)

	[AUDIT   ] CWWKE1100I: Waiting for up to 30 seconds for the server to quiesce.
	[AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
	[AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 6.156 seconds.
	[AUDIT   ] CWWKZ0009I: The application LibertyToOpenShiftInstantOn has stopped successfully.
	[AUDIT   ] CWWKE0036I: The server defaultServer stopped after 7.514 seconds.
	CWWKE0962E: The server checkpoint request failed. The following output is from the CRIU /logs/checkpoint/checkpoint.log file that contains details on why the checkpoint failed.
	Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
	Warn  (criu/kerndat.c:1103): $XDG_RUNTIME_DIR not set. Cannot find location for kerndat file
	Warn  (compel/src/lib/infect.c:133): Unable to interrupt task: 1088 (Operation not permitted)
	Error (criu/proc_parse.c:379): Failed to resolve mapping 558982528000 filename
	Error (criu/proc_parse.c:694): Can't open 1023's mapfile link 558982528000: Operation not permitted
	Error (criu/cr-dump.c:1558): Collect mappings (pid: 1023) failed with -1
	Error (criu/cr-dump.c:2093): Dumping FAILED.
	Error: building at STEP "RUN checkpoint.sh afterAppStart": while running runtime: exit status 74
	[sean@localhost LibertyToOpenShiftInstantOn]$


