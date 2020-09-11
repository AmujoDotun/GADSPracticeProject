## AK8S-09 Configuring Kubernetes Engine Networking

## Overview

In this lab, I created a private cluster, add an authorized network for API access to it, and then configure a network policy for Pod security.

## Objectives
In this lab, I learnt how to perform the following tasks:

1. Create and test a private cluster

2. Configure a cluster for authorized network master access

3. Configure a Cluster network policy.


# Task 1. Create a private cluster

1. Set up a private cluster

    gcloud container clusters create private-cluster-1 \
    --zone us-central1-c \
    --enable-master-authorized-networks \
    --network my-net-0 \
    --subnetwork my-subnet-2 \
    --cluster-secondary-range-name my-pods-2 \
    --services-secondary-range-name my-services-2 \
    --enable-private-nodes \
    --enable-ip-alias \
    --master-ipv4-cidr 172.16.0.0/28 \
    --no-enable-basic-auth \
    --no-issue-client-certificate

# Inspect your cluster

1. enter the following command to review the details of your new cluster

        gcloud container clusters describe private-cluster --region us-central1-a

# Task 2. Add an authorized network for cluster master access


# Task 3. Create a cluster network policy

1. Type the following command to set the environment variable for the zone and cluster name.
    export my_zone=us-central1-a
    export my_cluster=standard-cluster-1

2. Configure kubectl tab completion in Cloud Shell

        source <(kubectl completion bash)
    
3. Type the following command to create a Kubernetes cluster. Note that this command adds the additional flag --enable-network-policy to the parameters you have used in previous labs. This flag allows this cluster to use cluster network policies

        gcloud container clusters create $my_cluster --num-nodes 3 --enable-ip-alias --zone $my_zone --enable-network-policy

4. Configure access to your cluster for the kubectl command-line tool, using the following command

        gcloud container clusters get-credentials $my_cluster --zone $my_zone

5. Run a simple web server application with the label app=hello, and expose the web application internally in the cluster

        kubectl run hello-web --labels app=hello \
        --image=gcr.io/google-samples/hello-app:1.0 --port 8080 --expose

6. Enter the following command to clone the repository to the lab Cloud Shell

        git clone https://github.com/GoogleCloudPlatformTraining/training-data-analyst

7. Change to the directory that contains the sample files for this lab.

        cd ~/training-data-analyst/courses/ak8s/09_GKE_Networks/

# Restrict incoming traffic to Pods

1. Create an ingress policy.

    kubectl apply -f hello-allow-from-foo.yaml


2. Verify that the policy was created.

    kubectl get networkpolicy

# Validate the ingress policy

1. Run a temporary Pod called test-1 with the label app=foo and get a shell in the Pod.

    kubectl run test-1 --labels app=foo --image=alpine --restart=Never --rm --stdin --tty

2. Make a request to the hello-web:8080 endpoint to verify that the incoming traffic is allowed.

    wget -qO- --timeout=2 http://hello-web:8080

3. Type exit and press ENTER to leave the shell

4. Now you will run a different Pod using the same Pod name but using a label, app=other, that does not match the podSelector in the active network policy. This Pod should not have the ability to access the hello-web application.

    kubectl run test-1 --labels app=other --image=alpine --restart=Never --rm --stdin --tty

5. Make a request to the hello-web:8080 endpoint to verify that the incoming traffic is not allowed.

    wget -qO- --timeout=2 http://hello-web:8080

6. Type exit and press ENTER to leave the shell.


# Restrict outgoing traffic from the Pods

1. Create an egress policy.

    kubectl apply -f foo-allow-to-hello.yaml

2. Verify that the policy was created.

     kubectl get networkpolicy

# Validate the egress policy

1. Deploy a new web application called hello-web-2 and expose it internally in the cluster.

    kubectl run hello-web-2 --labels app=hello-2 \
    --image=gcr.io/google-samples/hello-app:1.0 --port 8080 --expose

2. Run a temporary Pod with the app=foo label and get a shell prompt inside the container.

    kubectl run test-3 --labels app=foo --image=alpine --restart=Never --rm --stdin --tty

3. Verify that the Pod can establish connections to hello-web:8080.

        wget -qO- --timeout=2 http://hello-web:8080

4. Verify that the Pod cannot establish connections to hello-web-2:8080.

        wget -qO- --timeout=2 http://hello-web-2:8080

5. Verify that the Pod cannot establish connections to external websites, such as www.example.com.

    wget -qO- --timeout=2 http://www.example.com



6. Type exit and press ENTER to leave the shell.

