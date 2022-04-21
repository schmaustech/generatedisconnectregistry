# generatedisconnectregistry

~~~bash
export GODEBUG="x509ignoreCN=0"
export VERSION="4.10.3"
export OCP_RELEASE="4.10.3"
export ARCHITECTURE="x86_64"
export UPSTREAM_REPO='openshift-release-dev'
export OCP_ARCH="x86_64"
export RELEASE_NAME="ocp-release"
export OCP_RELEASE="4.10.3"
export PATH=$PATH:/home/bschmaus
export CMD=openshift-baremetal-install
export EXTRACT_DIR=$(pwd)
export PULLSECRET=/home/bschmaus/pull-secret.json

curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$VERSION/openshift-client-linux-$VERSION.tar.gz | tar zxvf - oc
sudo cp ~/oc /usr/local/bin/oc
export RELEASE_IMAGE=$(curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$VERSION/release.txt| grep 'Pull From: quay.io' | awk -F ' ' '{print $3}' | xargs)
oc adm release extract --registry-config "${PULLSECRET}" --command=$CMD --to "${EXTRACT_DIR}" ${RELEASE_IMAGE}


sudo yum -y install podman httpd httpd-tools
sudo mkdir -p /opt/registry/{auth,certs,data}
sudo openssl req -newkey rsa:4096 -nodes -sha256 -keyout /opt/registry/certs/domain.key -x509 -days 365 -out /opt/registry/certs/domain.crt -subj "/C=US/ST=NorthCarolina/L=Raleigh/O=Red Hat/OU=Marketing/CN=provi
sioning.schmaustech.com" -addext "subjectAltName = DNS:provisioning.schmaustech.com" -addext "certificatePolicies = 1.2.3.4"
sudo cp /opt/registry/certs/domain.crt /home/bschmaus/domain.crt
sudo chown bschmaus:bschmaus /home/bschmaus/domain.crt
sudo cp /opt/registry/certs/domain.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract
sudo htpasswd -bBc /opt/registry/auth/htpasswd dummy dummy
sudo podman create --name poc-registry --net host -p 5000:5000 -v /opt/registry/data:/var/lib/registry:z -v /opt/registry/auth:/auth:z -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry" -e "R
EGISTRY_HTTP_SECRET=ALongRandomSecretForRegistry" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v /opt/registry/certs:/certs:z -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/
domain.key docker.io/library/registry:2
sudo podman start poc-registry
sudo podman ps
sleep 10


export PULLSECRET=/home/bschmaus/pull-secret.json
export LOCAL_REG='provisioning.schmaustech.com:5000'
export LOCAL_REPO='ocp4/openshift4'
oc adm release mirror -a $PULLSECRET --from=quay.io/${UPSTREAM_REPO}/${RELEASE_NAME}:${OCP_RELEASE}-${ARCHITECTURE} --to-release-image=$LOCAL_REG/$LOCAL_REPO:$VERSION --to=$LOCAL_REG/$LOCAL_REPO

~~~
