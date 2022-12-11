# bigly-data-infra
Infra as code for the bigly-data project

### K8s infra

Make sure micro k8s is installed with:

`sudo snap install microk8s --classic`

The newer versions of the snap have deprecated and removed beta features so you may need to install an older snap version of microk8s

`sudo snap info micro` to get a list of versions

`sudo snap install microk8s --classic --channel=1.20/stable`

You may need to open firewall rules for pods to pod and pod to internet comms

`sudo ufw allow in on cni0 && sudo ufw allow out on cni0`
`sudo ufw default allow routed`

With the following addons installed:

- dns
- ha-cluster
- helm3
- hostpath-storage
- metallb 
- registry
- storage

To get a list of addons already installed use the command: `microk8s status`

So `microk8s enable dns hostpath-storage metallb registry storage community`

When installing metallb, it will ask for an IP range for it to route traffic to, 
give it a range that is covers the k8s node IPs in this case it's a single node. e.g. 192.168.1.81-192.168.1.81

Get microk8s config and install the config into your local kubectl config by copying and pasting the
contents of microk8s config to the .kube/config file

once updated make sure you set the current context on your local machine using the command `kubectl config use-context microk8s`

### Installing the App infra

To make services available to external users, the nginx ingress will take the hosts IP and forward requests based off the port to the correct service.
All helm charts will need to run in either `NodePort` or `ClusterIP` `service.type` mode

To run the helmfile, run the command `helmfile apply --no-color` (no color as helm diff has changed recently and does not accept the --color option from helm file
You may need to update helm repos with `helm repo update` if helm errors with it cannot find the release xyz

### Build infra

run the helmfile for build (jenkins)

then run `microk8s kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo` for the password

add a ssh key but ensure you select the drop down that has the most options, host, node, item etc etc
in the pod run `ssh -T git@github.com` to ensure git will clone the repo with the key

microk8s kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/additional/chart-admin-password && echo

setting up web hooks
paulhoang.asuscomm.com:7999/github-webhook/

The k8s template pod is the container thats going to be in charge with communicating with the main jenkins 
server and spinning up other containers for the pipeline, this by convention should always be the jnlp container

`your-svc.your-namespace.svc.cluster.local`
jenkins.build.svc.cluster.local
### App infra

The current kafka helm chart does not work with that latest microk8 image, its based on k8s 1.25 and its deprecated `PodDisruptionBudget` so you have to use
`apiVersion: policy/v1` instead of `apiVersion: policy/v1Beta`, once they fix that, go back to using the standard
chart rather than the customised version by updating the helmfile to use the release `kafka` instead of `custom-kafka`

if you need to destroy the kafka infra, remember to delete the PVC's too as they don't get removed with the chart destroy,
otherwise, when you apply the chart again it will use the same volume and won't apply your changes