##  Elasticsearch cluster and Kibana up and running  



## stage-1


default admin user for ```kubana``` is ```elastic```


### Notes 



### .env edit
```
The .env file sets environment variables that are used when you run the docker-compose.yml configuration file. Ensure  that you specify a strong password for the elastic and kibana_system users with the ELASTIC_PASSWORD and KIBANA_PASSWORD v ariables. These variable are referenced by the docker-compose.yml file. 
```
### Exposing ports
```


This configuration exposes port 9200 on all network interfaces. Because of how Docker handles ports, a port that isn’t bound to localhost leaves your Elasticsearch cluster publicly accessible, potentially ignoring any firewall settings. If you don’t want to expose port 9200 to external hosts, set the value for ES_PORT in the .env file to something like 127.0.0.1:9200. Elasticsearch will then only be accessible from the host machine itself.
```




# Setup 

source : https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

``` docker-compose up ```



### Get request to the elastic server : 

```
$ curl  -X GET  https://localhost:9200/
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

### Secure request : 
```
#copy ca cet from one of the nodes

docker cp  stage-1-es03-1:/usr/share/elasticsearch/config/certs/ca/ca.crt .

# Test https request : 
$ curl --cacert ca.crt -X GET  https://localhost:9200/ | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   459  100   459    0     0  11158      0 --:--:-- --:--:-- --:--:-- 38250
{
  "error": {
    "root_cause": [
      {
        "type": "security_exception",
        "reason": "missing authentication credentials for REST request [/]",
        "header": {
          "WWW-Authenticate": [
            "Basic realm=\"security\" charset=\"UTF-8\"",
            "Bearer realm=\"security\"",
            "ApiKey"
          ]
        }
      }
    ],
    "type": "security_exception",
    "reason": "missing authentication credentials for REST request [/]",
    "header": {
      "WWW-Authenticate": [
        "Basic realm=\"security\" charset=\"UTF-8\"",
        "Bearer realm=\"security\"",
        "ApiKey"
      ]
    }
  },
  "status": 401
}
```

### authentication

```
$ curl -u elastic:Q1w2e3r4t5y6 --cacert ca.crt -X GET  https://localhost:9200/ | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   531  100   531    0     0   4577      0 --:--:-- --:--:-- --:--:--  4916
{
  "name": "es01",
  "cluster_name": "docker-cluster",
  "cluster_uuid": "w0ZE-x5QQm-hHqqPyb4LJA",
  "version": {
    "number": "8.4.3",
    "build_flavor": "default",
    "build_type": "docker",
    "build_hash": "42f05b9372a9a4a470db3b52817899b99a76ee73",
    "build_date": "2022-10-04T07:17:24.662462378Z",
    "build_snapshot": false,
    "lucene_version": "9.3.0",
    "minimum_wire_compatibility_version": "7.17.0",
    "minimum_index_compatibility_version": "7.0.0"
  },
  "tagline": "You Know, for Search"
}

```



### Query Index using elastic rest api  :
```
$ curl -H "Content-Type:application/json" -u elastic:Q1w2e3r4t5y6 --cacert ca.crt -X GET  https://localhost:9200/products/_search -d '{ "query": { "match_all": {} } }'  | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   409  100   377  100    32  24646   2092 --:--:-- --:--:-- --:--:-- 51125
{
  "error": {
    "root_cause": [
      {
        "type": "index_not_found_exception",
        "reason": "no such index [products]",
        "resource.type": "index_or_alias",
        "resource.id": "products",
        "index_uuid": "_na_",
        "index": "products"
      }
    ],
    "type": "index_not_found_exception",
    "reason": "no such index [products]",
    "resource.type": "index_or_alias",
    "resource.id": "products",
    "index_uuid": "_na_",
    "index": "products"
  },
  "status": 404
}
```
ass expected no such index [products] 



## Interactig Elastic using Kibana/DevTolls/Console

```
GET  /_cluster/health
get /_cat/nodes/?v
put /pages
get /_cluster/health
get /_cat/indices?v

get /_cat/shards?v
```

Create indices with sharsd :


Request:

```
put /pages
{
  "settings":{
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}

```
Response 
``` 
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "pages"
}

```

Creating Document : 
```
post /products/_doc
{
  "name": "Coffe Maker",
  "price": 49,
  "in_stock": 4
} 
```


```console
{
  "_index": "products",
  "_id": "s4dQ2YQBdupJyOWaGEPV",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

Lets retrive the document with curl get request : 
```console
 $ curl -H "Content-Type:application/json" -u elastic:Q1w2e3r4t5y6 --cacert ca.crt -X GET  https://localhost:9200/products/_doc/s4dQ2YQBdupJyOWaGEPV  | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   177  100   177    0     0   5837      0 --:--:-- --:--:-- --:--:--  7695
{
  "_index": "products",
  "_id": "s4dQ2YQBdupJyOWaGEPV",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Coffe Maker",
    "price": 49,
    "in_stock": 4
  }
}
```# elk-stack
