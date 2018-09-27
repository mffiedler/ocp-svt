# Elastic Search

## Heap

* [Don't configure heap above 32GB](https://www.elastic.co/guide/en/elasticsearch/guide/current/heap-sizing.html)

As long as the Java heap size stays below 32GB the JVM will use compressed ordinary pointers which are much faster than pointers used to address heap > 64GB.   In OpenShift, the max you should ever set the memory limit and INSTANCE_RAM to is 63Gi.

Look for this message in the ES log:
```sh
## from oc exec -c elasticsearch $POD -- grep "compressed ordinary object pointers" /elasticsearch/persistent/logging-es/logs/logging-es.log 
[2018-09-27T19:11:37,190][INFO ][o.e.e.NodeEnvironment    ] [logging-es-data-master-4bfyenbn] heap size [31.3gb], compressed ordinary object pointers [true] 
```
You can also curl JVM info with 
```sh
 # oc exec -n openshift-logging -c elasticsearch $POD -- curl --connect-timeout 2 -s -k --cert /etc/elasticsearch/secret/admin-cert --key /etc/elasticsearch/secret/admin-key https://logging-es:9200/_nodes/logging-es-data-master-4bfyenbn/jvm?pretty           
{                                                                                                                                                                                                                                                                                         
  "_nodes" : {                                                                                                                                                                                                                                                                            
    "total" : 1,                                                                                                                                                                                                                                                                          
    "successful" : 1,                                                                                                                                                                                                                                                                     
    "failed" : 0                                                                                                                                                                                                                                                                          
  },                                                                                                                                                                                                                                                                                      
  "cluster_name" : "logging-es",                                                                                                                                                                                                                                                          
  "nodes" : {                                                                                                                                                                                                                                                                             
    "g71cSL_pTfejAZ5WrDKZ0g" : {                                                                                                                                                                                                                                                          
      "name" : "logging-es-data-master-4bfyenbn",                                                                                                                                                                                                                                         
      "transport_address" : "172.23.2.34:9300",                                                                                                                                                                                                                                           
      "host" : "172.23.2.34",
      "ip" : "172.23.2.34",
      "version" : "5.6.10",
      "build_hash" : "b489379",
      "roles" : [
        "master",
        "data",
        "ingest"
      ],
      "jvm" : {
        "pid" : 1,
        "version" : "1.8.0_181",
        "vm_name" : "OpenJDK 64-Bit Server VM",
        "vm_version" : "25.181-b13",
        "vm_vendor" : "Oracle Corporation",
        "start_time_in_millis" : 1538073265126,
        "mem" : {
          "heap_init_in_bytes" : 8589934592,
          "heap_max_in_bytes" : 8476557312,
          "non_heap_init_in_bytes" : 2555904,
          "non_heap_max_in_bytes" : 0,
          "direct_max_in_bytes" : 8476557312
        },
        "gc_collectors" : [
          "ParNew",
          "ConcurrentMarkSweep"
        ],
        "memory_pools" : [
          "Code Cache",
          "Metaspace",
          "Compressed Class Space",
          "Par Eden Space",
          "Par Survivor Space",
          "CMS Old Gen"
        ],
        "using_compressed_ordinary_object_pointers" : "true",
        "input_arguments" : [
          "-XX:+UseConcMarkSweepGC",
          "-XX:CMSInitiatingOccupancyFraction=75",
          "-XX:+UseCMSInitiatingOccupancyOnly",
          "-XX:+AlwaysPreTouch",
          "-Xss1m",
          "-Djava.awt.headless=true",
          "-Dfile.encoding=UTF-8",
          "-Djna.nosys=true",
          "-Djdk.io.permissionsUseCanonicalPath=true",
          "-Dio.netty.noUnsafe=true",
          "-Dio.netty.noKeySetOptimization=true",
          "-Dio.netty.recycler.maxCapacityPerThread=0",
          "-Dlog4j.shutdownHookEnabled=false",
          "-Dlog4j2.disable.jmx=true",
          "-Dlog4j.skipJansi=true",
          "-XX:+HeapDumpOnOutOfMemoryError",
          "-XX:+UnlockExperimentalVMOptions",
          "-XX:+UseCGroupMemoryLimitForHeap",
          "-XX:MaxRAMFraction=2",
          "-XX:InitialRAMFraction=2",
          "-XX:MinRAMFraction=2",
          "-Dmapper.allow_dots_in_name=true",
          "-Xms8192m",
          "-Xmx8192m",
          "-XX:HeapDumpPath=/elasticsearch/persistent/heapdump.hprof",
          "-Dsg.display_lic_none=false",
          "-Des.path.home=/usr/share/elasticsearch"
        ]
      }
    }
  }
}
```