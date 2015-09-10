---
title: ArmA Server RPT Log shipping to elasticsearch using heka
date: 2015-03-17
comments: true
---

Arma RPT Logfiles are essential to solve many mysteries around the arma game series. Since i'm hosting multiple arma servers, log management is quite horrible.
Especially if others need informations what went wrong and there is no way to give them access.
Previously this problem was solved by FTP access to the log directory, which was neither user friendly nor free of trouble (file locking).

I thought the best way to solve this whole problem on our new servers is proper logshipping.

---------

# Setup

Some steps:

* [Get `heka` running on windows]({% post_url 2015-03-17-build-heka-on-windows %})
* [Change arma 3 servers rpt directory]({% post_url 2015-03-17-arma3-server-rpt-change-directory %})
* Setup an `elasticsearch` server
* Run Heka using the config below

# Heka config
```
[hekad]
base_dir = "C:\\hekad\\data\\var"
share_dir = "C:\\hekad\\data\\share"

[ArmaRPT]
type = "ScribbleDecoder"
    [ArmaRPT.message_fields]
    Type = "arma.rpt"

[ArmaServer]
type = "ScribbleDecoder"
    [ArmaServer.message_fields]
    InstanceType = "server"

[ArmaInstance1]
type = "ScribbleDecoder"
    [ArmaInstance1.message_fields]
    InstanceId|INTEGER = "1"    

[rpt_decoder]
type = "PayloadRegexDecoder"
### If timestamp should be parsed otherwise:
#match_regex = '^(\s*(?P<Hour>\d+):(?P<Minute>\d+):(?P<Second>\d+)\s+)?(?P<Message>...*)'
match_regex = '^((?P<Timestamp>[^ ]+)\s+)?(?P<Message>...*)'
timestamp_layout = "15:04:05"

[rpt_decoder.message_fields]
Type = "arma.rpt"
Message = "%Message%"


[ArmaLogServerDecoder]
type = "MultiDecoder"
subs = ["rpt_decoder", "ArmaServer"]
cascade_strategy = "all"
log_sub_errors = true

[ArmaLogServer1Decoder]
type = "MultiDecoder"
subs = ["ArmaLogServerDecoder", "ArmaInstance1"]
cascade_strategy = "all"
log_sub_errors = true

[ArmaLogServer2Decoder]
type = "MultiDecoder"
subs = ["ArmaLogServerDecoder", "ArmaInstance2"]
cascade_strategy = "all"
log_sub_errors = true

[ArmaLogServer3Decoder]
type = "MultiDecoder"
subs = ["ArmaLogServerDecoder", "ArmaInstance3"]
cascade_strategy = "all"
log_sub_errors = true

[ArmaLogServer4Decoder]
type = "MultiDecoder"
subs = ["ArmaLogServerDecoder", "ArmaInstance4"]
cascade_strategy = "all"
log_sub_errors = true

############ Inputs ############

[ArmaLogServer1]
type = "LogstreamerInput"
log_directory = "C:\\Program Files (x86)\\Steam\\steamapps\\common\\Arma 3 Server\\logs_server1"
file_match = 'arma3server_(?P<Year>\d+)-(?P<Month>\d+)-(?P<Day>\d+)_(?P<Hour>\d+)-(?P<Minute>\d+)-(?P<Second>\d+).rpt'
priority = ["Year", "Month", "Day","Hour","Minute","Second"]
decoder = "ArmaLogServer1Decoder"

[PayloadEncoder]
append_newlines = false

[LogOutput]
message_matcher = "TRUE"
encoder = "PayloadEncoder"

[ESJsonEncoder]
es_index_from_timestamp = true
type_name = "%{Type}"
index = "armalogs-%{2006.01.02}"

[ElasticSearchOutput]
server = "http://192.168.0.111:6400"
message_matcher = "Type == 'arma.rpt'"
encoder = "ESJsonEncoder"
flush_interval = 5
```
