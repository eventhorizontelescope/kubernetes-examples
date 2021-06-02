# Autoscaling through Kubernetes

This repository contains an example workflow that takes advantage of
horizonal autoscaling of Kubernetes.

It is based on this
[example](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/),
where
i) jobs are submitted to a work queue;
ii) multiple kubernetes pods are created;
iii) each pods take multiple work items from the queue, until all work are done.

In addition, this example uses Google Cloud Storage as the storage
backend.
This allows k8s to scale to thousands of pods.


## Step 1. Setting up Google Cloud Storage Bucket

On ng-eht-cloud, the storage is set up for you.
You may skip this step if you want to work on data on ng-eht-cloud.

We first setup a Google Cloud Storage Bucket to stage the input and
output files:

    gsutil mb gs://<BUCKET_NAME>/

In order to access this bucket, we need to create a service account:

    gcloud iam service-accounts list # list existing service accounts
    gcloud iam service-accounts create <short-name> --display-name="<LONG DISPLAY NAME>"

## Step 2. Setting up GCS Access

On ng-eht-cloud, we currently use method 2b to simplify GCS access.
The solution is still very salable, although we may be able to reduce
cost by using method 2a.

### Method 2a. Using Service Account

We then need to configure the necessary permission and create a service account key:

    gsutil iam ch \
        serviceAccount:<short-name>@<PROJECT_ID>.iam.gserviceaccount.com:objectAdmin \
        gs://<BUCKET_NAME>/
    gcloud iam service-accounts keys create \
        --iam-account <short-name>@<PROJECT_ID>.iam.gserviceaccount.com \
        service-account.json

The output file "service-account.json" contains sensitive information,
and should not be committed to git or added inside a container.
See this
[documentation](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)
for best practices on managing the keys.

Specifically, we will use
[Kubernetes Secret](https://kubernetes.io/docs/concepts/configuration/secret/)
to handle our key:

    kubectl get secret # list secret
    kubectl create secret generic <app-key> --from-file service-account.json

### Method 2b. Mounting GCS to Continer with `gcsfuse`

A simplier method is to use k8s' `lifecycle` to pre-mount the Google
Cloud Storage using gcsfuse.
We provide a sample YAML file "gcs-mount.yaml" for this.

Please replace `<BUCKET_NAME>` in that file with the proper GCS bucket
name, and then run:

    kubectl apply -f gcs-mount.yaml

One can then view the GCS bucket inside the container:

    kubectl exec gcs-mount-<ID> -it -- '/bin/sh'
    # ls /mnt/eval/
    index input


## Step 3. Setting up and Testing Redis for Work Queue

On ng-eht-cloud, a redis based work queue is set up.
You may skip this step if you want to work on data on ng-eht-cloud.

We follow this
[example](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/)
to use Redis for a work queue.
It is straightforward to deploy the redis YAML files come with this
repository:

    kubectl apply -f redis-pod.yaml
    kubectl apply -f redis-service.yaml

You may test redis by:

    kubectl run test -it --image redis --command '/bin/sh'

    # redis-cli -h redis
    redis:6379> rpush job2 "apple"
    (integer) 1
    redis:6379> rpush job2 "banana"
    (integer) 2
    redis:6379> lrange job2 0 -1
    1) "apple"
    2) "banana"

After you are done, you may connect back to the running test pod by:

    kubectl exec test -it -- '/bin/bash'

or you may delete the test pod by:

    kubectl get pods
    kubectl delete pod test
