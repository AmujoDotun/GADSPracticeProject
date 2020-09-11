## AK8S-04 Creating a GKE Cluster via Cloud Shell

## Overview

In this lab, I make used of the command line to build GKE clusters. I inspected the kubeconfig file, and I used kubectl to manipulate the cluster.

## Objectives

1. In this lab, I learnt how to perform the following tasks:

2. Use kubectl to build and manipulate GKE clusters

3. Use kubectl and configuration files to deploy Pods

4. Use Container Registry to store and deploy containers


# Let me take you on a ride on how I did all this

# Steps: Open Cloud Shell

To check the active accconts make use of this command

gcloud auth list


# Task 1. Deploy GKE clusters

1. Set the environment variable for the zone and cluster name using this command

    export my_zone=us-central1-a
    export my_cluster=standard-cluster-1

2. Create a Kubernetes cluster with this command

    gcloud container clusters create $my_cluster --num-nodes 3 --zone $my_zone --enable-ip-alias

# Task 2. Modify GKE clusters

1. Execute the following command to modify standard-cluster-1 to have four nodes:

    gcloud container clusters resize $my_cluster --zone $my_zone --size=4

2. when you run the command above, you will get a prompt, your answer to it is yes(y)
    

# Task 3. Connect to a GKE cluster

1. To create a kubeconfig file with the credentials of the current user (to allow authentication) and provide the endpoint details for a specific cluster (to allow communicating with that cluster through the kubectl command-line tool), execute the following command

    gcloud container clusters get-credentials $my_cluster --zone $my_zone

2. Open the kubeconfig file with the nano text editor

        nano ~/.kube/config

3. The command above is for you to examine the content of the file, press Ctrl + X to exit when done 

# Task 4. Use kubectl to inspect a GKE cluster

1. In order to print the content of the file, run the following command

        kubectl config view

2. Print out the cluster information for the active context

    kubectl cluster-info

    YOU SHOULD HAVE AN OUTPUT LIKE THIS BELOW(DON'T COPY)

        Kubernetes master is running at https://104.155.191.14
        GLBCDefaultBackend is running at https://104.155.191.14/api/v1/namespaces/kube-system/services/default-http-backend:http/proxy
        Heapster is running at https://104.155.191.14/api/v1/namespaces/kube-system/services/heapster/proxy
        KubeDNS is running at https://104.155.191.14/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
        Metrics-server is running at https://104.155.191.14/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
        To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

3. in Cloud Shell, execute the following command to print out the active context

    kubectl config current-context

    OUTPUT
        gke_[PROJECT_ID]_us-central1-a_standard-cluster-1

         PROJECT_ID is your project ID. This information is the same as the information in the current-context property of the kubeconfig file.

4. In Cloud Shell, execute the following command to print out some details for all the cluster contexts in the kubeconfig file

    kubectl config get-contexts

5. Execute the following command to change the active context

    kubectl config use-context gke_${GOOGLE_CLOUD_PROJECT}_us-central1-a_standard-cluster-1

6. Execute the following command to view the resource usage across the nodes of the cluster

    kubectl top nodes

    Output (do not copy)
        NAME                            CPU(cores)   CPU%  MEMORY(bytes)  MEMORY%
        gke-standard-cluster-1-def...   29m          3%    431Mi          16%
        gke-standard-cluster-1-def...   45m          4%    605Mi          22%
        gke-standard-cluster-1-def...   40m          4%    559Mi          21%
        gke-standard-cluster-1-def...   34m          3%    488Mi          18%

7. Execute the following command to enable bash autocompletion for kubectl

    source <(kubectl completion bash)

    
8. Type kubectl and press the Tab key twice

        The shell outputs all the possible commands:


# Task 5. Deploy Pods to GKE clusters

1. Execute the following command to deploy the latest version of nginx as a Pod named nginx-1

    kubectl create deployment nginx-1 --image nginx:latest

2. Execute the following command to view all the deployed Pods in the active context cluster

    kubectl get pods

    LIKELY OUTPUT(DON'T COPY)

        NAME                       READY     STATUS    RESTARTS   AGE
        nginx-1-74c7bbdb84-nvwsc   1/1       Running   0          9s

3. You will now enter your Pod name into a variable that we will use throughout this lab. Using variables like this can help you minimize human error when typing long names. You must type your Pod's unique name in place of [your_pod_name].

    export my_nginx_pod=[your_pod_name]

    Example (do not copy)
        export my_nginx_pod=nginx-1-74c7bbdb84-nvwsc

4. Confirm that you have set the environment variable successfully by having the shell echo the value back to you

    echo $my_nginx_pod

5. Execute the following command to view the complete details of the Pod you just created

    kubectl describe pod $my_nginx_pod

# Push a file into a container

1. In Cloud Shell, type the following commands to open a file named test.html in the nano text editor

    nano ~/test.html

2. Add the following text (shell script) to the empty test.html file

    <html> <header><title>This is title</title></header>
    <body> Hello world </body>
    </html>

3. Press CTRL+X, then press Y and enter to save the file and exit the nano editor

4. Execute the following command to place the file into the appropriate location within the nginx container in the nginx Pod to be served statically

        kubectl cp ~/test.html $my_nginx_pod:/usr/share/nginx/html/test.html

# Expose the Pod for testing

1. Execute the following command to create a service to expose our nginx Pod externally

    kubectl expose pod $my_nginx_pod --port 80 --type LoadBalancer

2. Execute the following command to view details about services in the cluster

    kubectl get services

3. xecute the following command to verify that the nginx container is serving the static HTML file that you copied. You replace [EXTERNAL_IP] with the external IP address of your service that you obtained from the output of the previous step.

        curl http://[EXTERNAL_IP]/test.html

4. Eecute the following command to view the resources being used by the nginx Pod

    kubectl top pods

# Task 6. Introspect GKE Pods

1. enter the following command to clone the repository to the lab Cloud Shell.

        git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst

2. Change to the directory that contains the sample files for this lab.

        cd ~/training-data-analyst/courses/ak8s/04_GKE_Shell/

3. To deploy your manifest, execute the following command:

        kubectl apply -f ./new-nginx-pod.yaml

4. To see a list of Pods, execute the following command:

        kubectl get pods
5. Execute the following command to start an interactive bash shell in the nginx container:

        kubectl exec -it new-nginx /bin/bash


6. in the nginx bash shell, execute the following commands to install the nano text editor:

    apt-get update
    apt-get install nano

7.  in the nginx bash shell, execute the following commands to switch to the static files directory and create a test.html file:

        cd /usr/share/nginx/html
        nano test.html

8. in the nginx bash shell nano session, type the following text:

        <html> <header><title>This is title</title></header>
        <body> Hello world </body>
        </html>

9. Press CTRL+0, then press  enter to save the file and exit the nano editor

10. in the nginx bash shell, execute the following command to exit the nginx bash shell:

        exit

12. click the plus sign (+) icon to start a new Cloud Shell session.

13. Execute the following command to test the modified nginx container through the port forwarding:

        curl http://127.0.0.1:10081/test.html


14. click the plus sign (+) icon to start another new Cloud Shell session.
A third Cloud Shell session appears in your Cloud Shell window. As before, you can switch sessions by clicking them in the menu bar.


15. Execute the following command to display the logs and to stream new logs as they arrive (and also include timestamps) for the new-nginx Pod:

    kubectl logs new-nginx -f --timestamps

16. You will see the logs display in this new window


17. Return to the second Cloud Shell window and re-run the curl command to generate some traffic on the Pod.

18. Close the third Cloud Shell window to stop displaying the log messages.

19. Close the original Cloud Shell window to stop the port forwarding process.