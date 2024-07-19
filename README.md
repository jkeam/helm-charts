# Helm Charts

Helm Charts Repo for my various projects.

## Hello Go

`charts/hello-go` is my `hello-go` app that you can find here on [GitHub](https://github.com/jkeam/hello-go) and on [Quay](https://quay.io/repository/jkeam/hello-go?tab=tags)

### Usage

```shell
helm repo add jonkeam https://jonkeam.com/helm-charts  # or use the UI
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

# any updates to the `main` branch will create a release
#   and potentially update the github pages
```

### References

1. [Inspired by Gerk Elznik](https://medium.com/@gerkElznik/provision-a-free-personal-helm-chart-repo-using-github-583b668d9ba4)
