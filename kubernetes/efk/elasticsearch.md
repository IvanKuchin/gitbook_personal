# ElasticSearch

## Timestamps

![](<../../.gitbook/assets/image (9).png>)

Timestamp (3) is the timestamp posted by application (for ex: app, apache2, etc...)

Timestamp (2) is the timestamp assigned by CRIO-container once that message has been taken from container stdio-stream.

Timestamp (1) is a Time\_Key calculated by Fluentbit from (2) or (3) depends on \[PARSER] config.&#x20;

Example of parser:

```
    [PARSER]
        Name   apache2_crio
        Format regex
        Regex  ^(?<crio_log_time>.+) ........
        Time_Keep   On
        Time_Key    crio_log_time
        Time_Format %Y-%m-%dT%H:%M:%S.%L%z
```

In this config Time\_Key calculated based on `crio_log_time` capture group.

{% hint style="warning" %}
If time zone information is not captured then UTC will be used by default. Which means that Kibana settings must be set to UTC time zone. \
Menu -> Management -> Stack Management -> Kibana -> Advanced Settings -> Timezone for date formatting
{% endhint %}

Calculated timestamp (1) is represented as seconds since epoch. Depends on above setting it will have different value.&#x20;

Here are couple use cases:

* if you want timestamp (1) match (2) and (3), then Kibana timezone must be set to application timezone
* if you want timestamp (1) show events time in local time, then use either local timezone or "browser" time zone.

Check out [associated post](fluent-bit.md#log-format-docker-vs-cri-o) about log format difference.

## Naming conventions

{% embed url="https://www.elastic.co/guide/en/fleet/7.13/data-streams.html#data-streams-naming-scheme" %}

#### Datasets

* Application logs: `app-prod`
* Apache logs: `apache-prod`

## Testing data received from fluent-bit

Create mapping

```json
DELETE test-index
PUT test-index
{
    "mappings" : {
      "properties" : {
        "@timestamp" : {
          "type" : "date"
        },
        "kubernetes" : {
          "properties" : {
            "annotations" : {
              "properties" : {
                "cni_projectcalico_org/podIP" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                },
                "cni_projectcalico_org/podIPs" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                },
                "kubectl_kubernetes_io/restartedAt" : {
                  "type" : "date"
                }
              }
            },
            "container_hash" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "container_image" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "container_name" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "docker_id" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "host" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "labels" : {
              "properties" : {
                "pod-template-hash" : {
              "type" : "keyword",
              "ignore_above" : 256
                },
                "type" : {
              "type" : "keyword",
              "ignore_above" : 256
                }
              }
            },
            "namespace_name" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "pod_id" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "pod_name" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "log" : {
          "type" : "text"
        },
        "log_processed" : {
          "properties" : {
            "chrono_duration" : {
              "type" : "long"
            },
            "crio_log_stream" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "crio_log_tag" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "crio_log_time" : {
              "type" : "date"
            },
            "datetime" : {
              "type" : "date"
            },
            "line_number" : {
              "type" : "long"
            },
            "log_message" : {
              "type" : "text"
            },
            "log_severity" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "pid" : {
              "type" : "long"
            },
            "pretty_function" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
}

```

Line (91) should be changed to date once app logs will be changed to proper timestamp format



Create document in that index:

```json
POST /test-index/_doc
{
    "kubernetes.labels.type": "app",
    "log": "2022-03-16T00:50:09.344001582+03:00 stdout F Mar 16 2022 00:50:08.399558[3191] DEBUG: c_config_cache::IsInCache[5] start",
    "log_processed.log_message": "start",
    "log_processed.crio_log_stream": "stdout",
    "kubernetes.annotations.cni_projectcalico_org/podIPs": "10.1.128.255/32",
    "kubernetes.labels.pod-template-hash": "77d97ffb69",
    "kubernetes.container_image": "sha256:beae173ccac6ad749f76713cf4440fe3d21d1043fe616dfbe30775815d1d0f6a",
    "log_processed.datetime": "2022-03-15T00:50:08.399558+03:00",
    "kubernetes.host": "microk8s",
    "log_processed.line_number": 5,
    "kubernetes.annotations.kubectl_kubernetes_io/restartedAt": "2022-01-30T22:02:09.000Z",
    "kubernetes.container_name": "log-app-container",
    "kubernetes.container_hash": "docker.io/library/busybox@sha256:5acba83a746c7608ed544dc1533b87c737a0b0fb730301639a0179f9344b1678",
    "log_processed.crio_log_time": "2022-03-15T21:50:09.344Z",
    "kubernetes.namespace_name": "www-infomed-stat-ru",
    "kubernetes.docker_id": "48f36045d5b38b2a373fd294411f51ee2f568b9fd3ac5e9cd44c1e983ac3b39f",
    "log_processed.pid": 3191,
    "kubernetes.annotations.cni_projectcalico_org/podIP": "10.1.128.255/32",
    "log_processed.crio_log_tag": "F",
    "@timestamp": "2022-03-15T21:50:09.344Z",
    "kubernetes.pod_id": "55e7e2e0-b138-4c62-9f87-e1e3c5fdc3ff",
    "log_processed.log_severity": "DEBUG",
    "kubernetes.pod_name": "app-deployment-77d97ffb69-7dvj5",
    "log_processed.pretty_function": " c_config_cache::IsInCache"
}
```

Get index content:

```
GET test-index/_search
```

Remove index

```
DELETE test-index
```

## Data stream settings

1. Create "index lifecycle policy"  (name: apps-prod-policy)
   1. Hot phase: 10GB max shard size or 30 days old&#x20;
   2. Remove after 30 days
2. Create component templates
   1. apps-prod-mappings
   2. apps-prod-settings
3. Create "index template" (name: apps-prod-index-template)
4. Start sending data to data-stream

### Data stream: app-prod

#### Component template: apps-prod-mappings

```json
PUT _component_template/apps-prod-mappings
{
  "template": {
    "mappings" : {
      "properties" : {
        "@timestamp" : {
          "type" : "date"
        },
        "kubernetes" : {
          "properties" : {
            "annotations" : {
              "properties" : {
                "cni_projectcalico_org/podIP" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                },
                "cni_projectcalico_org/podIPs" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                },
                "kubectl_kubernetes_io/restartedAt" : {
                  "type" : "date"
                }
              }
            },
            "container_hash" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "container_image" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "container_name" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "docker_id" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "host" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "labels" : {
              "properties" : {
                "pod-template-hash" : {
              "type" : "keyword",
              "ignore_above" : 256
                },
                "type" : {
              "type" : "keyword",
              "ignore_above" : 256
                }
              }
            },
            "namespace_name" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "pod_id" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "pod_name" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "log" : {
          "type" : "text"
        },
        "log_processed" : {
          "properties" : {
            "chrono_duration" : {
              "type" : "long"
            },
            "crio_log_stream" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "crio_log_tag" : {
              "type" : "keyword",
              "ignore_above" : 256
            },
            "crio_log_time" : {
              "type" : "date"
            },
            "datetime" : {
              "type" : "date"
            },
            "line_number" : {
              "type" : "long"
            },
            "log_message" : {
              "type" : "text"
            },
            "log_severity" : {
              "type" : "text",
              "fields" : {
                "keyword" : {
                  "type" : "keyword",
                  "ignore_above" : 256
                }
              }
            },
            "pid" : {
              "type" : "long"
            },
            "pretty_function" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  },
  "_meta": {
    "description": "Mappings for K8s applications"
  }
}

```

#### Component template: apps-prod-settings

```json
PUT _component_template/apps-prod-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "apps-prod-policy"
    }
  },
  "_meta": {
    "description": "ILM settings"
  }
}
```

#### Index template

```json
PUT _index_template/apps-prod-index-template
{
  "index_patterns": ["apps-prod-ds*"],
  "data_stream": { },
  "composed_of": [ "apps-prod-mappings", "apps-prod-settings" ],
  "priority": 500,
  "_meta": {
    "description": "Index template for apps series data"
  }
}
```



### Data stream: apache-prod

#### Component template: apache-prod-mappings

```json
PUT _component_template/apache-prod-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "kubernetes": {
          "properties": {
            "annotations": {
              "properties": {
                "cni_projectcalico_org/podIP": {
                  "type": "keyword",
                  "ignore_above": 256
                },
                "cni_projectcalico_org/podIPs": {
                  "type": "keyword",
                  "ignore_above": 256
                },
                "kubectl_kubernetes_io/restartedAt": {
                  "type": "date"
                }
              }
            },
            "container_hash": {
              "type": "keyword",
              "ignore_above": 256
            },
            "container_image": {
              "type": "keyword",
              "ignore_above": 256
            },
            "container_name": {
              "type": "keyword",
              "ignore_above": 256
            },
            "docker_id": {
              "type": "keyword",
              "ignore_above": 256
            },
            "host": {
              "type": "keyword",
              "ignore_above": 256
            },
            "labels": {
              "properties": {
                "pod-template-hash": {
                  "type": "keyword",
                  "ignore_above": 256
                },
                "type": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "namespace_name": {
                  "type": "keyword",
                  "ignore_above": 256
            },
            "pod_id": {
                  "type": "keyword",
                  "ignore_above": 256
            },
            "pod_name": {
                  "type": "keyword",
                  "ignore_above": 256
            }
          }
        },
        "log": {
          "type": "text",
        },
        "log_processed": {
          "properties": {
            "agent": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 256
                }
              }
            },
            "code": {
              "type": "long",
            },
            "crio_log_stream": {
                  "type": "keyword",
                  "ignore_above": 256
            },
            "crio_log_tag": {
                  "type": "keyword",
                  "ignore_above": 256
            },
            "crio_log_time": {
              "type": "date"
            },
            "host": {
              "type": "IP",
              "ignore_malformed": true
            },
            "method": {
                  "type": "keyword",
                  "ignore_above": 256
            },
            "path": {
                  "type": "keyword",
                  "ignore_above": 256
            },
            "referer": {
              "type": "text",
            },
            "size": {
              "type": "long",
            },
            "time": {
              "type": "date",
              "format": "dd/MMM/yyyy:HH:mm:ss Z"
            },
            "user": {
              "type": "text",
            }
          }
        }
      }
    }
  }
}
```

#### Component template: apache-prod-settings

```json
PUT _component_template/apache-prod-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "apache-prod-policy"
    }
  },
  "_meta": {
    "description": "ILM settings"
  }
}
```

#### Index template

```json
PUT _index_template/apache-prod-index-template
{
  "index_patterns": ["apache-prod-ds*"],
  "data_stream": { },
  "composed_of": [ "apache-prod-mappings", "apache-prod-settings" ],
  "priority": 500,
  "_meta": {
    "description": "Index template for apps series data"
  }
}
```

