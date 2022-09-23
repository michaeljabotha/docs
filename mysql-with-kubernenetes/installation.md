---
description: This shouldn't taken so long. (That's what...)
---

# Installation

## Installing a HA innoDB cluster using presslabs.

{% hint style="danger" %}
NOTE: I got the presslabs mysql operatoring but it is running 5.7 and we need 8 so we need to wait until they have updated their repo until we try that again.
{% endhint %}

Turns out that longhorn that manages our storage on kubernetes does not have the NFS kernel server running so lets install that so that we can claim a persistent volume.

```
sudo apt install nfs-kernel-server
```

Next lets install the helm chart for the MySQL cluster we will be using.

```
helm repo add presslabs https://presslabs.github.io/charts
```

Next we want to install the MySQL operator, I don't know much about this operator but it as far as I know it is in charge of managing the cluster (s) running different MySQL masters and slaves.

```
helm install mysql-operator presslabs/mysql-operator -n mysql-operator --create-namespace
```

## Installing a single node cluster MySQL instance.

Create a persistent vol in long horn called "mysql-development" this creates a volume that will persist after the pod is killed. Once created create a pvc on the volume using longhorn by clicking on the options of the volumn, call this mysql-development too.&#x20;

([https://www.fatalerrors.org/a/kubernetes-deploy-mysql-8.0-database-single-node.html](https://www.fatalerrors.org/a/kubernetes-deploy-mysql-8.0-database-single-node.html))

{% hint style="info" %}
Note the lower\_case\_table\_names = 1&#x20;
{% endhint %}

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-development-config
  labels:
    app: mysql-development
data:
  my.cnf: |-
    [client]
    default-character-set=utf8mb4
    [mysql]
    default-character-set=utf8mb4
    [mysqld] 
    max_connections = 2000
    secure_file_priv=/var/lib/mysql
    sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
    lower_case_table_names = 1
```

```
## Service
apiVersion: v1
kind: Service
metadata:
  name: mysql-development
  labels:
    app: mysql-development
spec:
  type: NodePort
  ports:
  - name: mysql-development
    port: 3306
    targetPort: 3306
    nodePort: 30336
  selector:
    app: mysql-development
---
## Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-development
  labels:
    app: mysql-development
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-development
  template:
    metadata:
      labels:
        app: mysql-development
    spec:     
      containers:
      - name: mysql-development
        image: mysql:8.0.19
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD    ## Configure Root user default password
          value: "f0rcel3nk"
        resources:
          limits:
            cpu: 2000m
            memory: 512Mi
          requests:
            cpu: 2000m
            memory: 512Mi
        livenessProbe:
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
          exec:
            command: ["mysqladmin", "-uroot", "-p${MYSQL_ROOT_PASSWORD}", "ping"]
        readinessProbe:  
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 3
          exec:
            command: ["mysqladmin", "-uroot", "-p${MYSQL_ROOT_PASSWORD}", "ping"]
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        - name: config
          mountPath: /etc/mysql/conf.d/my.cnf
          subPath: my.cnf
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: mysql-development
      - name: config      
        configMap:
          name: mysql-development-config
```

Write the above files to config.yaml and deploy.yaml respectively and apply them.

```
kubectl apply -f config.yaml
kubectl apply -f deploy.yaml
```

At this point you should have a MySQL pod running. But how do you connect to this database, you pull the MySQL image and create a pod in the cluster.

```
kubectl -n default run mysql-client --image=mysql:8.0.19 -it --rm --restart=Never -- /bin/bash
```

If you have already a created this pod and want to get back in:

```
kubectl exec -it mysql-client -- /bin/bash
#OR JUST DELETE IT
kubectl delete pod mysql-client
```

Once your pod is created and logged in you can now connect to your MySQL pod using the service that it created (Top of the deployment config above):\
mysql-development.default.svc.cluster.local (This is the alias that is given: \<deployment\_name>.\<namespace>.svc.cluster.local) and since we exposed 30336 to the whole cluster using nodeport we anyone in the cluster can connect. People outside the cluster can not connect on this port. But if we in the same namespace it will be exposed on 3306 not the specified 30336.

Now lets connect to it. In the pod that you created (mysql-client)

```
mysql -uroot -p'f0rcel3nk' -h 'mysql-development.default.svc.cluster.local'
```

Now you can add the:

## ****[**permissions** and](https://app.gitbook.com/@acumen-software/s/mysql/general-mysql-pages/adding-a-permission)

You must make sure that the IP matches your clusters!

## ****[**dump** the tables.](https://app.gitbook.com/@acumen-software/s/mysql/general-mysql-pages/creating-a-mysql-dump)

Create the dump files and copy them to your machine. Now to restore that back up... first copy your data into your mysql-client:

{% hint style="info" %}
Copy forcelink\_control\_data.sql local file to /tmp/forcelink\_controle\_data.sql in a remote pod in namespace**.**
{% endhint %}

```
kubectl cp forcelink_control_data.sql default/mysql-client:/tmp/forcelink_control_data.sql
```

First create the database if there is no database there:

To see the database name in MySQL CLI

```
SHOW DATABASES;
```

Create these on the new DB

```
create database forcelink_control;
create database forcelink_teti;

// Need this too
SET GLOBAL log_bin_trust_function_creators = 1;
```

```
#mysql -u [user] -p [database_name] < [filename].sql

mysql -uroot -p'f0rcel3nk' -h 'mysql-development.default.svc.cluster.local' forcelink_control < forcelink_control_tables.sql
mysql -uroot -p'f0rcel3nk' -h 'mysql-development.default.svc.cluster.local' forcelink_control < forcelink_control_data.sql
mysql -uroot -p'f0rcel3nk' -h 'mysql-development.default.svc.cluster.local' forcelink_teti < forcelink_teti_tables.sql
mysql -uroot -p'f0rcel3nk' -h 'mysql-development.default.svc.cluster.local' forcelink_teti < forcelink_teti_data.sql
```

{% hint style="warning" %}
YOU HAVE TO MAKE SURE THAT YOUR CURRENT HOST IN FL AND IN THE DATABASE MATCH!
{% endhint %}

If you don't do this you will get an error on login, something about a hibernate transaction error on login as there will be no active schemas. Check this by doing the following:

In the my-sql CLI run the following. It will show you the currently set default and current host.

```
USE forcelink_control;
select subscriber_id, defaulthost, currenthost from subscriber_database;
```

If these do no match your host you will have to change them with the following: (development.forcelink.net being your new host name)

```
UPDATE subscriber_database SET currenthost = 'development.forcelink.net' WHERE active=1;
UPDATE subscriber_database SET defaulthost = 'development.forcelink.net' WHERE active=1;
```

Once complete point your Forcelink instance to the right service and you should be sorted.
