# Helm Charts

Helm Charts Repo for my various projects.

## Tailscale

### Helm Chart

I created my own Tailscale Operator helm chart just so that I had total control.
This is what I did:

1. Created my own operator [here](https://github.com/jkeam/tailscale-ocp-operator)
2. Created a release of that operator [here](https://quay.io/jkeam/tailscale-ocp-operator), first release is called '0.0.1'
3. Copied over the contents of the helm chart [here](https://github.com/tailscale/tailscale/tree/main/cmd/k8s-operator/deploy/chart) into `./charts/tailscale`
4. Modified `./charts/tailscale/Chart.yaml`
5. Checked all this in, at which point my pipelines created a new helm chart
6. In OCP, made sure I had a helm chart repo for https://keamchart.com

### Tailscale Operator

#### Tailscale Prequisite

1. Create tailnet tags
2. Create OAuth client for `Devices - Core` and `Auth Keys`
3. Saved `clientId` and `clientSecret` away

#### Tailscale Operator App

1. In OCP, create `tailscale` project
2. In OCP, use my Keam Helm Chart repo to create a new Tailscale Operator
3. In the `values.yaml` that appears before creation, put in the following
    ```yaml
    oauth:
      clientId: 'whatever'
      clientSecret: 'whatever'
    ```
4. Click Save


#### Tailscale Proxy

We have to allow the tailscale proxy to run as root,
so we have to first create the `ProxyClass`


```yaml
apiVersion: tailscale.com/v1alpha1
kind: ProxyClass
metadata:
  name: ocp
spec:
  statefulSet:
    pod:
      tailscaleInitContainer:
        securityContext:
          privileged: true
      tailscaleContainer:
        securityContext:
          privileged: true
```

And then run:

```shell
oc adm policy add-scc-to-user privileged -z proxies -n tailscale
```

#### Annotate Services

Now we can annotate whatever service we want and that will get a Tailscale proxy.
Here's an example where we annotate the router-internal-default which will then
expose all routes, which is nice; and should be all we need.

```shell
oc label service router-internal-default tailscale.com/proxy-class=ocp -n openshift-ingress
oc annotate service router-internal-default tailscale.com/expose=true -n openshift-ingress
```

#### DNS

The console url is very specific, so you need to update some DNS somewhere in order to resolve the OCP Console URL to the new Tailscale IP address.  The quick and dirty is just to update your `/etc/hosts` on whatever computer you are using.


## Hello Go

`charts/hello-go` is my `hello-go` app that you can find here on [GitHub](https://github.com/jkeam/hello-go) and on [Quay](https://quay.io/repository/jkeam/hello-go?tab=tags).

### Usage

```shell
oc apply -k ./openshift/helm-repo.yaml  # or using the UI
# then create the `Hello Go` app using the Developer perspective in OpenShift
```

## Development

If you're wondering how I did all this, here are the instructions I followed.
Note, you will have to change to your own repo if you want to do the same.

```shell
# create your own github repo
git clone git@github.com:jkeam/hello-go.git
git checkout --orphan gh-pages
git rm -rf .  # delete everything in the gh-pages branch
git commit -m "Initial commit" --allow-empty
git push --set-upstream origin gh-pages

# download github cli, called gh
gh auth login
gh release create v0.0.1 --notes "Initial release"

# change permission of github action to read/write, "Read and write permissions"
#   url for that is here: https://github.com/jkeam/hello-go/settings/actions
git checkout main
mkdir -p charts && cd charts
helm create hello-go
# then update all of the values.yaml and various other things like adding a route template
# check in and push everything to `main` branch

# for updating in the future:
# Code Repo
# 1. update the code in hello-go
# 2. create a new release in that repo, name it something like v0.0.2
# This Repo in Main Branch
# 3. in ./charts/hello-go/Chart.yaml make sure `appVersion` matches, so it would be 0.0.2 (notice no 'v')
# 4. in ./charts/hello-go/Chart.yaml update the `version` to some number bigger so that a new helm chart is created

# To recap:
# - updating this repo in the main branch can trigger a new helm chart version creation (that appears in the gh-pages branch)
# - updating README.md in the `gh-pages` branch will trigger a build so that the [website](https://keamchart.com) gets updated
```

### References

1. [Inspired by Gerk Elznik](https://medium.com/@gerkElznik/provision-a-free-personal-helm-chart-repo-using-github-583b668d9ba4)
2. [Helm Chart Releaser](https://helm.sh/docs/howto/chart_releaser_action/)
3. [Tailscale on OCP](https://blog.cszevaco.com/blog/tailscale)
