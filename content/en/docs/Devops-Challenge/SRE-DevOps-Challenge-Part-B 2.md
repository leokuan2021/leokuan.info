---
title: "SRE DevOps Challenge Part B"
date: 2022-03-06T14:19:02-10:00
draft: true
weight: 20
type: docs

TableOfContents: true
---

## Deploying Prometheus and Grafana to k8s cluster

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack


the values file is fairly long at 2000+ lines. Working in the terminal, I would save a copy of the default and do a colordiff/vimdiff against the edited version to make sure I am making all the intended edits correctly before I install the helm chart with my custom values. 

```console
colordiff -yW"`tput cols`" values.yaml prometheus-grafana-values.yaml | less -R
```
```console
colordiff -yW"`tput cols`" values.yaml prometheus-grafana-values.yaml | less -R
```

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
`helm repo update`

`helm show values prometheus-community/kube-prometheus-stack > prometheus-grafana-values.yaml`


`helm install --namespace=monitoring prometheus prometheus-community/kube-prometheus-stack --values prometheus-grafana-values.yaml`


here we are making changes to the grafana configuration to 
  <pre>
grafana:
  enabled: true
    <b>
  ## Grafana's primary configuration
  ## NOTE: values in map will be converted to ini format
  ## ref: http://docs.grafana.org/installation/configuration/
  ##
  grafana.ini:
    paths:
      data: /var/lib/grafana/data
      logs: /var/log/grafana
      plugins: /var/lib/grafana/plugins
      provisioning: /etc/grafana/provisioning
    analytics:
      check_for_updates: true
    log:
      mode: console
    grafana_net:
      url: https://grafana.net
    server:
      domain: grafana.leokuan.info
      root_url: "https://grafana.leokuan.info"
      </b>
  </pre>
  
  
  namespaceOverride: ""

    ## Enable persistence using Persistent Volume Claims
  ## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  ##
  persistence:
    type: pvc
    enabled: true
    storageClassName: longhorn
    accessModes:
      - ReadWriteOnce
    size: 10Gi
    # annotations: {}
    finalizers:
      - kubernetes.io/pvc-protection
    # subPath: ""
    # existingClaim:





## Adding a Prometheus metrics endpoint to webhook-app

## Configuring Prometheus to scrape using ServiceMonitor

## Creating a provisioned Grafana dashboard

## Deploying a custom metrics adaptor for Prometheus

## Adding an HPA manifest to webhook-app


For testing, after we have the Grafana dashboard set up to watch the webhook counts and inflight counts, I suppose you could use something like POSTMAN, but
a simple way is using `curl`, to generate some POST requests to test the metrics we can do

```
curl -X POST http://www.example.com:port 
``` 
to keep spamming one could also use watch -n 0.1 curl -X POST http://www.example.com:port 

but again, the minimum interval for watch is only 0.1s, 
if you really want to hit it, this is one way to do it.
xargs -I % -P 4 curl -X POST http://localhost:3000 < <(printf '%s\n' {1..5000})

for some reason though, on my Mac when I run this command with the number of processes (-P) set to 10 or higher I always get the following
curl: (7) Couldn't connect to server

It works just fine on my Ubuntu laptop though, but that's another rabbit hole for another time.

Next thing we need for testing the inflight count is to introduce some latency since I am doing this on a local cluster and the delay is less than 1ms. It would take a lot of packets to get some requests queued up in our app's pods and something else could break or skew our experiment. Remember we are not testing the local network or trying to see how many requests a single nodejs instance can handle here. 

To introduce some artificial latency to slow things down, we can use tc in linux. This really brings back memory, the last time I used tc was about 10 years ago. 

tc qdisc add dev eth0 root netem delay 100ms

assuming eth0 is your network interface, and 100ms is the latency you want to add.
you can ping another host on your network before and after applying the command to verify.
tc -s qdisc
run this command to see 
 qdisc netem 8002: dev eth0 root refcnt 2 limit 1000 delay 97.0ms

when you're done, you can delete the rule
 tc qdisc del dev eth0 root netem

try playing with the parameters and see get your HPA to scale as expected.


{{< gist username gistId >}}

## Code highlight with clipboard

kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80


apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80




```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```