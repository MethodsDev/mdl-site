# Useful Tips

These are some tools, configuration tricks, and other tips that can make computational tasks easier. These are all matters of opinion, but they might help.

## Some general suggestions

 * Use `zsh` ("zee-shell"). `zsh` is an alternative to `bash`, which is the default interface when you open a Terminal on Mac. It has a lot of nice features compared to `bash`, like a better history and more useful auto-completion. See [`Oh-My-ZSH`](https://ohmyz.sh/) for a guide on how to install and configure.
 * Use `conda` and/or `mamba`. `conda` is a package manager, which makes it much easier to keep track of different versions and environments. [`miniconda`](https://docs.conda.io/en/latest/miniconda.html) is the minimal version which only includes the manager tool, without a set of default packages (which take up a lot of space). [`mamba`](https://mamba.readthedocs.io/en/latest/installation.html#installation) is a newer project which is much faster and works better at configuring complicated environments. If you are starting fresh, `micromamba` (from the link above) is probably the best option.

## Jupyter tips

[JupyterLab](https://jupyter.org/install) is the latest generation, and newer plugins will expect you to be using it rather than the original Jupyter Notebook.

A very useful plugin is [Notifications](https://github.com/mwakaba2/jupyterlab-notifications), which sends a pop-up when long-running tasks have completed. You can install it with `conda install -c conda-forge jupyterlab-notifications`. You might need to allow notifications on the website (in your browser) before it will work.

## SSH tips

### `.ssh/config` for UGER

If you use UGER interactively, you might want to log in to a specific node, _e.g._ to return to a `tmux` session. If you add this snippet to your `~/.ssh/config` file (create a file there if it doesn't exist), you can use `ssh broad` to log in to any node, and log in to a specific node with `ssh login01`, _etc_.

```
Host broad
    HostName login.broadinstitute.org
    User [your username]

Host login01 login02 login03 login04
    HostName %h.broadinstitute.org
    User [your username]
```

### Setting up SSH for your GitHub account

Some of our repositories are private to our GitHub account, which means that accessing the code requires logging in with your GitHub credentials. You can use SSH to make this automatic. [Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) has a useful guide for how to generate a key and add it to your account.

One trick to make this work is that you need to set the **remote url** for the repository, so that it knows you use ssh at all. You can do this from inside the repository directory, like this (using `slideseq-tools` as an example, even though it is a public repository):

```
git remote set-url origin git@github.com:MacoskoLab/slideseq-tools.git
```

## More?

[Edit this page](https://github.com/MethodsDev/mdl-site/edit/main/src/tips-and-tricks.md) to add your own suggestions!