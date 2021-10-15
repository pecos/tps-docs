# TPS docs

![docs](https://github.com/pecos/tps-docs/actions/workflows/docs.yaml/badge.svg)

This repository houses underlying documentation collateral using
[Sphinx](https://www.sphinx-doc.org) for the TPS code hosted at
https://github.com/pecos/tps.

The resulting site is published using GitHub Pages to a companion site at
https://pecos.github.io/tps-docs/. To build the documentation site locally, you
can clone this repo and use Sphinx to built static html pages.

#### Necessary software dependencies using Python3/pip:

* pip3 install sphinx
* pip3 install sphinx_rtd_theme

#### Minimal working container:

If you are the container type, below is a minimal ubuntu-based `Dockerfile` that 
includes dependencies needed to build the documentation:

```
FROM ubuntu:latest

RUN apt-get update
RUN apt-get -y install python3-pip
RUN pip3 install sphinx
RUN pip3 install sphinx_rtd_theme
```

#### Building the docs locally:

* clone the repository
* `cd docs`
* issue `make html`
* point your local web browser to html/index.html

#### Production changes:

* a GitHub action is in-place to build the production site when changes are
  pushed to the `main` branch.  The resulting static html pages are committed
  to the `gh-pages` branch.

* if the build is successful, updates should be available at the [production
  site](https://pecos.github.io/tps-docs/) within several minutes after the
  push.  See the latest CI at https://github.com/pecos/tps-docs/actions to
  watch builds in process and confirm a successful build.
