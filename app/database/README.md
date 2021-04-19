### Postgres configuration in Kubernetes

Reference: https://sandeepbaldawa.medium.com/basic-postgres-database-in-kubernetes-23c7834d91ef

Make the installation using helm:
```
helm search repo postgres
helm install postgres bitnami/postgresql
helm get notes postgres
```

Get the password for Postgres:
```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)
```

We can run a client to access the DB:
```
kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:11.9.0-debian-10-r48 --env="PGPASSWORD=$POSTGRES_PASSWORD" --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432
```


I can expose the service as well if I want to manipulate the data graphically (not recommended for a DB in use in prod):
```
kubectl expose pod postgres-postgresql-0 --type=LoadBalancer --port=5432
```

This command will forward the service to the localhost (using Minikube):
```
minikube service postgres-postgresql-0
```

It is also important to take a look at the notes when using Helm:

helm get notes postgres
NOTES:
** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS name from within your cluster:

    postgres-postgresql.default.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)

To connect to your database run the following command:

    kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:11.11.0-debian-10-r9 --env="PGPASSWORD=$POSTGRES_PASSWORD" --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/postgres-postgresql 5432:5432 & \
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

