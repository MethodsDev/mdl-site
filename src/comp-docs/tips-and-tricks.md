# Useful Tips

These are some tools, configuration tricks, and other tips that can make computational tasks easier. These are all matters of opinion, but they might help.

## Some general suggestions

 * Use `zsh` ("zee-shell"). `zsh` is an alternative to `bash`, which is the default interface when you open a Terminal on Mac. It has a lot of nice features compared to `bash`, like a better history and more useful auto-completion. See [`Oh-My-Zsh`](https://ohmyz.sh/) for a guide on how to install and configure.

    **Note:** `zsh` is now the default shell on Mac OS. `Oh-My-Zsh` is still a nice addition.

 * Use `conda` and/or `mamba`. `conda` is a package manager, which makes it much easier to keep track of different versions and environments. [`miniconda`](https://docs.conda.io/en/latest/miniconda.html) is the minimal version which only includes the manager tool, without a set of default packages (which take up a lot of space). [`mamba`](https://mamba.readthedocs.io/en/latest/installation.html#installation) is a newer project which is much faster and works better at configuring complicated environments. If you are starting fresh, `mamba` is the best option, via their [`mambaforge`](https://github.com/conda-forge/miniforge#mambaforge) downloader.

    [`micromamba`](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html) is another version which aims to provide a small, single-file version of `mamba`. It is still a little rough around the edges for interactive usage but it can be even faster.

## Jupyter tips

[JupyterLab](https://jupyter.org/install) is the latest generation, and newer plugins will expect you to be using it rather than the original Jupyter Notebook.

A very useful plugin is [Notifications](https://github.com/mwakaba2/jupyterlab-notifications), which sends a pop-up when long-running tasks have completed. You can install it with `conda install -c conda-forge jupyterlab-notifications`. You might need to allow notifications on the website (in your browser) before it will work.

### Using SSH forwarding to connect to a GCP VM

If you are running JupyterLab on a Google VM, you will want to access the server. One way is to expose a port on the VM so that you can connect to it, but this requires some care to make sure the machine remains secure, and you will need to VPN into our network.

A simpler solution is to use `ssh` with port-forwarding, and map a port on the VM to a port on your own computer (`localhost`). JupyterLab is typically running on port 8080, and this command will let you access the VM, without needing to be on our network:

```
gcloud compute ssh [instance-name] --tunnel-through-iap -- -NL [local-port]:localhost:8080
```

The command won't terminate, it will remain running until you want to close the connection (use `Ctrl-C` in the terminal to stop it).

The option `--tunnel-through-iap` lets you connect without being inside the Broad network. If you _are_ inside the network, it can cause problems. If you don't want to remember this detail, you can create a shell function to figure this out for you, based on your IP (the Broad has a block of IPs starting with `69.173`):

```
function iap() {
   ! [[ `curl -s ifconfig.me` =~ ^69.173 ]] && echo --tunnel-through-iap
}
```

And then you use this function in your ssh command like so:

```
gcloud compute ssh [instance-name] `iap`
```

## Setting up SSH for your GitHub account

Some of our repositories are private to our GitHub account, which means that accessing the code requires logging in with your GitHub credentials. You can use SSH to make this automatic. [Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) has a useful guide for how to generate a key and add it to your account.

One trick to make this work is that you need to set the **remote url** for the repository, so that it knows you use ssh at all. You can do this from inside the repository directory, like this (using the repository for this website as an example):

```
git remote set-url origin git@github.com:MethodsDev/mdl-site.git
```

## More?

[Edit this page](https://github.com/MethodsDev/mdl-site/edit/main/src/tips-and-tricks.md) to add your own suggestions!
