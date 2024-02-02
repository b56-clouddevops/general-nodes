# general-nodes

1) Create EC2 and run ansible playbook to configure rabbitmq
2) Create R53 Records for all the DB's
3) Configure Mutable App


### VPC ---> DB's ---> ALB's ---> Listeners


### What's next ?

1) Provision Infra.


Application Infrastructure Types : 
```
    1) Mutable Infra       : Infra remains same, while the application is going to be updated from one verison to the other ( Underlying infra remains same )
    2) Immuutable Infra    : 

```

no of subnets = 2  ( us-east-1a , us-east-1b ) 
if the count = 5      


### In Production, when you're doing business, what are all needed ????

1) We need to have Benchmarking !!!

    Benchmarking is a process of performing tests on the infrastructure and understanding the needed values and configuring your applicaiton infrastructure resources accordinly. 

2) Performance Testing On Your Application.

3) We need to have nice dashboards to understand what's going on in the applcaiton.


# Observability 
### As per Google SRE Book, any distributed applcations should have the below 4 metrics as apart of your obserbability platform.

#### The Four Golden Signals
```
    1) Traffic         ( logs )
    2) Latency         ( logs )
    3) Errors          ( logs )
    4) Saturation      ( System )
```


### ELK 

When are we going to send the logs to LOGSTASH ?

```
    1) If the logs are structured, then we need to sent those logs to Elastic only after the converison from Unstructured to structued. ( Logstansh does that job )

    2) If the logs are unstructured, then we can directly send the logs to ELASTIC ( most optimized way )
```

### ELK Ports 

```
    1) Elastic DB Port : 9200 
    2) Logstash        : 5044

```

### Tomorrow 
1) Install beats manually
2) Automate beats installation on servers 
3) Beats sending the logs to logstash
4) Logstash sending the logs to elastic





# Our Cart is Having Challenges ( Not able to add to cart )



### Goal 
1) Automate Beats Installation As a part of the Ansible CM  ( I will place it before you join )
2) Automate rsyslog configuration ( This will send the logs of the applications to a specific file of our choice )
3) Name of the log in the Kibana Dashboard should be based on the component.
4) Health check logs should be removed ( elastic should not have them )
5) Convert the logPattern of frontend 
6) Convert unstructured logs to structued.


### In our project :
    1) Cart, Catalogue, User Offers Structured Logs 
    2) Frontend , Shipping, Payment Offeres Unstructured logs



### Let's understand the log pattern of frontend : 

```
10.0.0.159 - - [27/Jan/2024:00:40:54 +0000] "GET / HTTP/1.1" 200 2542 "-" "Mozilla/5.0 (compatible; CensysInspect/1.1; +https://about.censys.io/)" "199.45.154.17"
```

### After Transforming the logPatter : 
27/Jan/2024:01:12:18 +0000 10.0.0.226 GET /users/sign_in HTTP/1.1 404 153 0.00

'$time_local $remote_addr $request $status $body_bytes_sent $request_time' ; 


### How to convert Unstructred Logs To Structued ???/
```
    Grok Debugger : This tool can convert the unstructured logs to structued.
```

### Grok Pattern for the above log :

```
    %{HTTPDATE:log_timestamp}%{SPACE}%{IP:source_ip}%{SPACE}%{WORD:method}%{SPACE}%{PATH:http_path}%{SPACE}%{WORD}/%{NUMBER}%{SPACE}%{NUMBER:http_status:int}%{SPACE}%{NUMBER:no_of_bytes_sent:int}%{SPACE}%{NUMBER:response_time:float}
```



We have Traffic, Latency in Kibana and we will start with errors tomorrow.



### Error :
2024-01-29 00:40:49.359 ERROR 1741 --- [io-8080-exec-11] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Handler dispatch failed; nested exception is java.lang.OutOfMemoryError: Java heap space] with root cause


### We don't have System Resource Level Metrics Yet
```
    1) What is the CPU Utilization ?
    2) What is the Memory Utilizaiton ?
    3) Status of the service ?
    4) Server Status

```


### Monitoring Tools and Types : 

    1) System Monitoring                    ( Prometheus )           
    2) Network or Packet Based Monitoring   ( Solarwinds )
    3) Application Monitorings              ( Using Logs : ELK )
    4) Application Performance Monitoring   ( Artificial Intelligence : Dynatrace , Datadog & NewRelic )



### Monitoring Variants : 
    1) Blackbox Monitoring
    2) Whilebox Monitoring


### Prometheus : 

```
    1) Create a t3.medium server with 20gb disk
    2) Install Prometheus and Grafana on the top of the server.
    3) Manually Install Exporter On the top of any server and see the whether metrics are coming up or not 
    4) Just like how we have automated the BEATS instllation, we also need to automate the EXPORTER's instllation.

```


### Prerequisites For Prometheus ?

```
    1) Exporters to be installed on the infra to be monitored 
    2) Network should be enabled between the Prometheus server and the node infra ( prometheus should be able to talk to node on 9100 port )
```



# Exporters :

```
    1) Exporters are offering the metrics.
    2) How prometheus comes to know that it has to go and fetch the metrcis from which node_exporter
    3) You might have 100 servers, but I would like my prometheus to monitor only the nodes of my choice.
```


$ If you've 100's of server's from different projects and would like to monitor only a specific project VM's, how can you let prometheus know to monitor only a specific vm's ???

```
    We have 2 options to do this :

        1) We can specify a specific network range and let prometheus monitor them.
        2) Since all the machines are on AWS, we can give IAM Role / Authority to Prometheus and prometheus can get the information based on the TAG's and use the "Service Discovery" Mechanism based on the cloud you're using.
```


### Prometheus Functions : 

```
    Ceil means upper value, 0.5 ceil is 1
    Floor means lower value, 0.5 ceil is 0  

    Reference : https://prometheus.io/docs/prometheus/latest/querying/functions
```


### One instance can have more than one CPU!!!!


# PQL For CPU Utilization :

```
    ceil(100 - (avg by(instance_name) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100))
```

# PQL For Memory Utilization :

```
    ((node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes) * 100 
```


### Infrastructure Types :

1) Mutable Infrastructure           ( On the same infra : we are going upgrade / update the applications from one verison to other version)

2) Immutable Infrastructure         ( Everytime, we upgrade/update we do them on the top of new hardware : Here the AMI is the artifact )

```
         CI ---> cart-0001.zip ---> Create a VM ---> Deploy cart-0001.zip ---> Take a AMI ( cart-0001 ) ----> Release ---> Create Infra using the AMI version of your choice ----> Test the version ----> Switch the traffic to the New One ( Blue Green Deployment )
```


3) Container Infrastrucure



### Deployment Types : 

```
    1) Rolling Update
    2) Blue Green Update
    3) Canary Deployment
```
