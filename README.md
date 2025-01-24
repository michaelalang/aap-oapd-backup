# OpenShift APIs for Data Protection (OADP) for Red Hat Ansbile Automation Platform Database

Taking a backup of your AAP Database
```
export STORAGE=<your OADP storage location>

envsubst < backup.yml | oc -n openshift-adp create -f -
``` 

Restoring the backup of your AAP Database

```
oc -n ansible-automation-platform scale --replicas=0 statefulset -l app.kubernetes.io/component=database,app.kubernetes.io/managed-by=automationcontroller-operator
oc -n ansible-automation-platform delete pvc -l app.kubernetes.io/component=database,app.kubernetes.io/managed-by=automationcontroller-operator
oc -n ansible-automation-platform delete statefulset -l app.kubernetes.io/component=database,app.kubernetes.io/managed-by=automationcontroller-operator
``` 

You can initiate the Restore as the tasks and web deployments should reconcile with the loss of the database.

```
export BACKUPID=$(oc -n openshift-adp get backup.velero.io  -l app=ansible-automation-platform -o name | cut -f2 -d'/')
envsubst < restore.yml | oc -n openshift-adp create -f -
```

Due to the duration of the Database restore (depending on your Database size) the web and task deployments might be in crashloop state as the 
Database was not active and serving correct content. 
Ensure to reconcile the pods accordingly after the Database restore has finished.

ToDo:

Restart the operator which we turned off 
```
oc -n ansible-automation-platform scale --replicas=1 deploy/odf-operator-controller-manager deploy/noobaa-operator
``` 

Verify functionality through following command

```
$ oc -n ansible-automation-platform exec -ti \
   $(oc -n ansible-automation-platform get pod -l app.kubernetes.io/component=database -o name) -c postgres \
   -- psql -U postgres -d automationcontroller -c "SELECT count(*) FROM main_job;"
 count
-------
    13
(1 row)
```

