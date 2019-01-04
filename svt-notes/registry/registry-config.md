# Registry configuration
## AWS
```yaml
log:
  level: info
http:
  addr: :5000
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  s3:
    accesskey: <key>
    secretkey: <secret>
    region: us-west-2
    bucket: aoe-svt-test
    encrypt: False
    secure: true
    v4auth: true
    rootdirectory: /registry
    chunksize: "26214400"
auth:
  openshift:
    realm: openshift
middleware:
  registry:
  - name: openshift
  repository:
  - name: openshift
    options:
      pullthrough: True
      acceptschema2: True
      enforcequota: False
  storage:
  - name: openshift
```

## Azure
```yaml
auth:
  openshift:
    realm: openshift
http:
  addr: :5000
log:
  level: debug
middleware:
  registry:
  - name: openshift
  repository:
  - name: openshift
    options:
      acceptschema2: true
      enforcequota: false
      pullthrough: true
  storage:
  - name: openshift
storage:
  azure:
    accountkey: az3kbKxF4mmRmE9tI9i4D1f+kGs9Q7Cs8kHRrTasSp5ks+DYCyHrESrxkDiw06PKZhj0eOgTGmTfWcUv7epSqA==
    accountname: t84oqs7q9ujtfneqvwlasxju
    container: registry
    realm: core.windows.net
  cache:
    blobdescriptor: inmemory
  delete:
    enabled: true
version: 0.1
```
## PVC mounted on /registry
[Hongkai's excellent guide](https://github.com/hongkailiu/svt-case-doc/blob/master/learn/docker_registry.md#use-filesystem-driver-for-docker-registry)
```yaml
version: 0.1
log:
  level: info
http:
  addr: :5000
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /registry
  delete:
    enabled: true
auth:
  openshift: {}
openshift:
  version: 1.0
  auth:
    realm: openshift
    # tokenrealm is a base URL to use for the token-granting registry endpoint.
    # If unspecified, the scheme and host for the token redirect are determined from the incoming request.
    # If specified, a scheme and host must be chosen that all registry clients can resolve and access:
    #
    # tokenrealm: https://example.com:5000
  audit:
    enabled: false
  metrics:
    enabled: false
    # secret is used to authenticate to metrics endpoint. It cannot be empty.
    # Attention! A weak secret can lead to the leakage of private data.
    #
    # secret: TopSecretLongToken
  requests:
    # GET and HEAD requests
    read:
      # maxrunning is a limit for the number of in-flight requests. A zero value means there is no limit.
      maxrunning: 0
      # maxinqueue sets the maximum number of requests that can be queued if the limit for the number of in-flight requests is reached.
      maxinqueue: 0
      # maxwaitinqueue is how long a request can wait in the queue. A zero value means it can wait forever.
      maxwaitinqueue: 0
    # PUT, PATCH, POST, DELETE requests and internal mirroring requests
    write:
      # See description of openshift.requests.read.
      maxrunning: 0
      maxinqueue: 0
      maxwaitinqueue: 0
  quota:
    enabled: false
    cachettl: 1m
  cache:
    blobrepositoryttl: 10m
  pullthrough:
    enabled: true
    mirror: true
  compatibility:
    acceptschema2: true
```