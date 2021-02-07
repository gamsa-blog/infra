# DEVELOPMENT

**kubernetes**
tools:
- multipass - VMs
- k3sup       - cluster bootsrapping & admin
- k3s         - thin version of k8s
- kubectl     - kubernetes client

```
# if VMs don't exist: launch / create VMs
multipass launch --cpus 1 --mem 1G --disk 5G --name master-node --cloud-init multipass.yaml
multipass launch --cpus 1 --mem 1G --disk 5G --name agent-master --cloud-init multipass.yaml
multipass launch --cpus 1 --mem 1G --disk 5G --name agent-worker --cloud-init multipass.yaml

multipass ls

# if VMs exist but stopped: start VMs
mp start --all

export NODE_MASTER=$( mp ls | grep master-node | awk '{print $3}' )
export AGENT_MASTER=$( mp ls | grep agent-master | awk '{print $3}' )
export AGENT_WORKER=$( mp ls | grep agent-worker | awk '{print $3}' )

# bootstrapping cluster
k3sup install --ip $NODE_MASTER --user ubuntu --k3s-extra-args "--cluster-init"
k3sup join --ip $AGENT_MASTER --user ubuntu --server-ip $NODE_MASTER --server-user ubuntu --server
k3sup join --ip $AGENT_WORKER --user ubuntu --server-ip $NODE_MASTER --server-user ubuntu

export KUBECONFIG=`pwd`/kubeconfig
kubectl get node

# initialize k8s objects
pushd /home/james/lab/kub-lab/k8s-intro-meetup-kit/k8s

k apply -f flask-deployment.yaml
k apply -f flask-service.yaml

popd

# run the following to get external-ip and port:
export INGRESS_IP=$(kubectl get svc -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$( kg svc -o jsonpath='{.items[0].spec.ports[0].port}' )


kg svc -o jsonpath='{items[0]}'

curl $INGRESS_IP:$INGRESS_PORT
```

