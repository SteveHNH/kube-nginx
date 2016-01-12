# Kubernetes NGINX Demo

***This demo assumes a running kubernetes cluster exists.***

I wanted to test out nginx running with an NFS mounted space for the files as a persistent volume. I didn't work with PVs and PVCs on this one, rather deciding to just mount the NFS shares directly into the containers for the testing.

This is a super simple config, but it helped me get a handle on some of the concepts regarding replication controllers, persistent storage, and rolling-updates.

## Setup
Get an nfs share running. You can modify the directory within the yml files as well as the server. Feel free to change up the host port as well if you want. I just put an index.html file with some stuff in it in the NFS share to show that the website changes when the updates roll.

## Using kubernetes to build the replication controller

One of the main goals was to have constant uptime with the two webservers. Replication Controllers via kubernetes can help do that as it maintains the state. In other words, "These containers should always look like this." It can be thought of as a maintained state rather than a set of instructions.

    kubectl create -f rc-nginx.yml

You can see the pods that started by running:

    kubectl get pods

You should see "nginx-controller-randomchars" twice. Since our replication controller says to always run 2 containers of nginx

## Initiate rolling updates

The rolling updates will take down one container at a time, replacing it with the new replication controller when it finishes. This command replaces one runnign container every 10 seconds

    kubectl rolling-update nginx-controller --update-period=10s -f rc-nginx-v2.yml

If you have to rollback the update, it's equally as simple just by reversing it.

    kubectl rolling-update nginx-controller-v2 --update-period=10s -f rc-nginx.yml

If you have a kubernetes service or load balancer configured, you can actually see the website refresh. Make sure different content is in the different shares you used so that you can actually see the content change.

## Service

Among the files here is also a kubernetes service yaml file for standing up a the NGINX install as a service. You can point a load balancer or proxy at any node in the cluster and the service will load balance the connections across all pods matching the selector. In our case, this is app: nginx-v1.

    kubectl create -f svc-nginx.yml

Once the service is up, you can point your browser to ${EXTERNAL_IP}:30061. The nodePort value was configured to 30061 but it could just as easily be any other port between 30000 and 32767.

*NOTE* If you use this option, be sure to remove hostPort from the container template in rc-nginx.yml.

## Optional: Bundle the app

You can also bundle the service and replication controller together into a single file so it stands everything up without having to reference two files. The syntax for that is found in the app-nginx.yml file in this repo.

    kubectl create -f app-nginx.yml

## Ideas

My next steps are to use Git to manage the web content, then try to get jenkins to pull it automatically and build the new servers with one simple job. Also want to learn how to get drupal/wordpress and mariadb working together with static content.
