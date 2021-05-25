# hive-idp

See https://dexidp.io/docs/kubernetes/

## Generate sel-signed cert

Dex needs its own cert, and k8s api-server needs to trust the CA.

```console
git clone git@github.com:dexidp/dex.git
cd dex/examples/k8s
vi gencert.sh
<change>
DNS.1 = idp.do.storageos.net

./gencert.sh
```

## Add CA cert to k8s control-plane CA trust store

Copy `ssl/ca.pem` to `/usr/share/ca-certificates/dex/dex-ca.crt` on each k8s control-plane node.

Run `dpkg-reconfigure ca-certificates` on each k8s control-plane node.

Ensure `/etc/ssl/certs/ca-certificates.crt` was updated:

```console
root@master-lon1-3:/etc/ssl/certs# ls -ltr /etc/ssl/certs/ | tail -3
lrwxrwxrwx 1 root root     41 May 25 21:50 dex-ca.pem -> /usr/share/ca-certificates/dex/dex-ca.crt
lrwxrwxrwx 1 root root     10 May 25 21:50 d43a9042.0 -> dex-ca.pem
-rw-r--r-- 1 root root 199647 May 25 21:50 ca-certificates.crt
```

## Create cluster secrets

```console
kubectl create secret tls idp.do.storageos.net.tls --cert=ssl/cert.pem --key=ssl/key.pem

kubectl create secret \
    generic github-client \
    --from-literal=client-id=$GITHUB_CLIENT_ID \
    --from-literal=client-secret=$GITHUB_CLIENT_SECRET
```
