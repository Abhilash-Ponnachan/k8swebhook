# Steps to setup Kind Kubernetes Cluster for Ingress

## Kind Config for Ing Enabled Cluster

```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

## Install NGINX Ingress

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Check if `ingress-nginx-controller` is up and running in the `ingress-nginx` namespace.

```bash
kubectl get all -n ingress-nginx
```

## Load Local Docker images to Kind

```bash
kind load docker-image demowebapp revproxy authsimple
```

Now Kind cluster is ready for applying the manifests

Note: on local machine modify `/etc/hosts` file:
```txt
127.0.0.1       demowebapp.com
127.0.0.1       authsimple.com
```

We can then use the URL `demowebapp.com` to access the app, it will get routed to `localhost` with `Host: demowebapp.com` and the Ingress will route it to the `svc-appwithproxy`.

Similarly traffic to `authsimple.com` will get routed to `localhost` with `Host: authsimple.com` and the Ingress will route it to the `svc-authsimple`.