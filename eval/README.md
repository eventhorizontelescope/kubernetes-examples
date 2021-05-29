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


## Step 1.  Setup Google Cloud Storage Bucket and Access

We first setup a Google Cloud Storage Bucket to stage the input and
output files:

    gsutil mb gs://<BUCKET_NAME>/

In order to access this bucket, we need to create a service account:

    gcloud iam service-accounts list # list existing service accounts
    gcloud iam service-accounts create <SHORT-NAME> --display-name="<LONG DISPLAY NAME>"

We then need to configure the necessary permission and create a service account key:

    gsutils iam ch serviceAccount:<SHORT-NAME>@<PROJECT_ID>.iam.gserviceaccount.com:objectAdmin gs://<BUCKET_NAME>/
    gcloud iam service-accounts keys create --iam-account <SHORT-NAME>@<PROJECT_ID>.iam.gserviceaccount.com service-account.json

The output file "service-account.json" contains sensitive information,
and should not be committed to git or added inside a container.
See this
[documentation](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)
for best practices on managing the keys.
