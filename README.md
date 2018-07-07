# kustomize-docker
Docker image for systems using Kustomize and kubectl.

Included additions beyond base Apline:
- Kustomize 1.0.3
- Kubectl 1.11.0
- envsubst

Working directory is set to `/working/` if you need to mount files.

# Usage
## envsubst
Envsubst may be useful in building deploy-specific Kustomize overlays. A general patterns for this is:

```bash
# envsubst for plain sh using docker. Passes all exported variables off to docker
ENV=$(env | grep = | grep -v '^_' | sed 's/\([^=]*\)=.*/ -e \1 /' | tr -d '\n')

docker run -i \
    $ENV \
    traherom/kustomize-docker \
    envsubst \
    < input_file.yaml \
    > output_file.yaml
```

## kustomize
```bash
docker run -i \
    -w /working/ \
    -v "$(pwd):/working/" \
    -v "$KUBECONFIG:/root/.kube/config" \
    traherom/kustomize-docker \
    kustomize apply -f - \
    < input_file.yaml
    > output_file.yaml
```

## kubectl
If `$KUBECONFIG` is the path to your K8s configuration file (this is the default variable named used by Gitlab's CI):

```bash
docker run -i \
    -v "$KUBECONFIG:/root/.kube/config" \
    traherom/kustomize-docker \
    kubectl apply -f - \
    < input_file.yaml
```
