# Helm Charts

Helm Charts Repo for my various projects.

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
