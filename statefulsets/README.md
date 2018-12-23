### Use a StatefulSet

Create a headless governing service that will be used to provide the network identity for stateful pods:
```
kubectl create -f kubia-service-headless.yaml
```
Create the StatefulSet:
```
kubectl create -f kubia-statefulset.yaml
```
Play with teh pods: Open a separate terminal, run
```
kubectl proxy
```
Get back to the previous terminal, run
```
curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
```
> Communicate with the pod through the API server. The last slash / is necessary
```
curl -X POST -d "Hi there! This greeting was submitted to kubia-0." localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
```
Delete a stateful pod
```
kubectl delete po kubia-0
```
After a while, check if the data is till there:
```
curl localhost:8001/api/v1/namespaces/default/pods/kubia-0/proxy/
```
Expose stateful pods through a regular, non-headless service:
```
kubectl create -f kubia-service-public.yaml
```
Connect to cluster-internal services through the API server:
```
curl localhost:8001/api/v1/namespaces/default/services/kubia-public/proxy/
```
You may hit kubia-0 or kubia-1 randomly.

### Discover peers in a StatefulSet

DNS SRV records:
```
kubectl run -it srvlookup --image=tutum/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local
```
You will see on the screen
```
; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> SRV kubia.default.svc.cluster.local
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62957
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;kubia.default.svc.cluster.local. IN	SRV

;; ANSWER SECTION:
kubia.default.svc.cluster.local. 5 IN	SRV	0 50 80 kubia-0.kubia.default.svc.cluster.local.
kubia.default.svc.cluster.local. 5 IN	SRV	0 50 80 kubia-1.kubia.default.svc.cluster.local.

;; ADDITIONAL SECTION:
kubia-0.kubia.default.svc.cluster.local. 5 IN A	172.17.0.5
kubia-1.kubia.default.svc.cluster.local. 5 IN A	172.17.0.6

;; Query time: 6 msec
;; SERVER: 10.96.0.10#53(10.96.0.10)
;; WHEN: Sun Dec 23 03:43:46 UTC 2018
;; MSG SIZE  rcvd: 350

pod "srvlookup" deleted
```

Update a StatefulSet
```
kubectl edit statefulset kubia
```
Change ```spec.replicas``` to 3, and modify rhe ```spec.template.spec.containers.image``` attribute (```luksa/kubia-pet-peers``` instead of ```luksa/kubia-pet```)

Forcibly delete a pod
```
kubectl delete po kubia-0 --force --grace-period 0
```
