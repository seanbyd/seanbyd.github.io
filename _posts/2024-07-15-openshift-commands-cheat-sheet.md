# OpenShift Local Commands

This post contains a list of OpenShift Local commands I've used.

| Command | Help | Examples |
| :--- | :--- | :--- |
**PROJECT COMMANDS**
| oc projects | List all projects you have access to, including the project you are connected to | oc projects |
| oc project | Display the project you are connected to | oc project |
| oc project PROJECT_NAME | Select the project named PROJECT_NAME | oc project liberty-to-openshift |
| oc new-project NEW_PROJECT_NAME | Create a new project named NEW_PROJECT_NAME | oc new-project liberty-to-openshift |
**POD/IMAGE COMMANDS**
| oc registry info | Retrieve the OpneShift registry details where the image will be pushed | oc registry info |
| oc get imagestream<br>oc get is | List the OpenShift images | oc get imagestream |
| oc apply -f file DEPLOY_YAML | Deploy the app using the contents of the DEPLOY_YAML file | oc apply -f deploy.yaml |
| oc get pods<br>oc get po | Display the running pods in the selected OpenShift project | oc get pods |
| oc describe po POD_NAME | Display the details of the running pod named POD_NAME | oc describe po liberty-to-openshift-77c96d4887-dmkzw |
| oc logs NAME | Display the logs for the NAME from the "oc get po" command | oc logs liberty-to-openshift-77c96d4887-dmkzw |
| oc rsh NAME | Connect to the running container. This is similar to the podman "podman exec -ti" command. | oc rsh liberty-to-openshift-77c96d4887-dmkzw |
| oc get routes | Display the OpenShift routes for the selected project | oc get routes |
| oc describe route ROUTE_NAME | Display the details of the route named ROUTE_NAME | oc describe route liberty-to-openshift |
| oc get services | List the OpenShift services | oc get services |
| oc describe service SERVICE_NAME | Display the details of the service named SERVICE_NAME | oc describe service liberty-to-openshift |
**TEAR DOWN PROJECT**
| oc delete -f deploy.yaml | Delete the deployed OpenShift application | oc delete -f deploy.yaml |
| oc delete imagestream/IMAGE_NAME | Delete the OpenShift image | oc delete imagestream/liberty-to-openshift |
| oc delete project PROJECT_NAME | Delete the OpenShift project | oc delete project liberty-to-openshift |









