# Autoscaling through Kubernetes

This repository contains an example workflow that takes advantage of
horizonal autoscaling of Kubernetes.

It is basedon this
[example](https://kubernetes.io/docs/tasks/job/fine-parallel-processing-work-queue/),
where
i) jobs are submitted to a work queue;
ii) multiple kubernetes pods are created;
iii) each pods take multiple work items from the queue, until all work are done.

In addition, this example uses Google Cloud Storage as the storage backend.
