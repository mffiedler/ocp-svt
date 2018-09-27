# Shards and Replicas
Changing the number of shards or replicas only takes effect at index creation
## Shards
* [How many shards?](https://www.elastic.co/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster)
* [Shard best practices](https://www.objectrocket.com/blog/elasticsearch/clustered-elasticsearch-indexing-shard-replica-best-practices/)

The degree to which an index is spread across cluster members.   For a cluster with 3 members, creating a template which sets shards to 3 will put 1/3 of the index on each cluster member.   This is the most performant, but also creates the largest number of total shards in the cluster.

The number of shards per node should be kept below 20 to 25 per GB of heap it has configured.  30GB heap = max 600-750 shards.   Shards are only for open indices?

### shards.json:
```json
{
    "order": 20,
    "settings": {
        "index": {
            "number_of_shards": 3
        }
    },
    "template": "project.*"
}
```
### create template for new indices
```sh
file=shards.json
cat $file | \
oc exec -n openshift-logging -i -c elasticsearch $POD -- \
    curl -s -k --cert /etc/elasticsearch/secret/admin-cert \
    --key /etc/elasticsearch/secret/admin-key \
    https://localhost:9200/_template/$file -XPUT -d@- | \
python -mjson.tool
```

## Replicas
* [How replica shards work](https://www.elastic.co/guide/en/elasticsearch/guide/current/replica-shards.html)
Replica shards are exactly that - they do just as much work at indexing time and provide a copy of a shard that can serve for HA purposes and for load balancing purposes.  It can be set as above, but using **number_of_replicas**.

* 0 replicas is most performant for indexing (not query)
* 1 replica provides backup

## Display number shards for the cluster
```sh
# oc exec $POD -c elasticsearch -- /usr/local/bin/es_cluster_health
{
  "cluster_name" : "logging-es",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 5706,
  "active_shards" : 5706,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```
## Display shards and replicas for each index
```sh
# oc exec -n openshift-logging -c elasticsearch $POD -- curl --connect-timeout 2 -s -k --cert /etc/elasticsearch/secret/admin-cert --key /etc/elasticsearch/secret/admin-key https://logging-es:9200/_cat/indices?v
health status index                                                              uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   project.logtest342.b5085d5f-c1c9-11e8-b791-fa163eaa1723.2018.09.27 PcJL0QxdTgKCjGp0uoetNg   3   0      50000            0    233.9mb        233.9mb
green  open   project.logtest868.3de51926-c1ca-11e8-b791-fa163eaa1723.2018.09.26 WRz5T-oUR26NzDL57tIylg   3   0      50000            0    232.6mb        232.6mb
```
