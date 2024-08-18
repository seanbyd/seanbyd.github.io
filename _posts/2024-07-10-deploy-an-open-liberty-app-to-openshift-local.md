# How to deploy an Open Liberty app to OpenShift Local

This guide steps you through the process of deploying a basic [Open Liberty](https://openliberty.io/) app to [OpenShift Local](https://developers.redhat.com/products/openshift-local/overview) running on your PC.

## Audience

The audience of this guide is anyone having an interest to deploy their first Open Liberty app to OpenShift.

While this guide installs to OpenShift Local on your PC, the same process can be followed when using a full installation of OpenShift.

The guide has been written with an admin in mind who may have little or no development experience. Me, I fit somewhere in-between.

## Pre-requisites

For this exercise you need to:

- [Install OpenShift Local](https://thechalkboards.com/install-openshift-local-to-my-windows-pc/) to your PC.

- While not mandatory, it is recommended to download **git** and **github desktop**. You can happily complete this guide by ignoring this step.


## Environment

This guide was completed using the following environment.

| Category | Version  | URL |
| :--- | :--- | :--- |
| OpenShift Local | **crc version**<br>CRC version: 2.33.0+c43b17<br>OpenShift version: 4.14.12<br>Podman version: 4.4.4 | [https://seanbyd.github.io/2024/03/13/installOpenShiftLocal.html](https://seanbyd.github.io/2024/03/13/installOpenShiftLocal.html) |
| Open Liberty  | 24.0.0.2<br>All GA features:<br>openliberty-24.0.0.2.zip | [https://openliberty.io/start/](https://openliberty.io/start/) |
| Java  | IBM Semeru Java 17<br>ibm-semeru-open-jdk_x64_windows_17.0.8.1_1_openj9-0.40.0.zip | [https://developer.ibm.com/languages/java/semeru-runtimes/downloads/](https://developer.ibm.com/languages/java/semeru-runtimes/downloads/) |

## Sample app

For this exercise, I wrote a [basic servlet](https://github.com/seanbyd/LibertyToOpenShift) to deploy to OpenShift. The goal being to learn about deploying apps to OpenShift, rather than developing apps for running in OpenShift.

To complete this guide you need to download this sample app.

### Download the sample project

Download the [sample project](https://github.com/seanbyd/LibertyToOpenShift) from github.

You can [clone the project](https://github.com/seanbyd/LibertyToOpenShift.git) using git or github desktop.

Alternatively you can download the [zip file](https://github.com/seanbyd/LibertyToOpenShift/archive/refs/heads/main.zip). Unzip to a location of your choice. For the non-developers, or those not experienced with git and github, I have followed this approach.

### Unzip the sample project

Unzip the downloaded file *LibertyToOpenShift-main.zip* to c:/ocp/LibertyToOpenShift.

Open a command window and change to the project directory.

````cmd
cd /d c:/ocp/LibertyToOpenShift
unzip LibertyToOpenShift-main.zip using a tool of your choice
````

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

### Prepare OC CLI

To use the OC CLI you'll need to set some variables. If you miss this step, you'll receive errors later on.

````cmd
@FOR /f "tokens=*" %i IN ('crc oc-env') DO @call %i
````

### (Optionally) Remove old podman images

Optionally delete any old podman images you no longer require.

| Command| Help |
| :--- | :--- |
| podman images | Display all the podman images  |
| podman rmi IMAGE_ID | Delete the podman image with image id IMAGE_ID |
| podman rmi REPOSITORY:TAG | Delete the podman image matching the combination of REPOSITORY name and TAG |
| podman rmi IMAGE_ID --force | This deletes all images including any tagged images |

````cmd
podman images
podman rmi 0e1aa95417a0
podman rmi icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
podman rmi 11e9b7915530 --force
````

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY                                                                                     TAG                            IMAGE ID      CREATED       SIZE
    localhost/libertytoopenshift                                                                   1.0-SNAPSHOT                   11e9b7915530  3 months ago  738 MB
    default-route-openshift-image-registry.apps-crc.testing/libertytoopenshift/libertytoopenshift  1.0-SNAPSHOT                   11e9b7915530  3 months ago  738 MB
    icr.io/appcafe/open-liberty                                                                    kernel-slim-java17-openj9-ubi  b97405ff9ff2  3 months ago  687 MB
    icr.io/appcafe/open-liberty                                                                    kernel-slim-java11-openj9-ubi  0e1aa95417a0  3 months ago  669 MB
    icr.io/ibm-messaging/mq                                                                        latest                         9e0011332935  9 months ago  1.62 GB

    c:\ocp\LibertyToOpenShift>podman rmi 0e1aa95417a0
    Untagged: icr.io/appcafe/open-liberty:kernel-slim-java11-openj9-ubi
    Deleted: 0e1aa95417a0f7d01fcadb4a1cfb62db0ce097c638cc619d3acb7ee31070022b

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY                                                                                     TAG                            IMAGE ID      CREATED       SIZE
    localhost/libertytoopenshift                                                                   1.0-SNAPSHOT                   11e9b7915530  3 months ago  738 MB
    default-route-openshift-image-registry.apps-crc.testing/libertytoopenshift/libertytoopenshift  1.0-SNAPSHOT                   11e9b7915530  3 months ago  738 MB
    icr.io/appcafe/open-liberty                                                                    kernel-slim-java17-openj9-ubi  b97405ff9ff2  3 months ago  687 MB
    icr.io/ibm-messaging/mq                                                                        latest                         9e0011332935  9 months ago  1.62 GB

    c:\ocp\LibertyToOpenShift>podman rmi icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
    Untagged: icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY                                                                                     TAG           IMAGE ID      CREATED       SIZE
    localhost/libertytoopenshift                                                                   1.0-SNAPSHOT  11e9b7915530  3 months ago  738 MB
    default-route-openshift-image-registry.apps-crc.testing/libertytoopenshift/libertytoopenshift  1.0-SNAPSHOT  11e9b7915530  3 months ago  738 MB
    icr.io/ibm-messaging/mq                                                                        latest        9e0011332935  9 months ago  1.62 GB

    c:\ocp\LibertyToOpenShift>podman rmi 11e9b7915530
    Error: unable to delete image "11e9b79155302d2643fe41e93b02a997b5ee292b5078f55b3e053cd3dfaa0858" by ID with more than one tag ([localhost/libertytoopenshift:1.0-SNAPSHOT default-route-openshift-image-registry.apps-crc.testing/libertytoopenshift/libertytoopenshift:1.0-SNAPSHOT]): please force removal

    c:\ocp\LibertyToOpenShift>podman rmi 11e9b7915530 --force
    Untagged: localhost/libertytoopenshift:1.0-SNAPSHOT
    Untagged: default-route-openshift-image-registry.apps-crc.testing/libertytoopenshift/libertytoopenshift:1.0-SNAPSHOT
    Deleted: 11e9b79155302d2643fe41e93b02a997b5ee292b5078f55b3e053cd3dfaa0858
    Deleted: da0b8c7287ce51ba66ee583375adc712487093702a63e74cc07a4a764c567707
    Deleted: 6efd16647b6311c86fa9a7b6799afe877b7a4f586db4853c76a6ba9186d0a8ab
    Deleted: 9b768d2566ce1f1f1d7cadc484e497025b30b489dad803385ac45318d813d1d9
    Deleted: 1222697c7fee41b72cd08c308e645a09a88b506281aafee16b4957329006ea10
    Deleted: 7480c0dd974212946186cb1ce2a3a69e1fb4d5f63664732a9b6bd5d72d12c95e
    Deleted: 84146279dff5cc08c74633c4955ef33b935af7b4073ad92e58768abc01b7f458

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY               TAG         IMAGE ID      CREATED       SIZE
    icr.io/ibm-messaging/mq  latest      9e0011332935  9 months ago  1.62 GB

### Build the image

Use podman to build the image.

If the image *icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi* has not already been downloaded, it will be automatically downloaded for you.

Ensure you are in the correct directory containing the file *Containerfile.olp.slim.java17*.

````cmd
cd /d c:/ocp/LibertyToOpenShift
podman build -t liberty-to-openshift:olp-java17-1.0 -f Containerfile.olp.slim.java17
````

    c:\ocp\LibertyToOpenShift>podman build -t liberty-to-openshift:olp-java17-1.0 -f Containerfile.olp.slim.java17
    STEP 1/8: FROM icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi
    Trying to pull icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi...
    Getting image source signatures
    Checking if image destination supports signatures
    Copying blob sha256:c7dbe24f5a00dddc9e94ac123c1074ba0e274471d416b686bd0bc50d1ccfc765
    Copying blob sha256:93ea1869a217544ebfd76beb58163674ee0703c139c90a3638f86664e59b779a
    Copying blob sha256:8e5bda555ea4b8dde4495a9fe2916c0faac04bea6095d915a9c371c782958c7c
    Copying blob sha256:fa7082d03c0eafc1a609cca7bea75e1175fd7d76e08bdbc7c24be16dfe415f92
    Copying blob sha256:fc3015c726e80fab591df2d32eeb3f8a66778b60fad0e0bf3101f2e627f01896
    Copying blob sha256:75ff2598460fed52f1bc3f9dd15ccfe506cbe4d4d3c5aebd8adcd4d15e8108fe
    Copying blob sha256:a47d643df641ef50875b37e80f68973305d970827f3f1675b6cb71fd98e0a78f
    Copying blob sha256:2a9f5c8721ae1b125e5918f3e73e0d468b58af219bec2a7edbc4e8cd54c83a3a
    Copying blob sha256:8edff4a27bb056a3c0400268d7e6ebb9aad74802d72f76eb3746245d45d4bc90
    Copying blob sha256:fb29b5d59484cb292ad0cb7ffa1309fc166c6143871b64cc47d2d67f2e980558
    Copying blob sha256:c89582f2d9a7f30a5193c0e83c967cf5451d5e579323129cf6e3e7191d1f1b6f
    Copying blob sha256:0de6c2b9f4c711537e20549fa96c36af8dfc15928d819d0ab387039830871e6f
    Copying blob sha256:11a56550594c55f775eb697cc4d91d6a4e774768f8963c64a549a3c4c8320cdb
    Copying blob sha256:7996a227101de05e67c2c2dededdba2fcf6e5e769060698a74e4fb8dd22c9b9e
    Copying blob sha256:b79cab41a1a55249eb77f383ba9ae4521f0feef6bd7e84f47d6d2480120f2764
    Copying blob sha256:36241880968fda69352d0bd6284322cedd8b3cfb77437417c5d6c78352bd01a3
    Copying blob sha256:f0db7275d2cad5e288d3a63ce9845040ffdeeb0d53dbc0cfd374c49cd460aaf1
    Copying config sha256:b5cc5094dd1f9428ffb0fe0e149915bb2aed7379025eac25f04dfbd266a2477a
    Writing manifest to image destination
    Storing signatures
    STEP 2/8: ARG VERSION=1.0
    --> bd1bf740bf6
    STEP 3/8: ARG REVISION=SNAPSHOT
    --> 624457344c7
    STEP 4/8: LABEL   org.opencontainers.image.authors="Sean Boyd"   org.opencontainers.image.url="local"   org.opencontainers.image.version="$VERSION"   org.opencontainers.image.revision="$REVISION"   name="LibertyToOpenShift"   version="$VERSION-$REVISION"   summary="Sample Open Liberty app for deploying to OpenShift"   description="This servlet has been writtent to help test running an Open Liberty app in OpenShift"
    --> 8d2eb52ae6d
    STEP 5/8: COPY --chown=1001:0 target/server.xml /config/
    --> d1a9d5de015
    STEP 6/8: RUN features.sh
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
    + xargs -0 -r chmod g+rw
    + find /opt/ol/wlp/lib /opt/ol/wlp/bin '!' -perm -g=rw -print0
    --> 221897bf5a2
    STEP 7/8: COPY --chown=1001:0 target/LibertyToOpenShift.war /config/apps/
    --> b7ec3ffcb6a
    STEP 8/8: RUN configure.sh
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
    + export KEYSTOREPWD=/n0ItOzsiVhcYXpMvSMUM152FP12VJTsLua2nqVLznQ=
    + KEYSTOREPWD=/n0ItOzsiVhcYXpMvSMUM152FP12VJTsLua2nqVLznQ=
    + sed 's|REPLACE|/n0ItOzsiVhcYXpMvSMUM152FP12VJTsLua2nqVLznQ=|g' /opt/ol/helpers/build/configuration_snippets/keystore.xml
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
    + FULL=24
    + echo 'SCC layer is 24% full. Destroying layer.'
    + java -XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,destroy
    JVMSHRC806I Compressed references persistent shared cache "openj9_system_scc" has been destroyed. Use option -Xnocompressedrefs if you want to destroy a non-compressed references cache.
    + true
    + SCC_SIZE=80
    ++ awk 'BEGIN {print int(80 * 24 / 100.0 + 0.5)}'
    + SCC_SIZE=19
    + '[' 19 -eq 0 ']'
    + SCC_SIZE=19m
    + echo 'Re-creating layer with size 19m.'
    + java -XX:+OriginalJDK8HeapSizeCompatibilityMode -XX:+IProfileDuringStartupPhase -Xshareclasses:name=openj9_system_scc,cacheDir=/opt/java/.scc,createLayer,groupAccess -Xscmx19m -version
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
    + rm -rf /output/messaging /logs/console.log /logs/messages_24.06.29_19.38.37.0.log /logs/messages.log /logs/verbosegc.001.log /logs/verbosegc.002.log /opt/ol/wlp/output/.classCache
    + chmod -R g+rwx /output/workarea
    + [[ -d /output/resources ]]
    ++ awk '/^Cache is [0-9.]*% .*full/ {print substr($3, 1, length($3)-1)}'
    + FULL=56
    + echo 'SCC layer is 56% full.'
    COMMIT liberty-to-openshift:olp-java17-1.0
    --> 71abf2ac483
    Successfully tagged localhost/liberty-to-openshift:olp-java17-1.0
    71abf2ac4835cc0d64e397eeab06438f7bc0b09b93f0e887d765ea90f7899a2f

### Verify that the image has been created

Use podman to display the images.

Look for the image you just created named *localhost/liberty-to-openshift:olp-java17-1.0*.

If you didn't already download the image *icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi* you'll notice that it also exists.

````cmd
podman images
````

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY                      TAG                            IMAGE ID      CREATED         SIZE
    localhost/liberty-to-openshift  olp-java17-1.0                 71abf2ac4835  29 seconds ago  740 MB
    icr.io/appcafe/open-liberty     kernel-slim-java17-openj9-ubi  b5cc5094dd1f  5 days ago      690 MB
    icr.io/ibm-messaging/mq         latest                         9e0011332935  9 months ago    1.62 GB

### (Optionally) Test the image

Optionally test the image using podman before you deploy to OpenShift.

#### Start the podman container

````cmd
podman run -d --name A_NAME_OF_YOUR_CHOICE -p PORT_YOU_WILL_ACCESS:INTERNAL_PORT_IN_SERVER_XML REPOSITORY:TAG
````

````cmd
podman run -d --name liberty-to-openshift -p 9080:9080 liberty-to-openshift:olp-java17-1.0
podman ps
````

    c:\ocp\LibertyToOpenShift>podman run -d --name liberty-to-openshift -p 9080:9080 liberty-to-openshift:olp-java17-1.0
    176c27c31d4486f88677418b6976e97a6b38585e44af54efe559b2418b9a641f

    c:\ocp\LibertyToOpenShift>podman ps
    CONTAINER ID  IMAGE                                          COMMAND               CREATED        STATUS        PORTS                   NAMES
    176c27c31d44  localhost/liberty-to-openshift:olp-java17-1.0  /opt/ol/wlp/bin/s...  3 seconds ago  Up 4 seconds  0.0.0.0:9080->9080/tcp  liberty-to-openshift

#### Check the logs

To check whether the container is running fine, check the logs.

| Command| Help |
| :--- | :--- |
| podman logs NAME | Display the container logs |

````cmd
podman logs liberty-to-openshift
````

    c:\ocp\LibertyToOpenShift>podman logs liberty-to-openshift

    Launching defaultServer (Open Liberty 24.0.0.6/wlp-1.0.90.cl240620240603-2001) on Eclipse OpenJ9 VM, version 17.0.11+9 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://176c27c31d44:9080/
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShift started in 0.242 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.902 seconds.

#### Connect to the running container

You can also verify whether the container is running fine by connecting to it. This opens a command shell where you can issue supported unix commands.

| Command| Help |
| :--- | :--- |
| podman exec -ti NAMES /bin/bash | Connect to the running container using the bash shell |

````cmd
podman exec -ti liberty-to-openshift /bin/bash
````

    c:\ocp\LibertyToOpenShift>podman exec -ti liberty-to-openshift /bin/bash
    bash-4.4$ pwd
    /
    bash-4.4$ ls -l
    total 60
    lrwxrwxrwx   1 root    root      7 Jun 21  2021 bin -> usr/bin
    dr-xr-xr-x   2 root    root   4096 Jun 21  2021 boot
    lrwxrwxrwx   1 default root     37 Jun 24 04:38 config -> /opt/ol/wlp/usr/servers/defaultServer
    drwxr-xr-x   5 root    root    340 Jun 29 19:45 dev
    drwxr-xr-x   1 root    root   4096 Jun 24 04:38 etc
    lrwxrwxrwx   1 root    root     13 Jun 24 04:38 fixes -> /opt/ol/fixes
    drwxr-xr-x   2 root    root   4096 Jun 21  2021 home
    lrwxrwxrwx   1 root    root      7 Jun 21  2021 lib -> usr/lib
    lrwxrwxrwx   1 root    root      9 Jun 21  2021 lib64 -> usr/lib64
    lrwxrwxrwx   1 root    root     11 Jun 24 04:38 liberty -> /opt/ol/wlp
    lrwxrwxrwx   1 root    root     48 Jun 24 04:38 lib.index.cache -> /opt/ol/wlp/usr/shared/resources/lib.index.cache
    drwxr-xr-x   1 root    root   4096 Jun 24 04:37 licenses
    drwxrwxr-x   1 default root   4096 Jun 29 19:45 logs
    drwx------   2 root    root   4096 Jun  5 10:54 lost+found
    drwxr-xr-x   2 root    root   4096 Jun 21  2021 media
    drwxr-xr-x   2 root    root   4096 Jun 21  2021 mnt
    drwxr-xr-x   1 root    root   4096 Jun 24 04:37 opt
    lrwxrwxrwx   1 root    root     32 Jun 24 04:38 output -> /opt/ol/wlp/output/defaultServer
    dr-xr-xr-x 307 nobody  nobody    0 Jun 29 19:45 proc
    dr-xr-x---   3 root    root   4096 Jun  5 10:57 root
    drwxr-xr-x   1 root    root   4096 Jun 29 19:45 run
    lrwxrwxrwx   1 root    root      8 Jun 21  2021 sbin -> usr/sbin
    drwxr-xr-x   2 root    root   4096 Jun 21  2021 srv
    dr-xr-xr-x  11 nobody  nobody    0 Jun 29 19:45 sys
    drwxrwxrwt   1 root    root   4096 Jun 29 19:45 tmp
    drwxr-xr-x   1 root    root   4096 Jun  5 10:54 usr
    drwxr-xr-x   1 root    root   4096 Jun  5 10:54 var
    bash-4.4$ cd logs
    bash-4.4$ ls -l
    total 148
    -rw-r----- 1 default root   4259 Jun 29 19:45 messages.log
    -rw-r----- 1 default root 140031 Jun 29 19:49 verbosegc.001.log
    bash-4.4$ cat messages.log
    ********************************************************************************
    product = Open Liberty 24.0.0.6 (wlp-1.0.90.cl240620240603-2001)
    wlp.install.dir = /opt/ol/wlp/
    server.output.dir = /opt/ol/wlp/output/defaultServer/
    java.home = /opt/java/openjdk
    java.version = 17.0.11
    java.runtime = IBM Semeru Runtime Open Edition (17.0.11+9)
    os = Linux (5.15.153.1-microsoft-standard-WSL2; amd64) (en_US)
    process = 1@176c27c31d44
    Classpath = /opt/ol/wlp/bin/tools/ws-server.jar
    Java Library path = /opt/java/openjdk/lib/default:/opt/java/openjdk/lib:/usr/lib64:/usr/lib
    ********************************************************************************
    [6/29/24, 19:45:31:054 UTC] 00000001 com.ibm.ws.kernel.launch.internal.FrameworkManager           A CWWKE0001I: The server defaultServer has been launched.
    [6/29/24, 19:45:31:219 UTC] 00000036 com.ibm.ws.config.xml.internal.ServerXMLConfiguration        A CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [6/29/24, 19:45:31:230 UTC] 00000036 com.ibm.ws.config.xml.internal.ServerXMLConfiguration        A CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [6/29/24, 19:45:31:289 UTC] 00000001 com.ibm.ws.kernel.launch.internal.FrameworkManager           I CWWKE0002I: The kernel started after 0.359 seconds
    [6/29/24, 19:45:31:293 UTC] 0000003f com.ibm.ws.kernel.feature.internal.FeatureManager            I CWWKF0007I: Feature update started.
    [6/29/24, 19:45:31:369 UTC] 00000034 com.ibm.ws.app.manager.internal.monitor.DropinMonitor        A CWWKZ0058I: Monitoring dropins for applications.
    [6/29/24, 19:45:31:453 UTC] 00000045 com.ibm.ws.app.manager.AppMessageHelper                      I CWWKZ0018I: Starting application LibertyToOpenShift.
    [6/29/24, 19:45:31:557 UTC] 00000045 com.ibm.ws.session.WASSessionCore                            I SESN8501I: The session manager did not find a persistent storage location; HttpSession objects will be stored in the local application server's memory.
    [6/29/24, 19:45:31:568 UTC] 00000045 com.ibm.ws.webcontainer.osgi.webapp.WebGroup                 I SRVE0169I: Loading Web Module: LibertyToOpenShift.
    [6/29/24, 19:45:31:569 UTC] 00000045 com.ibm.ws.webcontainer                                      I SRVE0250I: Web Module LibertyToOpenShift has been bound to default_host.
    [6/29/24, 19:45:31:569 UTC] 00000045 com.ibm.ws.http.internal.VirtualHostImpl                     A CWWKT0016I: Web application available (default_host): http://176c27c31d44:9080/
    [6/29/24, 19:45:31:633 UTC] 00000049 com.ibm.ws.session.WASSessionCore                            I SESN0176I: A new session context will be created for application key default_host/
    [6/29/24, 19:45:31:641 UTC] 00000049 com.ibm.ws.util                                              I SESN0172I: The session manager is using the Java default SecureRandom implementation for session ID generation.
    [6/29/24, 19:45:31:696 UTC] 00000049 com.ibm.ws.app.manager.AppMessageHelper                      A CWWKZ0001I: Application LibertyToOpenShift started in 0.242 seconds.
    [6/29/24, 19:45:31:703 UTC] 0000003f com.ibm.ws.tcpchannel.internal.TCPPort                       I CWWKO0219I: TCP Channel defaultHttpEndpoint has been started and is now listening for requests on host *  (IPv6) port 9080.
    [6/29/24, 19:45:31:791 UTC] 0000004a com.ibm.ws.webcontainer.osgi.mbeans.PluginGenerator          I SRVE9103I: A configuration file for a web server plugin was automatically generated for this server at /opt/ol/wlp/output/defaultServer/logs/state/plugin-cfg.xml.
    [6/29/24, 19:45:31:831 UTC] 0000003f com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [6/29/24, 19:45:31:832 UTC] 0000003f com.ibm.ws.kernel.feature.internal.FeatureManager            I CWWKF0008I: Feature update completed in 0.542 seconds.
    [6/29/24, 19:45:31:832 UTC] 0000003f com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.902 seconds.
    bash-4.4$ cd /config/
    bash-4.4$ ls -l
    total 16
    drwxrwx--- 1 default root 4096 Jun 29 19:38 apps
    drwxrwxr-x 1 default root 4096 Jun 24 04:38 configDropins
    drwxrwx--- 1 default root 4096 Jun 24 04:37 dropins
    -rw-rw-rw- 1 default root  725 Jun 29 19:38 server.xml
    bash-4.4$ cat server.xml
    <server description="new server">

        <!-- Enable features -->
        <featureManager>
            <feature>jsp-2.3</feature>
            <feature>localConnector-1.0</feature>
            <feature>servlet-4.0</feature>
        </featureManager>

        <!-- To access this server from a remote client add a host attribute to the following element, e.g. host="*" -->
        <httpEndpoint httpPort="9080" httpsPort="9443" id="defaultHttpEndpoint"/>

        <!-- Automatically expand WAR files and EAR files -->
        <applicationManager autoExpand="true"/>

        <applicationMonitor updateTrigger="mbean"/>

        <webApplication id="LibertyToOpenShift" location="LibertyToOpenShift.war" name="LibertyToOpenShift" contextRoot="/"/>
    </server>bash-4.4$

#### Use curl to access the app

Test the application using curl.

````cmd
curl -vk http://localhost:9080
````

    c:\ocp\LibertyToOpenShift>curl -vk http://localhost:9080
    * Host localhost:9080 was resolved.
    * IPv6: ::1
    * IPv4: 127.0.0.1
    *   Trying [::1]:9080...
    * Connected to localhost (::1) port 9080
    > GET / HTTP/1.1
    > Host: localhost:9080
    > User-Agent: curl/8.7.1
    > Accept: */*
    >
    * Request completely sent off
    < HTTP/1.1 200 OK
    < X-Powered-By: Servlet/4.0
    < Content-Type: text/html;charset=UTF-8
    < Content-Language: en-US
    < Transfer-Encoding: chunked
    < Date: Sat, 29 Jun 2024 19:55:26 GMT
    <
    <html>
    <head><title>Servlet Settings</title></head>
    <body>
    <h1>LibertyToOpenShift Testing </h1>
    <table border='1'>
    <tr><th>Name</th><th>Value</th></tr>
    <tr>
    <td>Request URL</td>
    :

#### Browse the app using a browser

Test the application using a browser.

[http://localhost:9080](http://localhost:9080)

### Stop the running container

| Command| Help |
| :--- | :--- |
| podman ps | Display the running containers |
| podman stop CONTAINER_ID |  Stop the container CONTAINER_ID |
| podman ps -a | Display the running containers, including the stopped containers |
| podman rm CONTAINER_ID | Delete a container |

````cmd
podman ps
podman stop 176c27c31d44
podman ps
podman ps -a
podman rm 176c27c31d44
````

    c:\ocp\LibertyToOpenShift>podman ps
    CONTAINER ID  IMAGE                                          COMMAND               CREATED         STATUS         PORTS                   NAMES
    176c27c31d44  localhost/liberty-to-openshift:olp-java17-1.0  /opt/ol/wlp/bin/s...  12 minutes ago  Up 12 minutes  0.0.0.0:9080->9080/tcp  liberty-to-openshift

    c:\ocp\LibertyToOpenShift>podman stop 176c27c31d44
    176c27c31d44

    c:\ocp\LibertyToOpenShift>podman ps
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

    c:\ocp\LibertyToOpenShift>podman ps -a
    CONTAINER ID  IMAGE                                          COMMAND               CREATED         STATUS                      PORTS                   NAMES
    176c27c31d44  localhost/liberty-to-openshift:olp-java17-1.0  /opt/ol/wlp/bin/s...  12 minutes ago  Exited (143) 5 seconds ago  0.0.0.0:9080->9080/tcp  liberty-to-openshift

    c:\ocp\LibertyToOpenShift>podman rm 176c27c31d44
    176c27c31d44

    c:\ocp\LibertyToOpenShift>podman ps -a
    CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES

### Push the image to OpenShift

Push the image to the OpenShift internal repository.

#### Set OC CLI variables

Before proceeding, ensure you have set the OC CLI variables. You don't know how many times I missed this only to encounter frustrating problems.

````cmd
crc podman-env
@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
````

    c:\ocp\LibertyToOpenShift>crc podman-env
    SET PATH=C:\Users\bdsae\.crc\bin\oc;%PATH%
    SET CONTAINER_SSHKEY=C:\Users\bdsae\.crc\machines\crc\id_ecdsa
    SET CONTAINER_HOST=ssh://core@127.0.0.1:2222/run/user/1000/podman/podman.sock
    SET DOCKER_HOST=npipe:////./pipe/crc-podman
    REM Run this command to configure your shell:
    REM @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

    c:\ocp\LibertyToOpenShift>@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i

#### Login to OpenShift

Log into OpenShift using the developer user. The password is developer.

````cmd
oc login -u developer https://api.crc.testing:6443
````

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
oc project liberty-to-openshift
````
    c:\ocp\LibertyToOpenShift>oc projects
    You have access to the following projects and can switch between them with ' project <projectname>':

        guide
    * liberty-to-openshift
        liberty-to-openshift-instanton

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
oc new-project liberty-to-openshift
oc project
````

    c:\ocp\LibertyToOpenShift>oc new-project liberty-to-openshift
    Now using project "liberty-to-openshift" on server "https://api.crc.testing:6443".

    You can add applications to this project with the 'new-app' command. For example, try:

        oc new-app rails-postgresql-example

    to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

        kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname


    c:\ocp\LibertyToOpenShift>oc project
    Using project "liberty-to-openshift" on server "https://api.crc.testing:6443".

#### Tag the image

| Command| Help |
| :--- | :--- |
| oc registry info | Retrieve the OpenShift registry details where the image will be pushed |
| podman tag REPOSITORY:TAG OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Tag the image to the OpenShift registry |

Identify the registry where the image will be pushed.

````cmd
oc registry info
````

Tag the image pointing to the registry.

````cmd
podman tag liberty-to-openshift:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0
````

View the tagged image.

````cmd
podman images
````

    c:\ocp\LibertyToOpenShift>oc registry info
    default-route-openshift-image-registry.apps-crc.testing

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY                      TAG                            IMAGE ID      CREATED        SIZE
    localhost/liberty-to-openshift  olp-java17-1.0                 2ae4533e05f0  4 seconds ago  739 MB
    icr.io/appcafe/open-liberty     kernel-slim-java17-openj9-ubi  385d889567ef  3 months ago   688 MB

    c:\ocp\LibertyToOpenShift>podman tag liberty-to-openshift:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0

    c:\ocp\LibertyToOpenShift>podman images
    REPOSITORY                                                                                         TAG                            IMAGE ID      CREATED             SIZE
    localhost/liberty-to-openshift                                                                     olp-java17-1.0                 2ae4533e05f0  About a minute ago  739 MB
    default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift  olp-java17-1.0                 2ae4533e05f0  About a minute ago  739 MB
    icr.io/appcafe/open-liberty                                                                        kernel-slim-java17-openj9-ubi  385d889567ef  3 months ago        688 MB

#### Push the image to OpenShift

| Command| Help |
| :--- | :--- |
| podman push OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Push the image to the OpenShift internal image stream |

````cmd
podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0
````

    c:\ocp\LibertyToOpenShift>podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0
    Getting image source signatures
    Copying blob sha256:3fe103da66fa70dba2988aeee5622af89f139b9dc25f56264a23f265d6b963b9
    Copying blob sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91
    Copying blob sha256:a0395d30313e17d544847ae4cef4328527ee69acaa49ce9b392278cd7f993195
    Copying blob sha256:5072c9ed8ec7c16b80e415f16abf0b3e2790f56a9b2fd056a7f1fdc57d890484
    Copying blob sha256:38fbe8c222cd3c5bf55c0f4e58bbb09beca4c1b87586b40953d697995a05b608
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
    Copying blob sha256:b47758c75148c6515e4f5e2d5b62a465c9f5b35b9fa1514c82786b7088fdbdf9
    Copying blob sha256:478a283866ca3138ebe105c70f9c26979048091ee4ea2afcdc09c00e3074fd8f
    Copying blob sha256:bf7a3eb8f4fca73f7cffd9a19a39cfb57e677005df3f0ea346c416a995c400de
    Copying blob sha256:b6f3c684801a7a628f4bb2f10f01e752fb368e9b7fd24822b1ffa934e869ccb3
    Copying config sha256:2ae4533e05f0db92b5b0cc08d75a25ef28e812a2a04242b31c8b547d76ceebf5
    Writing manifest to image destination
    Storing signatures

Possible problems:

- **Error: trying to reuse blob** sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91 at destination: unable to retrieve auth token: invalid username/password: authentication required

    Issue the following to resolve the problem
    ````cmd
    crc podman-env
    @FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
    ````

- **Error: trying to reuse blob** sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91 at destination: unable to retrieve auth token: invalid username/password: authentication required

    Issue the following to resolve the problem

    ````cmd
    oc registry login --insecure=true
    ````

### Check the image in OpenShift

#### Find the pushed image

Either issue the following OC CLI command or log into OpenShift to verify the image has been pushed successfully.

````cmd
oc get imagestream
oc get is
````

    c:\ocp\LibertyToOpenShift>oc get imagestream
    NAME                   IMAGE REPOSITORY                                                                                    TAGS             UPDATED
    liberty-to-openshift   default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift   olp-java17-1.0   8 minutes ago

    c:\ocp\LibertyToOpenShift>oc get is
    NAME                   IMAGE REPOSITORY                                                                                    TAGS             UPDATED
    liberty-to-openshift   default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift   olp-java17-1.0   8 minutes ago

-- OR --

Login into the OpenShift console using the developer user.

- Select the appropriate project
- Switch from the Developer role to the Administrator role
- Expand Builds
- Select ImageStreams - you should see your image here

#### (Optionally) Pull the image from OpenShift to podman

| Command| Help |
| :--- | :--- |
| podman pull OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Pull the image from OpenShift to podman |

````cmd
podman pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0
````

    c:\ocp\LibertyToOpenShift>podman pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0
    Trying to pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0...
    Getting image source signatures
    Copying blob sha256:77590358b45b84caafe5ecd361bd9337ac3319cb0fcfaa581e1c7d3efdad75ce
    Copying blob sha256:19580bd631ab6ee8ed1f47416e094b849c1d0c1260c8fd2f1cde47c892669129
    Copying blob sha256:8f935604158f23174be2299afb9463d20940039de707c2466dcc0734bc152fa1
    Copying blob sha256:c24702af7a76136c090fc91245eb7ce805e53cc786f5810a57e3d9b65385dc5d
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
    Copying blob sha256:15b973b7e658692708aa4705857afc18bcf7bfcc3da6abc8b4731bc0fdc12640
    Copying blob sha256:f8ac83ba3f93f24cdbd7f5fb89c66a47b22f4896a407860077ad4427162c2094
    Copying blob sha256:afa615b17d2f5174b31afe1510f40c07186be05fece770df91180286691ba87a
    Copying blob sha256:a143249a9a4c6073e97e299990c5bf273e468b2f15bc00114babb2b06d6b91d7
    Copying blob sha256:abb9b12d387a4cff10f8d1fd59cc03d4795ac1ac74d60c62d21c5695efd9d46a
    Copying blob sha256:bbe57b4b798e7212ac1e08bee6b098131ef2412e4ab9284911cc64001f3da031
    Copying blob sha256:c6ff0bf50c8a79d8bc9f6c278e9b219db92befe6e69b0c1b480defb3aef81ae1
    Copying config sha256:2ae4533e05f0db92b5b0cc08d75a25ef28e812a2a04242b31c8b547d76ceebf5
    Writing manifest to image destination
    Storing signatures
    2ae4533e05f0db92b5b0cc08d75a25ef28e812a2a04242b31c8b547d76ceebf5

## Deploy an application

### Deploy an application to OpenShift using the pushed image

| Command| Help |
| :--- | :--- |
| oc apply -f file DEPLOY_YAML | Deploy the app using the contents of the DEPLOY_YAML file |

````cmd
cd /d c:\ocp\LibertyToOpenShift
oc apply -f deploy.yaml
````

    c:\ocp\LibertyToOpenShift>oc apply -f deploy.yaml
    deployment.apps/liberty-to-openshift created
    service/liberty-to-openshift created
    route.route.openshift.io/liberty-to-openshift created

### Check whether the pod is running

````cmd
oc get pods
oc get po
````

    c:\ocp\LibertyToOpenShift>oc get po
    NAME                                    READY   STATUS    RESTARTS   AGE
    liberty-to-openshift-77c96d4887-dmkzw   1/1     Running   0          54s

### Check the logs

| Command| Help |
| :--- | :--- |
| oc logs NAME | Display the logs for the NAME from the "oc get po" command |

````cmd
oc logs liberty-to-openshift-77c96d4887-dmkzw
````

    c:\ocp\LibertyToOpenShift>oc logs liberty-to-openshift-77c96d4887-dmkzw

    Launching defaultServer (Open Liberty 24.0.0.2/wlp-1.0.86.cl240220240212-1928) on Eclipse OpenJ9 VM, version 17.0.10+7 (en_US)
    [AUDIT   ] CWWKE0001I: The server defaultServer has been launched.
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [AUDIT   ] CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [AUDIT   ] CWWKZ0058I: Monitoring dropins for applications.
    [AUDIT   ] CWWKT0016I: Web application available (default_host): http://liberty-to-openshift-77c96d4887-dmkzw:9080/
    [AUDIT   ] CWWKZ0001I: Application LibertyToOpenShift started in 0.221 seconds.
    [AUDIT   ] CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [AUDIT   ] CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.676 seconds.

| Command| Help |
| :--- | :--- |
| oc rsh NAME | Connect to the running container. This is similar to the podman "podman exec -ti" command. |

````cmd
oc rsh liberty-to-openshift-77c96d4887-dmkzw
````

    c:\ocp\LibertyToOpenShift>oc rsh liberty-to-openshift-77c96d4887-dmkzw
    sh-4.4$ pwd
    /
    sh-4.4$ cd logs
    sh-4.4$ ls -l
    total 8
    -rw-r-----. 1 1000680000 root 4284 Jul  1 18:28 messages.log
    sh-4.4$ cat messages.log
    ********************************************************************************
    product = Open Liberty 24.0.0.2 (wlp-1.0.86.cl240220240212-1928)
    wlp.install.dir = /opt/ol/wlp/
    server.output.dir = /opt/ol/wlp/output/defaultServer/
    java.home = /opt/java/openjdk
    java.version = 17.0.10
    java.runtime = IBM Semeru Runtime Open Edition (17.0.10+7)
    os = Linux (5.14.0-284.52.1.el9_2.x86_64; amd64) (en_US)
    process = 1@liberty-to-openshift-77c96d4887-dmkzw
    Classpath = /opt/ol/wlp/bin/tools/ws-server.jar
    Java Library path = /opt/java/openjdk/lib/default:/opt/java/openjdk/lib:/usr/lib64:/usr/lib
    ********************************************************************************
    [7/1/24, 18:28:16:288 UTC] 00000001 com.ibm.ws.kernel.launch.internal.FrameworkManager           A CWWKE0001I: The server defaultServer has been launched.
    [7/1/24, 18:28:16:400 UTC] 00000027 com.ibm.ws.config.xml.internal.ServerXMLConfiguration        A CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/keystore.xml
    [7/1/24, 18:28:16:405 UTC] 00000027 com.ibm.ws.config.xml.internal.ServerXMLConfiguration        A CWWKG0093A: Processing configuration drop-ins resource: /opt/ol/wlp/usr/servers/defaultServer/configDropins/defaults/open-default-port.xml
    [7/1/24, 18:28:16:448 UTC] 00000001 com.ibm.ws.kernel.launch.internal.FrameworkManager           I CWWKE0002I: The kernel started after 0.238 seconds
    [7/1/24, 18:28:16:449 UTC] 00000030 com.ibm.ws.kernel.feature.internal.FeatureManager            I CWWKF0007I: Feature update started.
    [7/1/24, 18:28:16:499 UTC] 00000026 com.ibm.ws.app.manager.internal.monitor.DropinMonitor        A CWWKZ0058I: Monitoring dropins for applications.
    [7/1/24, 18:28:16:551 UTC] 00000036 com.ibm.ws.app.manager.AppMessageHelper                      I CWWKZ0018I: Starting application LibertyToOpenShift.
    [7/1/24, 18:28:16:619 UTC] 00000036 com.ibm.ws.session.WASSessionCore                            I SESN8501I: The session manager did not find a persistent storage location; HttpSession objects will be stored in the local application server's memory.
    [7/1/24, 18:28:16:625 UTC] 00000036 com.ibm.ws.webcontainer.osgi.webapp.WebGroup                 I SRVE0169I: Loading Web Module: LibertyToOpenShift.
    [7/1/24, 18:28:16:626 UTC] 00000036 com.ibm.ws.webcontainer                                      I SRVE0250I: Web Module LibertyToOpenShift has been bound to default_host.
    [7/1/24, 18:28:16:626 UTC] 00000036 com.ibm.ws.http.internal.VirtualHostImpl                     A CWWKT0016I: Web application available (default_host): http://liberty-to-openshift-77c96d4887-dmkzw:9080/
    [7/1/24, 18:28:16:719 UTC] 0000003a com.ibm.ws.session.WASSessionCore                            I SESN0176I: A new session context will be created for application key default_host/
    [7/1/24, 18:28:16:733 UTC] 0000003a com.ibm.ws.util                                              I SESN0172I: The session manager is using the Java default SecureRandom implementation for session ID generation.
    [7/1/24, 18:28:16:773 UTC] 0000003a com.ibm.ws.app.manager.AppMessageHelper                      A CWWKZ0001I: Application LibertyToOpenShift started in 0.221 seconds.
    [7/1/24, 18:28:16:778 UTC] 00000030 com.ibm.ws.tcpchannel.internal.TCPPort                       I CWWKO0219I: TCP Channel defaultHttpEndpoint has been started and is now listening for requests on host *  (IPv6) port 9080.
    [7/1/24, 18:28:16:841 UTC] 0000003b com.ibm.ws.webcontainer.osgi.mbeans.PluginGenerator          I SRVE9103I: A configuration file for a web server plugin was automatically generated for this server at /opt/ol/wlp/output/defaultServer/logs/state/plugin-cfg.xml.
    [7/1/24, 18:28:16:885 UTC] 00000030 com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0012I: The server installed the following features: [el-3.0, jsp-2.3, localConnector-1.0, servlet-4.0].
    [7/1/24, 18:28:16:885 UTC] 00000030 com.ibm.ws.kernel.feature.internal.FeatureManager            I CWWKF0008I: Feature update completed in 0.437 seconds.
    [7/1/24, 18:28:16:885 UTC] 00000030 com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 0.676 seconds.
    sh-4.4$ exit
    exit

    c:\ocp\LibertyToOpenShift>

### Use curl to access the app - wrong way

If you try the same URL as used with podman, you'll see the below error.

````cmd
curl -vk http://localhost:9080
````

    c:\ocp\LibertyToOpenShift>curl -vk http://localhost:9080
    * Host localhost:9080 was resolved.
    * IPv6: ::1
    * IPv4: 127.0.0.1
    *   Trying [::1]:9080...
    *   Trying 127.0.0.1:9080...
    * connect to ::1 port 9080 from :: port 65004 failed: Connection refused
    * connect to 127.0.0.1 port 9080 from 0.0.0.0 port 65005 failed: Connection refused
    * Failed to connect to localhost port 9080 after 2247 ms: Couldn't connect to server
    * Closing connection
    curl: (7) Failed to connect to localhost port 9080 after 2247 ms: Couldn't connect to server

### Use curl to access the app - correct way

For OpenShift you need to access the URL using the OpenShift route. 

The command "oc apply -f deploy.yaml" creates the OpenShift route and service. These are used to access the application.

Find the route to the running pod.

````cmd
oc get routes
````

c:\ocp\LibertyToOpenShift>oc get routes
NAME                   HOST/PORT                                                    PATH   SERVICES               PORT    TERMINATION   WILDCARD
liberty-to-openshift   liberty-to-openshift-liberty-to-openshift.apps-crc.testing          liberty-to-openshift   <all>                 None

Now curl to the route.

````cmd
curl -vk http://liberty-to-openshift-liberty-to-openshift.apps-crc.testing
````

    c:\ocp\LibertyToOpenShift>curl -vk http://liberty-to-openshift-liberty-to-openshift.apps-crc.testing
    * Host liberty-to-openshift-liberty-to-openshift.apps-crc.testing:80 was resolved.
    * IPv6: (none)
    * IPv4: 127.0.0.1
    *   Trying 127.0.0.1:80...
    * Connected to liberty-to-openshift-liberty-to-openshift.apps-crc.testing (127.0.0.1) port 80
    > GET / HTTP/1.1
    > Host: liberty-to-openshift-liberty-to-openshift.apps-crc.testing
    > User-Agent: curl/8.7.1
    > Accept: */*
    >
    < HTTP/1.1 200 OK
    < x-powered-by: Servlet/4.0
    < content-type: text/html;charset=UTF-8
    < content-language: en-US
    < transfer-encoding: chunked
    < date: Mon, 01 Jul 2024 18:41:10 GMT
    < set-cookie: 1ccdf9b7cb530525017c6e7399d962a9=1f0aa231747917d5f0076a5de12198c4; path=/; HttpOnly
    <
    <html>
    <head><title>Servlet Settings</title></head>
    <body>
    <h1>LibertyToOpenShift Testing </h1>
    <table border='1'>
    <tr><th>Name</th><th>Value</th></tr>
    <tr>
    <td>Request URL</td>
    <td>http://liberty-to-openshift-liberty-to-openshift.apps-crc.testing/</td>
    :

### Browse the app using a browser

Open a browser of your choice and paste the below URL.

[http://liberty-to-openshift-liberty-to-openshift.apps-crc.testing](http://liberty-to-openshift-liberty-to-openshift.apps-crc.testing)

or if you have enables HTTPS from a later step, use

[https://liberty-to-openshift-liberty-to-openshift.apps-crc.testing](https://liberty-to-openshift-liberty-to-openshift.apps-crc.testing)

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
oc describe route liberty-to-openshift
````

    c:\ocp\LibertyToOpenShift>oc get routes
    NAME                   HOST/PORT                                                    PATH   SERVICES               PORT   TERMINATION   WILDCARD
    liberty-to-openshift   liberty-to-openshift-liberty-to-openshift.apps-crc.testing          liberty-to-openshift   9080   edge          None

    c:\ocp\LibertyToOpenShift>oc describe route liberty-to-openshift
    Name:                   liberty-to-openshift
    Namespace:              liberty-to-openshift
    Created:                33 minutes ago
    Labels:                 <none>
    Annotations:            kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"route.openshift.io/v1","kind":"Route","metadata":{"annotations":{},"name":"liberty-to-openshift","namespace":"liberty-to-openshift"},"spec":{"to":{"kind":"Service","name":"liberty-to-openshift"}}}

                            openshift.io/host.generated=true
    Requested Host:         liberty-to-openshift-liberty-to-openshift.apps-crc.testing
                            exposed on router default (host router-default.apps-crc.testing) 33 minutes ago
    Path:                   <none>
    TLS Termination:        edge
    Insecure Policy:        <none>
    Endpoint Port:          9080

    Service:        liberty-to-openshift
    Weight:         100 (100%)
    Endpoints:      10.217.0.109:9080

````cmd
oc get services
oc describe service liberty-to-openshift
````

    c:\ocp\LibertyToOpenShift>oc get services
    NAME                   TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    liberty-to-openshift   ClusterIP   10.217.4.60   <none>        9080/TCP   33m

    c:\ocp\LibertyToOpenShift>oc describe service liberty-to-openshift
    Name:              liberty-to-openshift
    Namespace:         liberty-to-openshift
    Labels:            <none>
    Annotations:       <none>
    Selector:          app=liberty-to-openshift
    Type:              ClusterIP
    IP Family Policy:  SingleStack
    IP Families:       IPv4
    IP:                10.217.4.60
    IPs:               10.217.4.60
    Port:              <unset>  9080/TCP
    TargetPort:        9080/TCP
    Endpoints:         10.217.0.109:9080
    Session Affinity:  None
    Events:            <none>

````cmd
oc get po
oc describe po liberty-to-openshift-77c96d4887-dmkzw
````

    c:\ocp\LibertyToOpenShift>oc get po
    NAME                                    READY   STATUS    RESTARTS   AGE
    liberty-to-openshift-77c96d4887-dmkzw   1/1     Running   0          35m

    c:\ocp\LibertyToOpenShift>oc describe po liberty-to-openshift-77c96d4887-dmkzw
    Name:             liberty-to-openshift-77c96d4887-dmkzw
    Namespace:        liberty-to-openshift
    Priority:         0
    Service Account:  default
    Node:             crc-vlf7c-master-0/192.168.126.11
    Start Time:       Mon, 01 Jul 2024 21:28:13 +0300
    Labels:           app=liberty-to-openshift
                    pod-template-hash=77c96d4887
    Annotations:      k8s.v1.cni.cncf.io/network-status:
                        [{
                            "name": "openshift-sdn",
                            "interface": "eth0",
                            "ips": [
                                "10.217.0.109"
                            ],
                            "default": true,
                            "dns": {}
                        }]
                    openshift.io/scc: restricted-v2
                    seccomp.security.alpha.kubernetes.io/pod: runtime/default
    Status:           Running
    SeccompProfile:   RuntimeDefault
    IP:               10.217.0.109
    IPs:
    IP:           10.217.0.109
    Controlled By:  ReplicaSet/liberty-to-openshift-77c96d4887
    Containers:
    liberty-to-openshift:
        Container ID:   cri-o://3ba5e136a5070881c2d4d60226bd55bc03d16dcb177314d5e72b0d207845684d
        Image:          image-registry.openshift-image-registry.svc:5000/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0
        Image ID:       image-registry.openshift-image-registry.svc:5000/liberty-to-openshift/liberty-to-openshift@sha256:3ca7b7c8660ba97595639c5067aafda1a346088836588107c897ffd04792df6d
        Port:           9080/TCP
        Host Port:      0/TCP
        State:          Running
        Started:      Mon, 01 Jul 2024 21:28:16 +0300
        Ready:          True
        Restart Count:  0
        Environment:    <none>
        Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tdvbj (ro)
    Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
    Volumes:
    kube-api-access-tdvbj:
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
    Normal  Scheduled       35m   default-scheduler  Successfully assigned liberty-to-openshift/liberty-to-openshift-77c96d4887-dmkzw to crc-vlf7c-master-0
    Normal  AddedInterface  35m   multus             Add eth0 [10.217.0.109/23] from openshift-sdn
    Normal  Pulling         35m   kubelet            Pulling image "image-registry.openshift-image-registry.svc:5000/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0"
    Normal  Pulled          35m   kubelet            Successfully pulled image "image-registry.openshift-image-registry.svc:5000/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0" in 461.834706ms (461.850303ms including waiting)
    Normal  Created         35m   kubelet            Created container liberty-to-openshift
    Normal  Started         35m   kubelet            Started container liberty-to-openshift
    
## Tear down the environment

To delete the environment, issue the following commands.

### Select the project

````cmd
oc projects
oc project liberty-to-openshift
````

### Delete the deployment including the service and route

````cmd
cd /d c:\ocp\LibertyToOpenShift
oc delete -f deploy.yaml
````

### Delete the pushed image

````cmd
oc delete imagestream/liberty-to-openshift
````

### Delete the project

````cmd
oc delete project liberty-to-openshift
oc projects
````

## Inspiration

I guess my main inspiration is a constant desire to learn new technologies and to stay relevant in this ever changing world.

Coming from a WebSphere Application Server/WebSphere Liberty admin background, where my passion is programming, I followed the IBM [Open Liberty](https://openliberty.io/guides/) guides to get started. Specifically, I settled on the following learning path. I highly recommend you do the same - no need to install any software, the guides can be completed online.

| Topic | URL |
| :---  | :--- |
| Creating a RESTful web service | [https://openliberty.io/guides/rest-intro.html](https://openliberty.io/guides/rest-intro.html) |
| Consuming a RESTful web service with AngularJS | [https://openliberty.io/guides/rest-client-angularjs.html](https://openliberty.io/guides/rest-client-angularjs.html) |
| Using Docker containers to develop microservices | [https://openliberty.io/guides/docker.html](https://openliberty.io/guides/docker.html) |
| Containerizing microservices | [https://openliberty.io/guides/containerize.html](https://openliberty.io/guides/containerize.html) |
| Deploying microservices to Kubernetes (1) | [https://openliberty.io/guides/kubernetes-intro.html](https://openliberty.io/guides/kubernetes-intro.html) |
| Deploying microservices to an OpenShift cluster using OpenShift Local | [https://openliberty.io/guides/openshift-codeready-containers.html](https://openliberty.io/guides/openshift-codeready-containers.html) |
| Deploying microservices to OpenShift 4 using Kubernetes Operators | [https://openliberty.io/guides/cloud-openshift-operator.html](https://openliberty.io/guides/cloud-openshift-operator.html) |

(1) I faced some problems issuing kubectl on my PC. I'm not sure why, but I skipped to the OpenShift Local guides as that was my primary focus.

