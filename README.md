# 🚢 ⛵ Harbor Demo on Minikube

_Why Harbor?_

* container proxy cache (fewer errors from DockerHub)
* store container images
* scan container images
* sign container images
* store helm charts
* distribute containers and charts
* RBAC, quotas, push rules, intuitive UI, full API
* *potentially not allow connections to Internet for containers and charts*

Use these instructions if you want to demo Harbor from a Helm chart on OSX using minikube.

## Dependencies

The minikube this was tested on uses hyperkit as the VM `minikube start --driver=hyperkit`

minkube addons:

* ingress

## 🛹 Installing Harbor using Helm

1. add the helm chart
    `helm repo add harbor https://helm.goharbor.io`
1. a values.yaml file was pulled from the [Harbor Git repo](https://github.com/goharbor/harbor) which can be used as a good base. There were a few edits to the _values.yaml_ mainly disabling notary.
1. install the chart
    `helm install harbor harbor/harbor -f values.yaml`
1. watch the action...
    `helm status harbor`
    or
    `kubectl get pods -w`
1. create */etc/hosts* entry for harbor.
    `192.168.64.2   core.harbor.domain`
1. once the harbor pods are running, you are able to log in using `admin` and `Harbor12345` using you favorite browser.
1. explore the UI, you're admin so you have complete control.
1. Now that you're logged in, download the `ca.crt` from harbor by selecting _projects-->library-->Registry Certificate_. This will download a _ca.crt_ file which you will add to your system certs keychain.
1. configure _Automatically scan images on push_ by choosing the _Configuration_ tab and toggle the checkbox.
1. restart *docker*

## 💿 create, tag, and upload `docker` image

1. find some _Dockerfile_ and built it fresh (you can also use an existing built docker image from your local repo).
    `docker build -t footest .`
1. tag the image
    `docker tag footest:latest core.harbor.domain/library/footest:latest`
1. log in to harbor
    `docker login core.harbor.domain/library`
    use _admin_ as the user and _Harbor12345_ as the password
1. push the image
    `docker push core.harbor.domain/library/footest:latest
1. view the image in _harbor_. Select the _Respositories_ tab and you should see an entry for image you just uploaded *library/footest*. Click on that link.
1. This will take you into the Artifacts for that tag where you will see a *SHA256 link* of the image you just uploaded. Click on that.
1. there is _Vulnerabilities_ and a _Build History_ tab. Explore them. Depending on what you pushed, you could see a bunch of issues or, perhaps it's clean.

## 🥥 Proxy/Cache through Harbor for DockerHub

1. log in to Harbor repository
    `docker login core.harbor.domain/foo`
1. pull the image as usual
    `docker pull core.harbor.domain/dockerhub_cache_proxy/library/alpine:2.6`
Subsequent image pulls will come from cache
