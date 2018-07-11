# Jenkins on GKE

### Create cluser and deploy Jenkins

The script that was used to create cluster and deploy jenkins

```bash
KUBE_VERSION=1.10.2-gke.3
SIZE=4
MACHINE_TYPE=g1-small
NAME=kubelab

cls() {
  echo "Creating cluster"
  gcloud container clusters create $NAME \
    --cluster-version=$KUBE_VERSION \
    --enable-network-policy \
    --machine-type=$MACHINE_TYPE \
    --preemptible \
    --enable-autorepair \
    --enable-autoupgrade \
    --num-nodes=$SIZE \
    --disk-size=10

  sleep 1
  echo "Getting creadentials for brand new cluster"
  gcloud container clusters get-credentials $NAME
}

hlm() {
  echo "Installing helm"
  kubectl create -f helm_rbac.yaml
  sleep 1
  helm init --service-account=tiller
}

jks() {
  echo "Installing jenkins"
  helm install --name jenkins stable/jenkins --set Master.ServicePort=80
}

hlp() {
 printf "
 Usage: $0 ACTION

 ACTION may be:
   cls - create GKE cluster
   hlm - install helm to existing cluster
   jks - install jenkins helm chart
"
}

case "$1" in
  cls)
    cls
    exit
    ;;
  hlm)
    hlm
    exit
    ;;
  jks)
    jks
    exit
    ;;
  *)
    hlp
    exit 1
    ;;
esac
    
```

### Files that were used to configure RBAC

- helm_rbac.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: tiller-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
```

- jenkins_rbac.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: jenkins
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: jenkins-clusterrolebinding
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: jenkins
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: ""
```
