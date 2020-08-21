Example Kubernetes configurations for tiger cluster
===================================================

This repository has example kubernetes resources definitions
for the tiger cluster at Morgridge; they are meant to demonstrate
various aspects of running services on the cluster.

Directory structure:

- `manifests/` holds all the relevant kubernetes
  resources.
- `manifests/base` contains the vanilla version of different
  components (such as a worker node).  Changes that go across all
  deployments should go here.
  - `manifests/base/xrootd-standalone` is the deployment for the
    standalone xrootd server.
- `manifests/production` contains the customizations for
  the production instance (currently, the only instance).  This
  references the base components that should be installed.

Using these configs
-------------------

Please contact Brian Bockelman or Jeff Peterson for access to the
tiger cluster.  OSG or CHTC staff can get read access to the appropriate
namespace to interact with and monitor running services.

For updates, instead of using `kubectl` directly, please either
commit to the repository directly (experts) or send a PR for
a review of the changes (if you are beginning with k8s).  Do
*not* commit secrets to this repository (it's public)
and use the sealed-secret mechanism described below.

The changes in `git` should be reflected in the cluster within
approximately a minute.

Secret generation
-----------------

We cannot push secrets to GitHub so we must encrypt them first;
the sealed-secrets controller will decrypt them once they are
sync'd to the cluster itself.

To create a "sealed secret", run:

```
echo $idtoken | kubectl create --namespace playgroundd secret generic SECRETNAME --dry-run --from-file=token=/dev/stdin -o json > ~/mysecret.json
kubeseal -o yaml --cert tiger-cert.pem < ~/mysecret.json > manifests/production/mysecret.yaml
```

You can get the `kubeseal` binary from the sealed-secrets
[release page](https://github.com/bitnami-labs/sealed-secrets/releases).

Note you will need to change the `SECRETNAME` to be the prefixed
version of the secret name (if your `kustomization.yaml` uses a prefix).
If your namespace or secret name is incorrect, then the controller
will fail to unseal it.
