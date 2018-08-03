# kustomize-docker
Docker image for systems using Kustomize and kubectl.

Included additions beyond base Apline:
- Kustomize 1.0.5
- Kubectl 1.11.1
- envsubst

Working directory is set to `/working/` if you need to mount files.

# Usage
## On docker
If you're making up your own workflow, the image is on [Docker Hub](https://hub.docker.com/r/traherom/kustomize-docker/).

## End-to-end Usage
Using the commands shown below, a complete deploy can be run by piping the output of each into the others:

```bash
# envsubst for plain sh using docker. Passes all exported variables off to docker
ENV=$(env | grep = | grep -v '^_' | sed 's/\([^=]*\)=.*/ -e \1 /' | tr -d '\n')

docker run --rm -i \
    $ENV \
    -w /working/ \
    -v "$(pwd):/working/" \
    traherom/kustomize-docker \
    kustomize build /working/overlays/$OVERLAY \
| docker run --rm -i \
    $ENV \
    traherom/kustomize-docker \
    envsubst \
| docker run --rm -i \
    -v "$KUBECONFIG:/root/.kube/config" \
    traherom/kustomize-docker \
    kubectl apply -f -
```

## envsubst
Envsubst may be useful in building deploy-specific Kustomize overlays. A general patterns for this is:

```bash
# envsubst for plain sh using docker. Passes all exported variables off to docker
ENV=$(env | grep = | grep -v '^_' | sed 's/\([^=]*\)=.*/ -e \1 /' | tr -d '\n')

docker run --rm -i \
    $ENV \
    traherom/kustomize-docker \
    envsubst \
    < input_file.yaml \
    > output_file.yaml
```

## kustomize
If `$OVERLAY` is the name of the overlay to use and your current working directory is the base of your
Kustomize files:

```bash
ENV=$(env | grep = | grep -v '^_' | sed 's/\([^=]*\)=.*/ -e \1 /' | tr -d '\n')
docker run --rm -i \
    $ENV \
    -w /working/ \
    -v "$(pwd):/working/" \
    traherom/kustomize-docker \
    kustomize build /working/overlays/$OVERLAY
```

Note that all `kustomization.yaml`s, resources, patches, etc must be under the working directory or the
container will not be able to access them.

We also include all the local environment variables in the kustomize run because configMap and secret
generators might do things like "`echo $ENV_VAR`" and we want that to work.

## kubectl
If `$KUBECONFIG` is the path to your K8s configuration file (this is the default variable named used by Gitlab's CI):

```bash
docker run --rm -i \
    -v "$KUBECONFIG:/root/.kube/config" \
    traherom/kustomize-docker \
    kubectl apply -f - \
    < input_file.yaml
```

If you're going to be doing any `kubectl cp`ing, don't forget to add the appropriate volumes.
