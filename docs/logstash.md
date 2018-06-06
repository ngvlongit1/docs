# LogStash

## API Monitoring

Ref: https://www.elastic.co/guide/en/logstash/current/monitoring.html

### Node Info API

### Plugins Info API

```curl -XGET 'localhost:9600/_node/plugins?pretty'```


### Logging

Ref: https://www.elastic.co/guide/en/logstash/current/logging.html


```curl -XGET 'localhost:9600/_node/logging?pretty'```

```
curl -XPUT 'localhost:9600/_node/logging?pretty' -H 'Content-Type: application/json' -d'
{
    "logger.logstash.outputs.kafka" : "DEBUG"
}
'
```

### Node Info API
