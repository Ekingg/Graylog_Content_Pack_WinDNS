# Windows DNS Content Pack

Tested with nxLog/Graylog 1.2

## Includes

* Input (GELF udp 5414)
* Extractor (WinDNS_Debug_Log)
* GROK Patterns
* Dashboard (WinDNS Summary)

## Requirements

* Windows DNS server configured for "Log packets for debugging" & "Packet direction: Incoming"
* A GELF supported log exporter/collector such as nxlog or Graylog Collector monitoring the log file path
* create a ES template to hard set the ThreadID field type to "String", otherwise ES may dynamically map the field type to INT and you'll get indexing errors later on when an alphanumeric ThreadID comes around.

```
curl -XPUT localhost:9200/_template/graylog -d '
{
  "template":"graylog*",
  "settings":{
    "index.refresh_interval":"30s"
    },
    "mappings":{
      "message":{
        "properties":{
          "ThreadID":{
            "index":"not_analyzed",
            "type":"String"
          }
        }
      }
    }
}'
```

## NXLog Configuration Example
```
define ROOT C:\Program Files (x86)\nxlog

Moduledir %ROOT%\modules
CacheDir %ROOT%\data
Pidfile %ROOT%\data\nxlog.pid
SpoolDir %ROOT%\data
LogFile %ROOT%\data\nxlog.log

<Extension gelf>
    Module xm_gelf
</Extension>

<Input dns>
    Module  im_file
    File  "C:\dns.txt"
    SavePos TRUE
    InputType LineBased
</Input>

<Output out> 
    Module      om_udp
    Host        graylog.server.com
    Port        5414
    OutputType  GELF
</Output>

<Route 2>
    Path        dns => out
</Route>
```

## Screenshots

![Dashboard](http://i0.wp.com/www.ohjeah.net/wp-content/uploads/2015/09/windows_dns_logs.png)