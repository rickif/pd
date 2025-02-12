pd-ctl
========

pd-ctl is a command line tool for PD, pd-ctl obtains the state information of the cluster and tunes the cluster.

## Build
1. [Go](https://golang.org/) Version 1.9 or later
2. In the root directory of the [PD project](https://github.com/pingcap/pd), use the `make` command to compile and generate `bin/pd-ctl`

> **Note:** Generally, you don't need to compile source code as the PD Control tool already exists in the released Binary or Docker. However, dev users can refer to the above instruction for compiling source code.

## Usage

Single-command mode:

    ./pd-ctl store -u http://127.0.0.1:2379

Interactive mode:

    ./pd-ctl -i -u http://127.0.0.1:2379

Use environment variables:

```bash
export PD_ADDR=http://127.0.0.1:2379
./pd-ctl
```

Use TLS to encrypt:

```bash
./pd-ctl -u https://127.0.0.1:2379 --cacert="path/to/ca" --cert="path/to/cert" --key="path/to/key"
```

## Command line flags

### \-\-pd,-u

+ PD address
+ Default address: http://127.0.0.1:2379
+ Environment variable: PD_ADDR

### \-\-detach,-d

+ Use single command line mode (not entering readline)
+ Default: true

### \-\-interact,-i

+ Use interactive mode (entering readline)
+ Default: false

### --cacert

+ Specify the path to the certificate file of the trusted CA in PEM format
+ Default: ""

### --cert

+ Specify the path to the certificate of SSL in PEM format
+ Default: ""

### --key

+ Specify the path to the certificate key file of SSL in PEM format, which is the private key of the certificate specified by `--cert`
+ Default: ""

### --version,-V

+ Print the version information and exit
+ Default: false

## Command

### `cluster`

Use this command to view the basic information of the cluster.

Usage:

```bash
>> cluster                                     // To show the cluster information
{
  "id": 6493707687106161130,
  "max_peer_count": 3
}
```

### `config [show | set <option> <value>]`

Use this command to view or modify the configuration information.

Usage:

```bash
>> config show                                // Display the config information of the replication and schedule
{
  "replication": {
    "location-labels": "",
    "max-replicas": 3,
    "strictly-match-label": "false"
  },
  "schedule": {
    "enable-cross-table-merge": "false",
    "enable-location-replacement": "true",
    "enable-make-up-replica": "true",
    "enable-one-way-merge": "false",
    "enable-remove-down-replica": "true",
    "enable-remove-extra-replica": "true",
    "enable-replace-offline-replica": "true",
    "high-space-ratio": 0.6,
    "hot-region-cache-hits-threshold": 3,
    "hot-region-schedule-limit": 4,
    "leader-schedule-limit": 4,
    "leader-schedule-strategy": "\"count\"",
    "low-space-ratio": 0.8,
    "max-merge-region-keys": 200000,
    "max-merge-region-size": 20,
    "max-pending-peer-count": 16,
    "max-snapshot-count": 3,
    "max-store-down-time": "30m0s",
    "merge-schedule-limit": 8,
    "patrol-region-interval": "100ms",
    "region-schedule-limit": 2048,
    "replica-schedule-limit": 64,
    "scheduler-max-waiting-operator": 3,
    "schedulers": {
      "balance-hot-region-scheduler": "null",
      "balance-leader-scheduler": "null",
      "balance-region-scheduler": "null",
      "label-scheduler": "null"
    },
    "schedulers-v2": [
      {
        "args": null,
        "args-payload": "",
        "disable": false,
        "type": "balance-region"
      },
      {
        "args": null,
        "args-payload": "",
        "disable": false,
        "type": "balance-leader"
      },
      {
        "args": null,
        "args-payload": "",
        "disable": false,
        "type": "hot-region"
      },
      {
        "args": null,
        "args-payload": "",
        "disable": false,
        "type": "label"
      }
    ],
    "split-merge-interval": "1h0m0s",
    "store-balance-rate": 15,
    "tolerant-size-ratio": 0
  }
}
>> config show all                            // Display all config information
>> config show replication                    // Display the config information of replication
{
  "max-replicas": 3,
  "location-labels": ""
}
>> config show cluster-version                // Display the current version of the cluster, which is the current minimum version of TiKV nodes in the cluster and does not correspond to the binary version.
"2.0.0"
```

- `max-snapshot-count` controls the maximum number of snapshots that a single store receives or sends out at the same time. The scheduler is restricted by this configuration to avoid taking up normal application resources. When you need to improve the speed of adding replicas or balancing, increase this value.

    ```bash
    >> config set max-snapshot-count 16  // Set the maximum number of snapshots to 16
    ```

- `max-pending-peer-count` controls the maximum number of pending peers in a single store. The scheduler is restricted by this configuration to avoid producing a large number of Regions without the latest log in some nodes. When you need to improve the speed of adding replicas or balancing, increase this value. Setting it to 0 indicates no limit.

    ```bash
    >> config set max-pending-peer-count 64  // Set the maximum number of pending peers to 64
    ```

- `max-merge-region-size` controls the upper limit on the size of Region Merge (the unit is M). When `regionSize` exceeds the specified value, PD does not merge it with the adjacent Region. Setting it to 0 indicates disabling Region Merge.

    ```bash
    >> config set max-merge-region-size 16 // Set the upper limit on the size of Region Merge to 16M
    ```

- `max-merge-region-rows` controls the upper limit on the row count of Region Merge. When `regionRowCount` exceeds the specified value, PD does not merge it with the adjacent Region.

    ```bash
    >> config set max-merge-region-rows 50000 // Set the the upper limit on rowCount to 50000
    ```

- `split-merge-interval` controls the interval between the `split` and `merge` operations on a same Region. This means the newly split Region won't be merged within a period of time.

    ```bash
    >> config set split-merge-interval 24h  // Set the interval between `split` and `merge` to one day
    ```

- `enable-one-way-merge` controls the merge scheduler behavior. This means a region can only be merged into the next region of it.

    ```bash
    >> config set enable-one-way-merge true  // Enable one way merge.
    ```

- `enable-cross-table-merge` controls the merge scheduler behavior. This means two Regions can be merged with different table IDs.
This option only works when key type is "table".

    ```bash
    >> config set enable-cross-table-merge true  // Enable cross table merge.
    ```

- `patrol-region-interval` controls the execution frequency that `replicaChecker` checks the health status of Regions. A shorter interval indicates a higher execution frequency. Generally, you do not need to adjust it.

    ```bash
    >> config set patrol-region-interval 10ms // Set the execution frequency of replicaChecker to 10ms
    ```

- `max-store-down-time` controls the time that PD decides the disconnected store cannot be restored if exceeded. If PD does not receive heartbeats from a store within the specified period of time, PD adds replicas in other nodes.

    ```bash
    >> config set max-store-down-time 30m  // Set the time within which PD receives no heartbeats and after which PD starts to add replicas to 30 minutes
    ```

- `leader-schedule-limit` controls the number of tasks scheduling the leader at the same time. This value affects the speed of leader balance. A larger value means a higher speed and setting the value to 0 closes the scheduling. Usually the leader scheduling has a small load, and you can increase the value in need.

    ```bash
    >> config set leader-schedule-limit 4         // 4 tasks of leader scheduling at the same time at most
    ```

- `region-schedule-limit` controls the number of tasks scheduling the Region at the same time. This value avoids too many region balance operators being created. The default value is 2048 which suits enough for all kinds sizes of clusters, setting the value to 0 closes the scheduling. Usually the Region scheduling speed is limited by the store-limit, users do not need to customize this value. Only change it when you know exactly what you are doing.

    ```bash
    >> config set region-schedule-limit 2         // 2 tasks of Region scheduling at the same time at most
    ```

- `replica-schedule-limit` controls the number of tasks scheduling the replica at the same time. This value affects the scheduling speed when the node is down or removed. A larger value means a higher speed and setting the value to 0 closes the scheduling. Usually the replica scheduling has a large load, so do not set a too large value.

    ```bash
    >> config set replica-schedule-limit 4        // 4 tasks of replica scheduling at the same time at most
    ```

- `merge-schedule-limit` controls the number of Region Merge scheduling tasks. Setting the value to 0 closes Region Merge. Usually the Merge scheduling has a large load, so do not set a too large value.

    ```bash
    >> config set merge-schedule-limit 16       // 16 tasks of Merge scheduling at the same time at most
    ```

- `tolerant-size-ratio` controls the size of the balance buffer area. When the score difference between the leader or Region of the two stores is less than specified multiple times of the Region size, it is considered in balance by PD.

    ```bash
    >> config set tolerant-size-ratio 20        // Set the size of the buffer area to about 20 times of the average regionSize
    ```

- `low-space-ratio` controls the threshold value that is considered as insufficient store space. When the ratio of the space occupied by the node exceeds the specified value, PD tries to avoid migrating data to the corresponding node as much as possible. At the same time, PD mainly schedules the remaining space to avoid using up the disk space of the corresponding node.

    ```bash
    config set low-space-ratio 0.9              // Set the threshold value of insufficient space to 0.9
    ```

- `high-space-ratio` controls the threshold value that is considered as sufficient store space. When the ratio of the space occupied by the node is less than the specified value, PD ignores the remaining space and mainly schedules the actual data volume.

    ```bash
    config set high-space-ratio 0.5             // Set the threshold value of sufficient space to 0.5
    ```

- `cluster-version` is the version of the cluster, which is used to enable or disable some features and to deal with the compatibility issues. By default, it is the minimum version of all normally running TiKV nodes in the cluster. You can set it manually only when you need to roll it back to an earlier version.

    ```bash
    config set cluster-version 1.0.8              // Set the version of the cluster to 1.0.8
    ```

- `enable-remove-down-replica` is used to enable the feature of automatically deleting DownReplica. When you set it to `false`, PD does not automatically clean up the downtime replicas.

- `enable-replace-offline-replica` is used to enable the feature of migrating OfflineReplica. When you set it to `false`, PD does not migrate the offline replicas.

- `enable-make-up-replica` is used to enable the feature of making up replicas. When you set it to `false`, PD does not adding replicas for Regions without sufficient replicas.

- `enable-remove-extra-replica` is used to enable the feature of removing extra replicas. When you set it to `false`, PD does not remove extra replicas for Regions with redundant replicas.

- `enable-location-replacement` is used to enable the isolation level check. When you set it to `false`, PD does not improve the isolation level of Region replicas by scheduling.

### `health`

Use this command to view the health information of the cluster.

Usage:

```bash
>> health                                // Display the health information
{"health": "true"}
```

### `hot [read | write | store]`

Use this command to view the hot spot information of the cluster.

Usage:

```bash
>> hot read                             // Display hot spot for the read operation
>> hot write                            // Display hot spot for the write operation
>> hot store                            // Display hot spot for all the read and write operations
```

### `label [store <name> <value>]`

Use this command to view the label information of the cluster.

Usage:

```bash
>> label                                // Display all labels
>> label store zone cn                  // Display all stores including the "zone":"cn" label
```

### `member [delete | leader_priority | leader [show | resign | transfer <member_name>]]`

Use this command to view the PD members, remove a specified member, or configure the priority of leader.

Usage:

```bash
>> member                               // Display the information of all members
{
  "members": [......],
  "leader": {......},
  "etcd_leader": {......},
}
>> member delete name pd2               // Delete "pd2"
Success!
>> member delete id 1319539429105371180 // Delete a node using id
Success!
>> member leader show                   // Display the leader information
{
  "name": "pd",
  "addr": "http://192.168.199.229:2379",
  "id": 9724873857558226554
}
>> member leader resign // Move leader away from the current member
......
>> member leader transfer pd3 // Migrate leader to a specified member
......
```

### `operator [show | add | remove]`

Use this command to view and control the scheduling operation.

Usage:

```bash
>> operator show                                        // Display all operators
>> operator show admin                                  // Display all admin operators
>> operator show leader                                 // Display all leader operators
>> operator show region                                 // Display all Region operators
>> operator add add-peer 1 2                            // Add a replica of Region 1 on store 2
>> operator add add-learner 1 2                         // Add a learner replica of Region 1 on store 2
>> operator add remove-peer 1 2                         // Remove a replica of Region 1 on store 2
>> operator add transfer-leader 1 2                     // Schedule the leader of Region 1 to store 2
>> operator add transfer-region 1 2 3 4                 // Schedule Region 1 to stores 2,3,4
>> operator add transfer-peer 1 2 3                     // Schedule the replica of Region 1 on store 2 to store 3
>> operator add merge-region 1 2                        // Merge Region 1 with Region 2
>> operator add split-region 1 --policy=approximate     // Split Region 1 into two Regions in halves, based on approximately estimated value
>> operator add split-region 1 --policy=scan            // Split Region 1 into two Regions in halves, based on accurate scan value
>> operator remove 1                                    // Remove the scheduling operation of Region 1
```

### `ping`

Use this command to view the time that `ping` PD takes.

Usage:

```bash
>> ping
time: 43.12698ms
```

### `region <region_id> [--jq="<query string>"]`

Use this command to view the region information. For a jq formatted output, see [jq-formatted-json-output-usage](#jq-formatted-json-output-usage).

Usage:

```bash
>> region                               //　Display the information of all regions
{
  "count": 1,
  "regions": [......]
}

>> region 2                             // Display the information of the region with the id of 2
{
  "region": {
      "id": 2,
      ......
  }
  "leader": {
      ......
  }
}
```

### `region key [--format=raw|pb|proto|protobuf] <key>`

Use this command to query the region that a specific key resides in. It supports the raw and protobuf formats.

Raw format usage (default):

```bash
>> region key abc
{
  "region": {
    "id": 2,
    ......
  }
}
```

Protobuf format usage:

```bash
>> region key --format=pb t\200\000\000\000\000\000\000\377\035_r\200\000\000\000\000\377\017U\320\000\000\000\000\000\372
{
  "region": {
    "id": 2,
    ......
  }
}
```

### `region sibling <region_id>`

Use this command to check the adjacent Regions of a specific Region.

Usage:

```bash
>> region sibling 2
{
  "count": 2,
  "regions": [......],
}
```

### `region store <store_id>`

Use this command to list all Regions of a specific store.

Usage:

```bash
>> region store 2
{
  "count": 10,
  "regions": [......],
}
```

### `region topread [limit]`

Use this command to list Regions with top read flow. The default value of the limit is 10.

Usage:

```bash
>> region topread
{
  "count": 10,
  "regions": [......],
}
```

### `region topwrite [limit]`

Use this command to list Regions with top write flow. The default value of the limit is 10.

Usage:

```bash
>> region topwrite
{
  "count": 10,
  "regions": [......],
}
```

### `region topconfver [limit]`

Use this command to list Regions with top conf version. The default value of the limit is 10.

Usage:

```bash
>> region topconfver
{
  "count": 10,
  "regions": [......],
}
```

### `region topversion [limit]`

Use this command to list Regions with top version. The default value of the limit is 10.

Usage:

```bash
>> region topversion
{
  "count": 10,
  "regions": [......],
}
```

### `region check [miss-peer | extra-peer | down-peer | pending-peer | offline-peer | empty-region]`

Use this command to check the Regions in abnormal conditions.

Description of various types:

- miss-peer: the Region without enough replicas
- extra-peer: the Region with extra replicas
- down-peer: the Region in which some replicas are Down
- pending-peer：the Region in which some replicas are Pending

Usage:

```bash
>> region miss-peer
{
  "count": 2,
  "regions": [......],
}
```

### `scheduler [show | add | remove]`

Use this command to view and control the scheduling strategy.

Usage:

```bash
>> scheduler show                             // Display all schedulers
>> scheduler add grant-leader-scheduler 1     // Schedule all the leaders of the regions on store 1 to store 1
>> scheduler add evict-leader-scheduler 1     // Move all the region leaders on store 1 out
>> scheduler add shuffle-leader-scheduler     // Randomly exchange the leader on different stores
>> scheduler add shuffle-region-scheduler     // Randomly scheduling the regions on different stores
>> scheduler remove grant-leader-scheduler-1  // Remove the corresponding scheduler
```

### `store [delete | label | weight | remove-tombstone | limit] <store_id>  [--jq="<query string>"]`

Use this command to view the store information or remove a specified store. For a jq formatted output, see [jq-formatted-json-output-usage](#jq-formatted-json-output-usage).

Usage:

```bash
>> store                        // Display information of all stores
{
  "count": 3,
  "stores": [...]
}
>> store 1                      // Get the store with the store id of 1
  ......
>> store delete 1               // Delete the store with the store id of 1
  ......
>> store label 1 zone cn        // Set the value of the label with the "zone" key to "cn" for the store with the store id of 1
>> store weight 1 5 10          // Set the leader weight to 5 and region weight to 10 for the store with the store id of 1
>> store remove-tombstone       // Remove stores that are in tombstone state
>> store limit                  // Show limits for all stores
>> store limit all 5            // Limit 5 operators per minute for all stores
>> store limit 1 5              // Limit 5 operators per minute for store 1
```

### `tso`

Use this command to parse the physical and logical time of TSO.

Usage:

```bash
>> tso 395181938313123110        // Parse TSO
system:  2017-10-09 05:50:59 +0800 CST
logic:  120102
```

## Jq formatted JSON output usage

### Simplify the output of `store`

```bash
» store --jq=".stores[].store | { id, address, state_name}"
{"id":1,"address":"127.0.0.1:20161","state_name":"Up"}
{"id":30,"address":"127.0.0.1:20162","state_name":"Up"}
...
```

### Query the remaining space of the node

```bash
» store --jq=".stores[] | {id: .store.id, available: .status.available}"
{"id":1,"available":"10 GiB"}
{"id":30,"available":"10 GiB"}
...
```

### Query the distribution status of the Region replicas

```bash
» region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id]}"
{"id":2,"peer_stores":[1,30,31]}
{"id":4,"peer_stores":[1,31,34]}
...
```

### Filter Regions according to the number of replicas

For example, to filter out all Regions whose number of replicas is not 3:

```bash
» region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length != 3)}"
{"id":12,"peer_stores":[30,32]}
{"id":2,"peer_stores":[1,30,31,32]}
```

### Filter Regions according to the store ID of replicas

For example, to filter out all Regions that have a replica on store30:

```bash
» region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(any(.==30))}"
{"id":6,"peer_stores":[1,30,31]}
{"id":22,"peer_stores":[1,30,32]}
...
```

You can also find out all Regions that have a replica on store30 or store31 in the same way:

```bash
» region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(any(.==(30,31)))}"
{"id":16,"peer_stores":[1,30,34]}
{"id":28,"peer_stores":[1,30,32]}
{"id":12,"peer_stores":[30,32]}
...
```

### Look for relevant Regions when restoring data

For example, when [store1, store30, store31] is unavailable at its downtime, you can find all Regions whose Down replicas are more than normal replicas:

```bash
» region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length as $total | map(if .==(1,30,31) then . else empty end) | length>=$total-length) }"
{"id":2,"peer_stores":[1,30,31,32]}
{"id":12,"peer_stores":[30,32]}
{"id":14,"peer_stores":[1,30,32]}
...
```

Or when [store1, store30, store31] fails to start, you can find Regions where the data can be manually removed safely on store1. In this way, you can filter out all Regions that have a replica on store1 but don't have other DownPeers:

```bash
» region --jq=".regions[] | {id: .id, peer_stores: [.peers[].store_id] | select(length>1 and any(.==1) and all(.!=(30,31)))}"
{"id":24,"peer_stores":[1,32,33]}
```

When [store30, store31] is down, find out all Regions that can be safely processed by creating the `remove-peer` Operator, that is, Regions with one and only DownPeer:

```bash
» region --jq=".regions[] | {id: .id, remove_peer: [.peers[].store_id] | select(length>1) | map(if .==(30,31) then . else empty end) | select(length==1)}"
{"id":12,"remove_peer":[30]}
{"id":4,"remove_peer":[31]}
{"id":22,"remove_peer":[30]}
...
```
