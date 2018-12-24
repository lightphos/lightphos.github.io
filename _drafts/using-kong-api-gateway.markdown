---
layout: post
title: Using Kong API Gateway
---

## Why have an API Gateway

- Isolated external layer to your internal APIs
- Multichannel concerns (mobile, app, web) abstracted away
- Orchestrate, compose services to a single external api
- Security: api tokens, access control abstracted away 
- Rate limiting, circuit breaking 
- Allows different protocals and services to be aggregated

However, needs to be architectured (scaling/redundancy) to ensure you don't have a single point of failure or bottleneck. 

## Install

Assume docker is available and using postgres.


https://konghq.com/install/

```
docker network create kong-net
docker run -d --name kong-database \
               --network=kong-net \
               -p 5432:5432 \
               -e "POSTGRES_USER=kong" \
               -e "POSTGRES_DB=kong" \
               postgres:9.6
               
docker run --rm \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     kong:latest kong migrations up
     
     
docker run -d --name kong \
     --network=kong-net \
     -e "KONG_DATABASE=postgres" \
     -e "KONG_PG_HOST=kong-database" \
     -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
     -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
     -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
     -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
     -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
     -p 8000:8000 \
     -p 8443:8443 \
     -p 8001:8001 \
     -p 8444:8444 \
     kong:latest     
     
curl -i http://localhost:8001/   

HTTP/1.1 200 OK
Date: Fri, 21 Dec 2018 12:46:22 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Access-Control-Allow-Origin: *
Server: kong/0.14.1
Content-Length: 5659

{"plugins":{"enabled_in_cluster":[],"available_on_server":{"response-transformer":true,"oauth2":true,"acl":true,"correlation-id":true,"pre-function":true,"jwt":true,"cors":true,"ip-restriction":true,"basic-auth":true,"key-

```

Pretty response
```
{
  "plugins": {
    "enabled_in_cluster": [
      
    ],
    "available_on_server": {
      "response-transformer": true,
      "oauth2": true,
      "acl": true,
      "correlation-id": true,
      "pre-function": true,
      "jwt": true,
      "cors": true,
      "ip-restriction": true,
      "basic-auth": true,
      "key-auth": true,
      "rate-limiting": true,
      "request-transformer": true,
      "http-log": true,
      "file-log": true,
      "hmac-auth": true,
      "ldap-auth": true,
      "datadog": true,
      "tcp-log": true,
      "zipkin": true,
      "post-function": true,
      "request-size-limiting": true,
      "bot-detection": true,
      "syslog": true,
      "loggly": true,
      "azure-functions": true,
      "udp-log": true,
      "response-ratelimiting": true,
      "aws-lambda": true,
      "statsd": true,
      "prometheus": true,
      "request-termination": true
    }
  },
  "tagline": "Welcome to kong",
  "configuration": {
    "plugins": [
      "bundled"
    ],
    "admin_ssl_enabled": true,
    "lua_ssl_verify_depth": 1,
    "trusted_ips": {
      
    },
    "prefix": "\/usr\/local\/kong",
    "loaded_plugins": {
      "response-transformer": true,
      "request-termination": true,
      "prometheus": true,
      "ip-restriction": true,
      "pre-function": true,
      "jwt": true,
      "cors": true,
      "statsd": true,
      "basic-auth": true,
      "key-auth": true,
      "ldap-auth": true,
      "aws-lambda": true,
      "http-log": true,
      "response-ratelimiting": true,
      "hmac-auth": true,
      "request-size-limiting": true,
      "datadog": true,
      "tcp-log": true,
      "zipkin": true,
      "post-function": true,
      "bot-detection": true,
      "acl": true,
      "loggly": true,
      "syslog": true,
      "azure-functions": true,
      "udp-log": true,
      "file-log": true,
      "request-transformer": true,
      "correlation-id": true,
      "rate-limiting": true,
      "oauth2": true
    },
    "cassandra_username": "kong",
    "admin_ssl_cert_csr_default": "\/usr\/local\/kong\/ssl\/admin-kong-default.csr",
    "ssl_cert_key": "\/usr\/local\/kong\/ssl\/kong-default.key",
    "admin_ssl_cert_key": "\/usr\/local\/kong\/ssl\/admin-kong-default.key",
    "dns_resolver": {
      
    },
    "pg_user": "kong",
    "mem_cache_size": "128m",
    "cassandra_data_centers": [
      "dc1:2",
      "dc2:3"
    ],
    "nginx_admin_directives": {
      
    },
    "custom_plugins": {
      
    },
    "pg_host": "kong-database",
    "nginx_acc_logs": "\/usr\/local\/kong\/logs\/access.log",
    "proxy_listen": [
      "0.0.0.0:8000",
      "0.0.0.0:8443 ssl"
    ],
    "client_ssl_cert_default": "\/usr\/local\/kong\/ssl\/kong-default.crt",
    "ssl_cert_key_default": "\/usr\/local\/kong\/ssl\/kong-default.key",
    "dns_no_sync": false,
    "db_update_propagation": 0,
    "nginx_err_logs": "\/usr\/local\/kong\/logs\/error.log",
    "cassandra_port": 9042,
    "dns_order": [
      "LAST",
      "SRV",
      "A",
      "CNAME"
    ],
    "dns_error_ttl": 1,
    "headers": [
      "server_tokens",
      "latency_tokens"
    ],
    "dns_stale_ttl": 4,
    "nginx_optimizations": true,
    "database": "postgres",
    "pg_database": "kong",
    "nginx_worker_processes": "auto",
    "lua_package_cpath": "",
    "admin_acc_logs": "\/usr\/local\/kong\/logs\/admin_access.log",
    "lua_package_path": ".\/?.lua;.\/?\/init.lua;",
    "nginx_pid": "\/usr\/local\/kong\/pids\/nginx.pid",
    "upstream_keepalive": 60,
    "cassandra_contact_points": [
      "kong-database"
    ],
    "client_ssl_cert_csr_default": "\/usr\/local\/kong\/ssl\/kong-default.csr",
    "proxy_listeners": [
      {
        "ssl": false,
        "ip": "0.0.0.0",
        "proxy_protocol": false,
        "port": 8000,
        "http2": false,
        "listener": "0.0.0.0:8000"
      },
      {
        "ssl": true,
        "ip": "0.0.0.0",
        "proxy_protocol": false,
        "port": 8443,
        "http2": false,
        "listener": "0.0.0.0:8443 ssl"
      }
    ],
    "proxy_ssl_enabled": true,
    "admin_access_log": "\/dev\/stdout",
    "ssl_ciphers": "ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256",
    "enabled_headers": {
      "latency_tokens": true,
      "X-Kong-Proxy-Latency": true,
      "Via": true,
      "server_tokens": true,
      "Server": true,
      "X-Kong-Upstream-Latency": true,
      "X-Kong-Upstream-Status": false
    },
    "cassandra_ssl": false,
    "ssl_cert_csr_default": "\/usr\/local\/kong\/ssl\/kong-default.csr",
    "db_resurrect_ttl": 30,
    "client_max_body_size": "0",
    "cassandra_consistency": "ONE",
    "db_cache_ttl": 0,
    "admin_error_log": "\/dev\/stderr",
    "pg_ssl_verify": false,
    "dns_not_found_ttl": 30,
    "pg_ssl": false,
    "client_ssl": false,
    "db_update_frequency": 5,
    "cassandra_repl_strategy": "SimpleStrategy",
    "nginx_kong_conf": "\/usr\/local\/kong\/nginx-kong.conf",
    "cassandra_repl_factor": 1,
    "nginx_http_directives": [
      {
        "value": "prometheus_metrics 5m",
        "name": "lua_shared_dict"
      }
    ],
    "error_default_type": "text\/plain",
    "kong_env": "\/usr\/local\/kong\/.kong_env",
    "real_ip_header": "X-Real-IP",
    "dns_hostsfile": "\/etc\/hosts",
    "admin_listeners": [
      {
        "ssl": false,
        "ip": "0.0.0.0",
        "proxy_protocol": false,
        "port": 8001,
        "http2": false,
        "listener": "0.0.0.0:8001"
      },
      {
        "ssl": true,
        "ip": "0.0.0.0",
        "proxy_protocol": false,
        "port": 8444,
        "http2": false,
        "listener": "0.0.0.0:8444 ssl"
      }
    ],
    "admin_ssl_cert": "\/usr\/local\/kong\/ssl\/admin-kong-default.crt",
    "ssl_cert": "\/usr\/local\/kong\/ssl\/kong-default.crt",
    "proxy_access_log": "\/dev\/stdout",
    "admin_ssl_cert_key_default": "\/usr\/local\/kong\/ssl\/admin-kong-default.key",
    "cassandra_ssl_verify": false,
    "cassandra_lb_policy": "RoundRobin",
    "ssl_cipher_suite": "modern",
    "real_ip_recursive": "off",
    "proxy_error_log": "\/dev\/stderr",
    "client_ssl_cert_key_default": "\/usr\/local\/kong\/ssl\/kong-default.key",
    "nginx_daemon": "off",
    "anonymous_reports": true,
    "cassandra_timeout": 5000,
    "nginx_proxy_directives": {
      
    },
    "pg_port": 5432,
    "log_level": "notice",
    "client_body_buffer_size": "8k",
    "cassandra_schema_consensus_timeout": 10000,
    "lua_socket_pool_size": 30,
    "admin_ssl_cert_default": "\/usr\/local\/kong\/ssl\/admin-kong-default.crt",
    "cassandra_keyspace": "kong",
    "ssl_cert_default": "\/usr\/local\/kong\/ssl\/kong-default.crt",
    "nginx_conf": "\/usr\/local\/kong\/nginx.conf",
    "admin_listen": [
      "0.0.0.0:8001",
      "0.0.0.0:8444 ssl"
    ]
  },
  "version": "0.14.1",
  "node_id": "7a83aacb-4c6f-4e03-9310-a21555838d15",
  "lua_version": "LuaJIT 2.1.0-beta3",
  "prng_seeds": {
    "pid: 59": 191751351148,
    "pid: 63": 171484444272,
    "pid: 61": 463105229581,
    "pid: 58": 851872381411,
    "pid: 62": 947710712223,
    "pid: 60": 168217198722
  },
  "timers": {
    "pending": 5,
    "running": 0
  },
  "hostname": "00242784a3b3"
}
```
### Add a mock bin service api

Create service entry
```
curl -i -X POST \
  --url http://localhost:8001/services/ \
  --data 'name=example-service' \
  --data 'url=http://mockbin.org'
```

Add route
```
curl -i -X POST \
  --url http://localhost:8001/services/example-service/routes \
  --data 'hosts[]=example.com'
```

Check it

```
curl -i -X GET \
  --url http://localhost:8000/ \
  --header 'Host: example.com'
```

note the host header attribute.


Result

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 10750
Connection: keep-alive
Server: openresty/1.13.6.2
Date: Fri, 21 Dec 2018 12:52:56 GMT
Etag: W/"29fe-zRGDbSTAzeA2BElashPm2g"
Vary: Accept-Encoding
Via: kong/0.14.1
X-Kong-Upstream-Status: 200
X-Kong-Upstream-Latency: 240
X-Kong-Proxy-Latency: 480
Kong-Cloud-Request-ID: ea8f694a34ab729fd3da3b5196541d90

<!DOCTYPE html><html><head><meta charset="utf-8"><title>Mockbin by Mashape</title><meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no"><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.4/css/bootstrap.min.css" media="all"><link rel="stylesheet" type="text/css" href="https://fonts.googleapis.com/css?family=Open+Sans:400,600|Source+Code+Pro:200,300,400,500,600,700,900" media="all"><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.3.0/css/font-awesome.css" media="all"><link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/8.4/styles/github.min.css" media="all"><link rel="stylesheet" type="text/css" href="/static/main.css" media="all"><script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script><script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.4/js/bootstrap.min.js"></script><script 
```

### UI

The UI couldn't talk via the network (kong-net)

So
docker network inspect kong-net

get the ip of kong eg 172.18.0.3

```
docker run -d -p 1337:1337 \
             --network=kong-net \
             --name konga \
             pantsel/konga
                          
```
       
NOTE: when you first run up you will need to register an admin user make sure the password is at least 8 characters long.


Once logged in create a connection using the inner IP (172...) not localhost, as konga can't seem to talk to it. So use this Eg http://172.18.0.3:8001


You should see the following:

![Screenshot-2018-12-21-at-17.41.44](/content/images/2018/12/Screenshot-2018-12-21-at-17.41.44.png)


## Secure it

### With JWT

Can do this via the UI as well, but here lets use the API

#### Enable JWT on the example-service

curl -X POST http://localhost:8001/services/example-service/plugins \
    --data "name=jwt" 
```
{"created_at":1545417485000,"config":{"secret_is_base64":false,"key_claim_name":"iss","cookie_names":{},"maximum_expiration":0,"anonymous":"","run_on_preflight":true,"uri_param_names":["jwt"]},"id":"f843a5c5-cf15-4df8-accf-366e718cd169","enabled":true,"service_id":"d9de2b3b-825f-4155-bd27-a61e056bca51","name":"jwt"}
```
   
#### Enable it on the route as well

curl -X POST http://localhost:8001/routes/b2acc60b-a34e-4171-83b0-e39b5f67e026/plugins \
    --data "name=jwt" 
```
{"created_at":1545417612000,"config":{"secret_is_base64":false,"key_claim_name":"iss","cookie_names":{},"maximum_expiration":0,"anonymous":"","run_on_preflight":true,"uri_param_names":["jwt"]},"id":"645c1f20-dcfa-4816-bf04-7ad86750d0a9","enabled":true,"route_id":"b2acc60b-a34e-4171-83b0-e39b5f67e026","name":"jwt"}
```
    
#### Create a consumer

curl -X POST http://localhost:8001/consumers --data "username=mockbinuser" 
    
```
{"custom_id":null,"created_at":1545416569,"username":"mockbinuser","id":"15ff70f5-463f-4afc-84eb-87080d799287"}
```

#### Create the JWT Authentication

curl -X POST http://localhost:8001/consumers/mockbinuser/jwt -H "Content-Type: application/x-www-form-urlencoded"

```
{
  "created_at":1545416676000,
  "id":"6d925c30-788c-49cc-8427-21b42c8b37fd",
  "algorithm":"HS256",
  "key":"nCnMHTPpk9Out9KcyS2alvohWEx5uSoG",
  "secret":"YlDcaQ9NvjC8ewRN81X9ID4Bhjt8Z2Hq",
  "consumer_id":"15ff70f5-463f-4afc-84eb-87080d799287"
}
```

#### Craft the JW Token to use

{
    "typ": "JWT",
    "alg": "HS256"
}
{
    "iss": "nCnMHTPpk9Out9KcyS2alvohWEx5uSoG"
}

Process this at http://jwt.io and you'll get the token:

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuQ25NSFRQcGs5T3V0OUtjeVMyYWx2b2hXRXg1dVNvRyJ9.IHyWgccyv91XK6B62oiDyqor_IF9Lz0My2uHhQ0FYjw

#### Try it

curl -i http://localhost:8000 --header 'host:example-com'
```
HTTP/1.1 401 Unauthorized

```

curl -i http://localhost:8000?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuQ25NSFRQcGs5T3V0OUtjeVMyYWx2b2hXRXg1dVNvRyJ9.IHyWgccyv91XK6B62oiDyqor_IF9Lz0My2mple.com' --header 'host:example-com'

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 10750
Connection: keep-alive
Server: openresty/1.13.6.2
Date: Fri, 21 Dec 2018 18:47:50 GMT
Etag: W/"29fe-zRGDbSTAzeA2BElashPm2g"
Vary: Accept-Encoding
Via: kong/0.14.1
X-Kong-Upstream-Status: 200
X-Kong-Upstream-Latency: 238
X-Kong-Proxy-Latency: 63
Kong-Cloud-Request-ID: 73612bc3d69637ffab140cfb3e883da4

<!DOCTYPE html><html><head><meta....
```


Happy days.

### Routing

Via the konga API remove the host field and instead add the path /example-service


curl -i http://localhost:8000/example-service?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuQ25NSFRQcGs5T3V0OUtjeVMyYWx2b2hXRXg1dVNvRyJ9.IHyWgccyv91XK6B62oiDyqor_IF9Lz0My2uHhQ0FYjw

Kong re-directs via the route path to mockbin.org

And it's secured



### With RSA Token

openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -outform PEM -pubout -out public.pem

curl -X POST http://localhost:8001/consumers/mockbinuser/jwt -F "algorithm=RS256" -F "rsa_public_key=@public.pem" -F "key=ramjee"

```
{"rsa_public_key":"-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0cC1IIkGRaV3E+JE\/ffc\nGF0aWIiD\/AzciL1peoC1N4Z+weld9\/\/GQF5soTtl8q+ROtN7mOjeux7CaKLDXhDR\nYulEpXuXWEsbDBQmaw3d\/odJzXJ0m846fIXRfRnY8SS3gLo08LpetuWx7Z8xlMSP\nC\/eZtkiUsU9Ex7nRl992Ms5ZgX4FqAtvZYth4vwcIRlQhPnaTnnfBAX1E0au5nQH\nxxaPej7PKVxEiqyZmONeocXDB8LZevK\/LtlNR2ddJO5\/u6NbUax2cpPcs\/U16B9E\nuzVPr4orXmSHq98yXS6XJElCxxkIdzrpt5lK+9vYFxNjQWRwGhTx6400TFjwTL9t\nXQIDAQAB\n-----END PUBLIC KEY-----","created_at":1545423269000,"id":"66d13eeb-49cd-4424-8e83-d8f281f3bfdd","algorithm":"RS256","key":"ramjee","secret":"kMWbY1j9fP2P6yZ0t3Z1rLRfxhWUDL5O","consumer_id":"15ff70f5-463f-4afc-84eb-87080d799287"}```


In jwt.io

add public and private keys (from the pem files) and add {iss='ramjee'}

Should get the following token

```eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJyYW1qZWUifQ.lJfzJThuP1ZAquCv53fH_QuaXOkcdHTWlzz8pJqrghOLccTwq1tedptpcIt7zQwyOk0Q3U-9MCpwQ5qHRNoC5yg0oStQ8vJ9lpHTJCI1yxS0REop2BQQailnm4OrIBk0dWeVkmXyfNNPRoZu-a5ZmlbHeL2ZfHeQL5My-w-3bysZpVdTrRsVhm3ZrdZZxZUwJqep1Em-Je-4bNOLYr26rSp_76G-ddE0QTI94Ce14xml51OpUIOHNWW72huuhjcm24q-LQ96UBdOKudz7DxwZ44kh0M58WjV3G6aiU4iHcFJRI2JfE4OO-GFHYYsrXTcsCrOOmrCcCHyNOT7GLPYRQ
```

Try calling mockbin

curl -i http://localhost:8000/example-service?jwt=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJyYW1qZWUifQ.lJfzJThuP1ZAquCv53fH_QuaXOkcdHTWlzz8pJqrghOLccTwq1tedptpcIt7zQwyOk0Q3U-9MCpwQ5qHRNoC5yg0oStQ8vJ9lpHTJCI1yxS0REop2BQQailnm4OrIBk0dWeVkmXyfNNPRoZu-a5ZmlbHeL2ZfHeQL5My-w-3bysZpVdTrRsVhm3ZrdZZxZUwJqep1Em-Je-4bNOLYr26rSp_76G-ddE0QTI94Ce14xml51OpUIOHNWW72huuhjcm24q-LQ96UBdOKudz7DxwZ44kh0M58WjV3G6aiU4iHcFJRI2JfE4OO-GFHYYsrXTcsCrOOmrCcCHyNOT7GLPYRQ

Should be allowed through

Change the token to an invalid one, should get:

```
HTTP/1.1 403 Forbidden
Date: Fri, 21 Dec 2018 20:17:49 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Server: kong/0.14.1
Content-Length: 32

{"message":"Invalid signature"}
```


curl -X POST http://localhost:8001/consumers/mockbinuser/jwt --data "algorithm=RS256" --data-urlencode "rsa_public_key=[[
-----BEGIN CERTIFICATE-----
MIIDBTCCAe2gAwIBAgIQV68hSN9DrrlCaA3NJ0bnfDANBgkqhkiG9w0BAQsFADAt
MSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MB4X
DTE4MTExMTAwMDAwMFoXDTIwMTExMTAwMDAwMFowLTErMCkGA1UEAxMiYWNjb3Vu
dHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldDCCASIwDQYJKoZIhvcNAQEBBQAD
ggEPADCCAQoCggEBALvfCr6FB37Ns9mCcn5Cc2hhWDOfHg9HqR3xE08DQ5egC/3E
3zpJXMTOI6y1r1aqqdrB2h9IBaWD8qLzfv2pJhP+H5HNFcP8BjOYwz/o5zidbwb2
xaBe7gQMuK95Z9nstT6BlIaZF3Q2sISH3QG3O1i7cqKRzVkFyN9+q14sI73Iy/HR
4YnrwzpLbALIpQAz+cCU9Jck4nzjT2Tqvl1gsPRbVwEK+w54jgubg7lGi9JjNVCQ
oYgqw5hTgH+gjXbtksC4p12GrqjPTkRJSmBAoaBH4udX3LJpbJ+JrTT5MbLb0ezi
YiQab5OxS3omgbTJ7Ducd9Az4K4QGoK1Z9yGikUCAwEAAaMhMB8wHQYDVR0OBBYE
FGWLmYFSm5Exg9VcAGSg5sFE1mXgMA0GCSqGSIb3DQEBCwUAA4IBAQB0yTGzyhx+
Hz2vwBSo5xCkiIom6h7b946KKiWvgBLeOvAuxOsB15N+bbf51sUfUJ6jBaa1uJjJ
f27dxwH0oUe2fcmEN76QSrhULYe+k5yyJ7vtCnd6sHEfn9W6iRozv0cb48tESOTl
FuwbYDVW+YZxzM9EQHC32CjugURzuN9/rf6nJ9etSeckRMO8QPqyIi4e5sGSDYEx
xNs7J4prhIbtYT4NRiqc4nWzA/p5wSYOUgAZMSTLD/beSI81UN1Ao9VBBJu3v83d
62WL3zHSbpUwtG/utNhSi/n/7Q94claEWJVhBx6LiA1hrU6YZkjRGqBOrWIZkSkh
75xW6Xujocy4
-----END CERTIFICATE-----]]" --data "key=https://sts.windows.net/ec8969fa-9414-4afd-9440-2f368b73310d/"