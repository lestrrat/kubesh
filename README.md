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

`kubectl` needs a JSON file that contains the credentials to access
GCP resources. To do this, the easiest way is probably just to run

```
gcloud auth application-default login
```

Then copy the file `~/.config/gcloud/application_default_credentials.json` to
a location where it does not get overwritten. Then pass the path to
that file in `--credentials-file`:

```
kubesh \
    --context=my-context \
    --credentials-file=/path/to/credentials.json
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

