# EFK

EFK stands for Elastic, Fluentd / Fluent-bit. Kibana

## Logs architecture

### Application logs

Application sends logs to file on local storage. Here are challenges:

*   File size: file will be growing up until POD restarts. If POD will alive for a long time or site will be popular file size might become huge. Example

    * connme's app log file in 2 months grew up to 1GB
    * timecard's app log file in 2 month grew up to 10GB

    To address the problem log rotation is required which is not easy to implement in containerized environment
* Reading from it requires side-car container in the POD
* Files might be needed for incidents investigations

### Apache logs

Web-server logs saved in persistent storage, just for purposes of log analysis (goaccess).

Following considerations must be taken into account:

* apache log persistent storage is a bad idea due to it prevents POD horizontal scaling
* log rotation is very complicated due to persistent storage doesn't have any reference to actual POD. If files will be deleted than Apache must be restarted, but there is no link between Apache inside POD and persistent log storage.
