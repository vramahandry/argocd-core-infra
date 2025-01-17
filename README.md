# argocd-core-infra
Argocd apps/appsets for team core infra
Structure copy from this blog [repo](https://github.com/kostis-codefresh/many-appsets-demo) and this [article](https://codefresh.io/blog/how-to-structure-your-argo-cd-repositories-using-application-sets/) 

## cert-manager
### Create your own CA certificate with cert-manager himself (recommended)
You can check the file app/cert-manager/templates/root-ca.yaml<br>
You can check also the all workflow when create a certificate with this [link](https://cert-manager.io/docs/usage/certificaterequest/#inner-workings-diagram-for-developers)

#### Get the rootCA.crt from cert-manager

```bash
kubectl get -n cert-manager secret root-secret -o
jsonpath='{.data.ca\.crt}' | base64 -d > rootCA.crt
```
#### Add the root CA in trustore for Mac

```bash
sudo security add-trusted-cert -d -r trustRoot -k
 /Library/Keychains/System.keychain rootCA.crt
```

### Create your own CA certficate with mkcert

```shell
CAROOT=<PATH> mkcert -install -ecdsa
```
You can check the certificate
```shell
openssl x509 -in $CAROOT/rootCA.crt -text -noout
```

#### Create a secret ca-key-pair for cert-manager
```shell
kubectl create secret generic ca-key-pair \    
--from-file=tls.crt=rootCA.pem \
--from-file=tls.key=rootCA-key.pem \
--namespace=cert-manager
```
## Ingresses

### Setup dnsmasq

Mac
https://kharysharpe.com/blog/automatic-local-domains-dnsmasq-for-macos/

Linux
https://medium.com/@charled.breteche/using-dnsmasq-with-a-local-kind-clusters-9a27c8987073


### Nginx ingress controller
```bash
LB_IP=$(kubectl get services \
   --namespace ingress-controller \
   ingress-controller-nginx-controller \
   --output jsonpath='{.status.loadBalancer.ingress[0].ip}')
```
### Change dns resolution to use cloud-provider-kind with dnsmasq
#### With brew
```bash
brew install dnsmasq
```
Change the dnsmasq config in `$(brew -prefix)/etc/dnsmasq.conf`<br>
Uncomment `conf-dir=/opt/homebrew/etc/dnsmasq.d/,*.conf`in the end of file

Add your domain name like this
```bash
echo "address=/kind.cluster/<IP_CLOUD_PROVIDER_KIND>" | sudo tee $(brew --prefix)/etc/dnsmasq.d/kind.k8s.conf
```
Start/Restart the service dnsmasq
```bash
sudo brew services restart dnsmasq
```
Check the status
```bash
sudo brew services list | grep dnsmasq
```
Add dns resolver for your domain
```bash
sudo mkdir -v /etc/resolver

sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/kind.cluster'
```

Now you can access to `*.kind.cluster` domains
