# Exploring Redis

This tutorial shows how you can use Docker to explore Redis. You can run the commands with Docker installed, or Docker for Windows in Linux mode. But you can also use [Play with Docker](https://labs.play-with-docker.com) to try this out.

Using some commands from the [Elasticsearch 6.4 Getting Started Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)

### Start a new container running elasticsearch
We're exposing ports 9200 (for the REST API) and 9300 (used for cluster communication - not really needed here), and setting up a single node cluster, from the official elasticsearch 6.4.2 image.

```
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.4.2
```

### Check the cluster health
With Linux containers communicate `localhost`. 
Docker for Windows you might need to use `docker inspect <containerid>` to find out the IP address.
Docker Toolbox will tell you an IP address e.g. 192.168.99.100


```bash
curl -X GET "localhost:9200/_cat/health?v"
```

If you're in PowerShell

```powershell
$eshost = "192.168.99.100" # or whatever
iwr "$($eshost):9200/_cat/health?v" | select -ExpandProperty Content
```

### Create an index

```bash
curl -X PUT "localhost:9200/customer?pretty"
```

or

```powershell
iwr -Method PUT "$($eshost):9200/customer?pretty" | select -ExpandProperty Content
```

### view the indexes:

```bash
curl -X GET "localhost:9200/_cat/indices?v"
```

PowerShell:
```powershell
Invoke-RestMethod -Uri "$($eshost):9200/_cat/indices?v"
```

### Add a document to the index

```bash
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "Mark Heath"
}
'
```

PowerShell:
```powershell
$json = @{ "name"="Mark Heath"} | ConvertTo-Json
Invoke-RestMethod -Method Put -Uri "$($eshost):9200/customer/_doc/1?pretty" -Body $json -ContentType "application/json"
```

### Retrieve a document

```bash
curl -X GET "localhost:9200/customer/_doc/1?pretty"
```

PowerShell:
```powershell
Invoke-RestMethod -Method Get -Uri "$($eshost):9200/customer/_doc/1?pretty"
```

### Update a document

```bash
curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'
```

```powershell
$json = @{ "doc" = @{ "name" = "Jane Doe"; "age" = 20}} | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/customer/_doc/1/_update?pretty" -Body $json -ContentType "application/json"
```

### Update with a script (TODO)
```bash
curl -X POST "localhost:9200/customer/_doc/1/_update?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.age += 5"
}
'
```

PowerShell:
```powershell
$json = @{ "script" = "ctx._source.age += 5"} | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/customer/_doc/1/_update?pretty" -Body $json -ContentType "application/json"
```

### Delete a Document
```bash
curl -X DELETE "localhost:9200/customer/_doc/1?pretty"
```

PowerShell:
```powershell
Invoke-RestMethod -Method Delete -Uri "$($eshost):9200/customer/_doc/1?pretty"
```


### Delete an Index
```bash
curl -X DELETE "localhost:9200/customer?pretty"
```

PowerShell:
```powershell
Invoke-RestMethod -Method Delete -Uri "$($eshost):9200/customer?pretty"
```

### Batch load
Using this [accounts.json](https://raw.githubusercontent.com/elastic/elasticsearch/master/docs/src/test/resources/accounts.json#) file.

```bash
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" --data-binary "@accounts.json"
```

PowerShell:
```powershell
$bulk = Get-Content .\accounts.json -Raw
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_doc/_bulk?pretty&refresh" -ContentType "application/json" -Body $bulk
```

### Simple Search Syntax

```bash
curl -X GET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
```

```PowerShell
Invoke-RestMethod -Uri "$($eshost):9200/bank/_search?q=*&sort=account_number:asc&pretty"
```

### Search with object

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```

Powershell (n.b. won't let us GET with a body):
```powershell
$json = @{
  "query"= @{ "match_all"= @{} };
  "sort"= @(
    @{ "account_number"= "asc" }
  )
 } | ConvertTo-Json

Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body $json
```

### Paged Search

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
'@
```

### Search with limited return fields

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
'@
```

### Search for a term

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill" } }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": { "match": { "address": "mill" } }
}
'@
```

### Search for a phrase

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'@
```


### Search must

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'@
```

### Search should

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
'@
```

### Search must not

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'@
```

### Search range

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'@
```

### Search with group (aggregations)

```bash
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'
```

```powershell
Invoke-RestMethod -Method Post -Uri "$($eshost):9200/bank/_search" -ContentType "application/json" -Body @'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
'@
```

### Volume

Create a container with a volume

```powershell
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" `
-v esdata:/usr/share/elasticsearch/data `
docker.elastic.co/elasticsearch/elasticsearch:6.4.2 
```

Create some indexes, add some documents.

Now delete the container

```bash
docker rm -f <container-id>
```

Then restart the container with the same command

```powershell
docker run -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" `
-v esdata:/usr/share/elasticsearch/data `
docker.elastic.co/elasticsearch/elasticsearch:6.4.2 
```

Request to see the indexes and the same indexes should be present.


### Cluster

Using the example [Docker Compose YAML file](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

Start up two node cluster
```
docker-compose up -d
```

Check health
```powershell
Invoke-RestMethod -Uri "$($eshost):9200/_cat/health?v"
```

complete cleanup
```
docker-compose down -v
```

