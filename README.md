# What is this?
It's a output plugin for [Fluentd](https://www.fluentd.org/), that sends data into [Yandex ClickHouse](https://clickhouse.yandex) database. By now it supports buffered output in json format.  

# Whats's difference from parent fork?
* Support http auth
* Inserting in ClickHouse database in json format so we don't need to enumerate fields in fluentd config and don't bother with escaping csv separators     

# How to use it?
I'm not a ruby programmer who knows how to write gems, so **just put [out_clickhousejson.rb](out_clickhousejson.rb) to /etc/td-agent/plugin**.  
There is mimimum fields in example td-agent.conf:
```
<source>
    @type http
    port 8888
</source>
<match inp>
    @type clickhousejson
    host 127.0.0.1
    port 8123
    table FLUENT
</match>
```

Additional fields:
```
<match inp>
    database <database>
    user <user>, default user is "default"
    password <password>, default password is ""
    datetime_name <field name>, field with DateTime value
    tz_offset <minutes>, timezone offset in minutes
<match inp>
```

Before launching td-agent, create table into ClickHouse:  
`CREATE TABLE FLUENT ( Date Date MATERIALIZED toDate(DateTime),  DateTime DateTime,  Str String,  Num Int32) ENGINE = MergeTree(Date, DateTime, 8192)`  
Start td-agent and send a few events to fluentd:  
```
curl -X POST -d 'json={"Num":1}' http://localhost:8888/inp
curl -X POST -d 'json={"Num":2}' http://localhost:8888/inp
curl -X POST -d 'json={"Num":3}' http://localhost:8888/inp
```
After a few seconds, when buffer flushes, in ClickHouse you could see this:
```:) SELECT * FROM FLUENT ;  
┌───────Date─┬────────────DateTime─┬─Str─┬─Num─┐  
│ 2017-11-06 │ 2017-11-06 14:42:03 │ inp │   1 │  
│ 2017-11-06 │ 2017-11-06 14:42:06 │ inp │   2 │  
│ 2017-11-06 │ 2017-11-06 14:42:09 │ inp │   3 │  
└────────────┴─────────────────────┴─────┴─────┘  
```
# There's still a work to do:  
* SSL
* Timezones that doesn't suck
* GZIP. ClickHouse supports compressing, so why not?
* and more
