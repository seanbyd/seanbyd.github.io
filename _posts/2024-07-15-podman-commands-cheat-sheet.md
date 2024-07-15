# Podman commands

This post contains a list of podman commands I've used.

| Command| Help | Example |
| :--- | :--- | :--- |
| podman images | Display all the podman images  | podman images |
| podman rmi IMAGE_ID | Delete the podman image with image id IMAGE_ID | podman rmi 0e1aa95417a0 |
| podman rmi REPOSITORY:TAG | Delete the podman image matching the combination of REPOSITORY name and TAG | podman rmi icr.io/appcafe/open-liberty:kernel-slim-java17-openj9-ubi |
| podman rmi IMAGE_ID --force | This deletes all images including any tagged images | podman rmi 11e9b7915530 --force |
| podman build -t IMAGE_NAME -f CONTAINER_FILE or DOCKER_FILE | Build a podman image | podman build -t liberty-to-openshift:olp-java17-1.0 -f Containerfile.olp.slim.java17 |
| podman run -d --name A_NAME_OF_YOUR_CHOICE -p PORT_YOU_WILL_ACCESS:INTERNAL_PORT_IN_SERVER_XML REPOSITORY:TAG | Start a container | podman run -d --name liberty-to-openshift -p 9080:9080 liberty-to-openshift:olp-java17-1.0 |
| podman logs NAME | Display the container logs | podman logs liberty-to-openshift | 
| podman exec -ti NAMES /bin/bash | Connect to the running container using the bash shell | podman exec -ti liberty-to-openshift /bin/bash |
| podman ps | Display the running containers | podman ps |
| podman stop CONTAINER_ID |  Stop the container CONTAINER_ID | podman stop 176c27c31d44 |
| podman ps -a | Display the running containers, including the stopped containers | podman ps -a |
| podman rm CONTAINER_ID | Delete a container | podman rm 176c27c31d44 |
| podman tag REPOSITORY:TAG OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Tag the image to the OpenShift registry | podman tag liberty-to-openshift:olp-java17-1.0 default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0 |
| podman push OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Push the image to the OpenShift internal image stream | podman push default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0 |
| podman pull OC_REGISTRY_INFO/PROJECT_NAME/REPOSITORY:TAG | Pull the image from OpenShift to podman | podman pull default-route-openshift-image-registry.apps-crc.testing/liberty-to-openshift/liberty-to-openshift:olp-java17-1.0 |




