# TriggerMesh Operator

This is an Operator to install TriggerMesh This does not install Knative and
Istio

## Pre-requisites

Install the following operators:
- [tektoncd](https://github.com/tektoncd/operator)
- [Knative Eventing Operator](https://operatorhub.io/operator/knative-eventing-operator)
- [OpenShift Serverless Operator](https://github.com/openshift-knative/serverless-operator)

### Access to the Container Images

The frontend and backend are available as container images. You should be able
to pull them from the TriggerMesh repository:

```
docker pull gcr.io/triggermesh/tmback
docker pull gcr.io/triggermesh/tmfront
```

### Auth0

TriggerMesh uses [Auth0](https://auth0.com/) for Authentication. You need to
create a Auth0 application before using this operator.

To do so please follow this [how-to](./auth0.md).

Once you are done note down the Auth0 `clientid` and `clientsecret` values.

### Image registry

Since all user functions are eventually stored as Docker container images,
having an image registry reachable from the backend pod is one of the
prerequesites. By default, Triggermesh backend is looking for a local
unauthenticated registry at `knative.registry.svc.cluster.local` but it can be
changed by setting backend env variables:

- `REGISTRY_HOST` - docker registry domain, eg. `index.docker.io`
- `REGISTRY_SECRET_NAME` - if specified, use this K8s secret to authenticate in
  registry

The registry secret must be of type `docker-registry`. Also, since this secret is being
used as both SA image pull secret and image push config for kaniko executor,
creating `config.json` secret key with the same value may be required.

Please be aware that the provided registry secret will be copied to all user
namespaces and eventually can be accessed by the user. Creating a dedicated
profile and/or using an access token instead of your password may be a good
idea.

### `anyuid` policy

Kaniko project, which is being used in Triggermesh as an image builder, is
having an [issue](https://github.com/GoogleContainerTools/kaniko/issues/105)
with running in non-privileged containers. To make it work properly in
OpenShift, we're adding `anyuid` security permission for authenticated users:

```
oc adm policy add-scc-to-group anyuid system:authenticated
```

### Ingress TLS certificate

Triggermesh Cloud provides an opportunity to configure
[tm](https://github.com/triggermesh/tm) CLI by downloading configuration file
from our dashboard. To generate valid CLI configuration you need to set a
backend's `FRONTEND_TLS_SECRET_NAME` env variable value to make it point to k8s
secret with a frontend ingress TLS certificate.

If you configure TLS in your cluster, the secret name can be found in the
ingress object specs - `spec.tls.hosts[0].secretName`.

If you didn't configure TLS, default Operator certificate should be stored in
`router-certs-default` secret of `openshift-ingress` namespace. In case when the
namespace with the secret is not the namespace of the backend pod,
`FRONTEND_TLS_SECRET_NAME` value must contain target namespace:
`router-certs-default/openshift-ingress`

## Usage

Create the CRD:

```
kubectl create -f https://raw.githubusercontent.com/triggermesh/operator/master/deploy/crds/app_v1alpha1_triggermesh_crd.yaml
```

Deploy the Operator in your k8s cluster:

```
kubectl create -f deploy/
```

Note: This deploys the Operator into the namespace `default`. Please update the
manifests if you want to deploy the Operator into another namespace.

### Create image pull secret

Create a pull secret named `docker-registry` in the operator's namespace. The secret name can be overridden with the `spec.backend.registrySecretName` variable of the CRD.

```
oc create secret docker-registry "docker-registry" \
  --docker-password $password \
  --docker-username $user \
  --docker-server docker.io
```

### Deploy TriggerMesh

You need to write a custom resource object and apply it. For example:

```
apiVersion: app.triggermesh.io/v1alpha1
kind: TriggerMesh
metadata:
  name: example-triggermesh
spec:
  # Default values copied from <project_dir>/helm-charts/triggermesh/values.yaml
...
```

An example is
[available](https://github.com/triggermesh/operator/blob/master/deploy/crds/app_v1alpha1_triggermesh_cr.yaml)

And apply it with:

```
kubectl apply -f tm.yaml
```

Enjoy TriggerMesh !

## Development

You can build the operator yourself with `operator-sdk`:

```
operator-sdk build gcr.io/triggermesh/operator:0.0.1
docker push gcr.io/triggermesh/operator:0.0.1
```

> Note that our CI builds the operator on every commit, hence
> `gcr.io/triggermesh/operator:latest` is always the latest revision

## Sponsor

This work is made possible by a partnership between TriggerMesh, Inc and
[VSHN, the DevOps Company](https://vshn.ch/), the developers of the Swiss
Container Platform [APPUiO](https://www.appuio.ch/)

## Support

This is heavily **Work In Progress** We would love your feedback on this
Operator so don't hesitate to let us know what is wrong and how we could improve
it, just file an [issue](https://github.com/triggermesh/aktion/issues/new)

## Code of Conduct

This operator is by no means part of [CNCF](https://www.cncf.io/) but we abide
by its
[code of conduct](https://github.com/cncf/foundation/blob/master/code-of-conduct.md)
