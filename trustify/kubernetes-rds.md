---
title: Attaching to an AWS RDS instance
layout: page
permalink: /trustify/kubernetes-rds/
nav_order: 2
parent: Kubernetes
---

# Attaching to an AWS RDS instance

Sometimes it may be helpful to have direct access to the database. In the case
of AWS RDS, this can be a bit more difficult, as the database is not directly
accessible, for good reasons.

However, as the Kubernetes/OpenShift cluster has access to the database, it is
possible to run a container/pod with the `psql` command line to on the cluster
to access it.

You can run the following command to create a psql pod:

`kubectl create -f examples/psql-pod.yaml`

This requires the file `psql-pod.yaml` with the following content:

```
apiVersion: v1
kind: Pod
metadata:
  name: psql
spec:
  containers:
    - name: run
      image: docker.io/library/postgres:17
      stdin: true
      tty: true
      command:
        - psql
      env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: db.password
        - name: PGUSER
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: db.user
        - name: PGDATABASE
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: db.name
        - name: PGPORT
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: db.port
        - name: PGHOST
          valueFrom:
            secretKeyRef:
              name: postgresql-credentials
              key: db.host
```

The secret `postgresql-credentials` is created by the Terraform script.

Afterward, you can attach to the pod by running:

`kubectl attach -ti psql`

Once you are finished, delete the pod using:

`kubectl delete pod psql`
