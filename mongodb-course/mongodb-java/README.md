# Mongodb for java developers

## MongoDB URI

```
mongodb://<username>:<password>@<cluster-node01-url:port><cluster-node02-url:port>,<cluster-node03-url:port>/<database>

or srv string (service record that holds the lists of hostnames of the cluster). we dont need to know each server in the cluster

mongodb+srv://<username>:<password>@<host>/database
```