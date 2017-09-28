# kubesh

Spawn a shell for a particular kubectl context (cluster).

# Synopsis

```
kubesh --context=name-of-context [--credentials-file=/path/to/credentials.json]
```

# Installation

TODO: I am not sure if this is the best way. Patches welcome

```
$ git clone git@github.com:lestrrat/kubesh.git ~/.kubesh
```

Then add PATH to ~/.kubesh/bin

```
export PATH="~/.kubesh/bin:$PATH"
```

# Description

When you are using multiple clusters, you normally use `kubectl config use-context`
to change which cluster (and assocaited configuration) we use. The
problem with using contexts this way is that changes are global:
You use `use-context` in one terminal, and the changes are observed
in another terminal as well.

So what happens when you want to work on one cluster which also
depents on another cluster, and you want to be monitoring both?

This is where `kubesh` comes in. It does not force you to create
multiple `KUBE_CONFIG`s, and it allows you to keep working on
the same context for the duration of the shell session, and you
do not have to explicitly set the context by yourself.

# Using Multiple GCP Accounts

Currently, there are only two ways I know that really works for working with
multiple accounts.

## 1. Add Your Main-Account As A Member Of Your Sub-Account

Configure your project so that your main account is a member of the project
for your sub-account. May have security implications, but it's easy.

## 2. Modify ~/.kube/config By Hand

This has proven to be sometimes problematic, possibly because of caching.
It seemd to work for me when I started with a fresh ~/.kube/config file.

The way kubectl gets tokens is by invoking `gcloud config config-helper`.
How this command is invoked is stored in `~/.kube/config` file, like so:

```yaml
- name: gke_project_zone_cluster
  user:
    auth-provider:
      config:
        cmd-args: config config-helper --format=json
        cmd-path: /usr/local/google-cloud-sdk/bin/gcloud
        ...
```

You can change tell `gcloud config config-helper` to work with a particular
GCP configuration by setting the `--configuration` parameter. So you could do
something like below:

```yaml
- name: gke_project_zone_cluster_A
  user:
    auth-provider:
      config:
        cmd-args: config config-helper --format=json --configuration=A
        cmd-path: /usr/local/google-cloud-sdk/bin/gcloud
- name: gke_project_zone_cluster_B
  user:
    auth-provider:
      config:
        cmd-args: config config-helper --format=json --configuration=B
        cmd-path: /usr/local/google-cloud-sdk/bin/gcloud
        ...
```

This will then force `kubectl` to use different configurations when
generating access tokens

# Integration With `peco`

If you are too lazy to type the context name, you can integrate it with [peco](https://github.com/peco/peco)

```
function peco-kubesh() {
    local selected=$(kubectl config get-contexts | sed -e 's/\*//' | awk 'NR < 2 { next } {print $1}' | peco)
    ST=$?
    if [[ $ST != 0 ]]; then
        exit 1
    fi
        
    if [ -n "$selected" ]; then
        kubesh --context=$selected
    fi  
}

zle -N peco-kubesh
```

# Support For Other Utilitites

`helm`: helm since 2.0.0 supports `--kube-context` argument, and
we provide support for it.

`stern`: stern supports switching contexts, and we provide support for it.

# Cavetas

As of this writing, it only works for zsh. Because we spawn a new shell,
we need to stop the newly created shell from reading the rcfiles
again, but I could not find a portable way to do it. Patches welcome.

[If `kubectl` ever supports `KUBE_CONTEXT` environment variable, this utility probably be deprecated.](https://github.com/kubernetes/kubernetes/issues/27308)

