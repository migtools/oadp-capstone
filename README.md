# oadp-capstone

# Step 5
In this step we will walkthrough the steps of installing a complex application
that will be used to demonstration the use of velero in a backup and restore scenario.

Available Applications
* Cassandra
* Postgres HA (Patroni)  

**Installing Cassandra**  
Please follow the link below for installing Cassandra.  
Running *ansible-playbook install.yaml* should set everything up.  

[Cassandra Example](https://github.com/konveyor/velero-examples/tree/master/cassandra "Cassandra")

End results should look like the following in openshift

![](Images/CassandraOpenshift.png "Cassandra Example")

Output should also have the looking when running the command 'oc get all -n cassandra-stateful' 
<pre>pod/cassandra-0   1/1     Running   0          6d1h
pod/cassandra-1   1/1     Running   0          6d1h
pod/cassandra-2   1/1     Running   0          6d1h

NAME                TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/cassandra   ClusterIP   None         &lt;none&gt;        9042/TCP   6d1h

NAME                         READY   AGE
statefulset.apps/cassandra   3/3     6d1h
</pre>

# Step 6
Now that we have a complex application on openshift, we can demonstrate a backup on the application.  
Click the link based on which application was installed.

-[Cassandra](#cassandra-app)

## Cassandra App
