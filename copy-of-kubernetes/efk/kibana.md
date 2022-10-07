# Kibana

## Kibana is not ready

### Option 1 (Index is in red state)

Content taken from here: [https://discuss.elastic.co/t/solved-kibana-update-7-9-1-to-7-13-1-failed/275532](https://discuss.elastic.co/t/solved-kibana-update-7-9-1-to-7-13-1-failed/275532)

Kibana reports errors in containers log

```
[.kibana_task_manager] Action failed with 'search_phase_execution_exception'. Retrying attempt 1 in 2 seconds.
[.kibana_task_manager] Action failed with 'search_phase_execution_exception'. Retrying attempt 2 in 4 seconds.
[.kibana_task_manager] Action failed with 'search_phase_execution_exception'. Retrying attempt 3 in 8 seconds.
[.kibana_task_manager] Action failed with 'search_phase_execution_exception'. Retrying attempt 4 in 16 seconds.
```

Above messages report error in is Elasticsearch broken index `.kibana_task_manager`

For further troubleshooting create side-car container

```
microk8s.kubectl run -it --rm --image alpine debug -- /bin/sh
```

Install curl.&#x20;

Use Elastic API to retrieve list of indexes ([https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-indices.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-indices.html)):

```
curl -k http://elasticsearch:9200/_cat/indices
```

Example of output

```
green  open .kibana_7.13.3_001              Ocw1pykZQTC5j_i3uIw_GA 1 0     107 0   4.2mb   4.2mb
yellow open app-2021.12.12                  1Fv8hpLhSMKWomqAWZffsw 1 1   82059 0  17.8mb  17.8mb
red    open app-2021.12.11                  XKCyVz8dSLCJS6DiRAL-yA 1 1                          
yellow open app-2021.12.07                  VmcKLudpStmgQvKfJw-58w 1 1   66708 0    23mb    23mb
yellow open app-2021.12.06                  Wv8WyT2uSuWDjkNdjYeLhg 1 1   55460 0    18mb    18mb
green  open .apm-agent-configuration        TcEp5m2mQ8C64m4c6reSiA 1 0       0 0    208b    208b
yellow open app-2021.12.10                  0jL6LbuhQ7Ge9OfnFhkz2g 1 1  840540 0 256.8mb 256.8mb
green  open .tasks                          7NPszsM4QMaUz5XKieFbGQ 1 0     311 0  68.4kb  68.4kb
red    open .kibana_task_manager_7.13.3_001 9BMWIikASpWtkz9BfzWNNA 1 0                          
green  open .apm-custom-link                zaPHY6fVRhaMnTSb89acUQ 1 0       0 0    208b    208b
green  open .kibana-event-log-7.13.3-000001 89BV2mMkSg6eJcI2t-TtLw 1 0       1 0   5.6kb   5.6kb
yellow open app-2021.12.09                  tKK3iByIScOrgzNcaWlA8w 1 1 1260962 0 442.3mb 442.3mb
yellow open app-2021.12.08                  zXgF1yLCS2mlolmfxUuNhQ 1 1  534325 0 180.6mb 180.6mb
green  open .async-search                   DRsgr24gT3WXLKgZ79oe2w 1 0       0 0   6.4kb   6.4kb
```

1-st column contains "color of health": red means smth is wrong and those indexes doesn't show any statistics. Note that `.kibana_task_manager` index has problems. So next step is just remove them.

```
curl -k -XDELETE http://elasticsearch:9200/.kibana_task_manager_7.13.3_001
curl -k -XDELETE http://elasticsearch:9200/app-2021.12.11
```

Restart ES and Kibana, the error must go.

### Option 2 (red index is a part of a data stream)

If index is a part of a data stream, then it can't be removed. Due to it could be active one that is open for writing.

Most probably you are hitting, Scenario 4 from this [list](https://www.datadoghq.com/blog/elasticsearch-unassigned-shards/#reason-4-shard-data-no-longer-exists-in-the-cluster)

1\) create temp pod

```
microk8s.kubectl run -it --rm --image alpine debug -- /bin/sh
```

2\) check state of indexes

```
curl -XGET elasticsearch:9200/_cluster/allocation/explain?pretty
```

if following will be found `"allocate_explanation" : "can allocate the shard"` means no shard allocated for this index.

{% hint style="danger" %}
And it can't be deleted with previous action (curl -k XDELETE ....)

"reason":"index \[.ds-apps-prod-ds-2022.09.20-000008] is the write index for data stream \[apps-prod-ds] and cannot be deleted"
{% endhint %}

3\) Allocate empty shard for the index. Notice that all data will be lost from this index after rollover.

```
curl -XPOST "elasticsearch:9200/_cluster/reroute?pretty" -H 'Content-Type: application/json' -d'
{
    "commands" : [
        {
          "allocate_empty_primary" : {
                "index" : ".ds-apps-prod-ds-2022.09.14-000007", ######## index name
                "shard" : 0,
                "node" : "elasticsearch-69f48fdf49-t694c",      ######## hostname
                "accept_data_loss" : "true"
          }
        }
    ]
}'
```

## Daily report

#### Top requests taken the most time (in milliseconds)

![](<../../.gitbook/assets/image (12).png>)

#### Number of ERROR or PANIC message logs

![](<../../.gitbook/assets/image (15).png>)
