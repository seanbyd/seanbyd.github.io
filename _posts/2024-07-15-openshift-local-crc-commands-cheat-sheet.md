# OpenShift Local Commands

This post contains a list of OpenShift Local commands I've used.

| Command| Help |
| :--- | :--- |
| crc start | Start OpenShift Local |
| crc podman-env | Display the OpenShift Local variables that need to be set to use the OC CLI |
| crc stop | Stop OpenShift Local |
| @FOR /f "tokens=*" %i IN ('crc oc-env') DO @call %i | Set the OpenShift Local OC CLI variables |
| oc login -u developer https://api.crc.testing:6443 | Log into OpenShift Local using the developer account |
| https://console-openshift-console.apps-crc.testing | Access the OpenShift Local console |



