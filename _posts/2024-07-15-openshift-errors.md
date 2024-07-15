# Possible OpenShift errors with solutions

This post contains errors I have encountered while using OpenShift and podman.

## trying to reuse blob** sha256:ecf6a89969f55913ddb3946ec16ae6f081ea6da1bbbdd9405acc637c25409b91 at destination: unable to retrieve auth token: invalid username/password: authentication required

This error could be caused by a number of reasons.

### The OC CLI environment variables have not been set.

I have encountered this error on more than one occasion, especially when opening multiple command windows.

To overcome this error, try issuing the below command then re-trying.

````cmd
crc podman-env
@FOR /f "tokens=*" %i IN ('crc podman-env') DO @call %i
````

### You have not logged into the registry where the images will be pushed.

Another reason the error may appear is that you are not logged into the registry where the images will be pushed. If you are using OpenShift Local, try issuing the below command and then re-trying.

````cmd
oc registry login --insecure=true
````
