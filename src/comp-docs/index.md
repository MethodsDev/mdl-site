# Computational Docs

This set of pages is to document computational tools used in the MDL. Right now we have:
 
 * [Some tips](tips-and-tricks.md) that might be useful for getting started
 * A guide to [setting up Jupyterlab on a GCP VM](gcp-vms.md)
 * A short guide to [code style](code-style.md) for Python projects in the lab

## MDL packages

We have a small but growing library of Python packages on [Github](https://www.github.com/MethodsDev). Packages live in the `mdl` [namespace package](https://packaging.python.org/en/latest/guides/packaging-namespace-packages/), but can be installed separately based on need.

[`isoscelles`](https://github.com/MethodsDev/isoscelles) is a toolkit for single-cell analysis, with a focus on isoforms (although much of the functionality applies to short-read data well). It does things like gene selection, shared-nearest-neighbors, and clustering.

[`MasSeqViz`](https://github.com/MethodsDev/MasSeqViz) currently contains some legacy code used for visualizing isoforms. It is not currently very usable, but as we process and visualize more data it is sure to grow.
