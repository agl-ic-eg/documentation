---
title: Adding Documentation
---

The [documentation gerrit repository](https://gerrit.automotivelinux.org/gerrit/admin/repos/AGL/documentation) contains AGL documentation website template and content, rendering is visible at [https://docs-agl.readthedocs.io/en/latest/](https://docs-agl.readthedocs.io/en/latest/). The documentation site is hosted on [readthedocs](https://readthedocs.org/projects/docs-agl/) and corresponding builds are mentioned [here](https://readthedocs.org/projects/docs-agl/builds/).

## Download Repository


Kindly check [this](https://wiki.automotivelinux.org/agl-distro/contributing) and clone with commit-msg hook :

```sh
$ git clone "ssh://$USER@gerrit.automotivelinux.org:29418/AGL/documentation" && scp -p -P 29418 $USER@gerrit.automotivelinux.org:hooks/commit-msg "documentation/.git/hooks/"
```

## Building a local site

1. Change into the directory

    ```sh
    $ cd documentation
    ```

2. Install MkDocs and rtd-dropdown theme

    ```sh
    $ sudo pip install -r requirements.txt
    ```

3. Serve locally (defaultly rendered at [127.0.0.1:8000/](127.0.0.1:8000/)):

    ```sh
    $ sudo mkdocs serve
    ```

Process to **add new or edit existing** markdown files to AGL documentation:

## Directory Structure

Find existing or add new markdowns in the following directory structure.

```sh
documentation
├── docs
│   ├── 0_Getting_Started
│   │   ├── 1_Quickstart
│   │   └── 2_Building_AGL_Image
|   ├── .....
|   |
|   ├──<Chapter-Number>_<Chapter-Name>
|   |   ├──<Subchapter-Number>_<Subchapter-Name>
|   |   |   ├──<Index-Number>_<Markdown-Title>.md
|   |   |   ├── .....
```

## Markdown Formatting

  1. Add following at the start of each markdown :

    ```sh
    ---
    title: <enter-title>
    ---
    ```

  2. Internal Linking :

    ```sh
    [<enter-title>](../<Chapter-Number>_<Chapter-Name>/<Subchapter-Number>_<Subchapter-Name>/<Index-Number>_<Markdown-Title>.md)
    ```

## Test Hyperlinks

[LinkChecker](https://wummel.github.io/linkchecker/) is a tool that allows to check all the hyperlinks in the site.

For testing hyperlinks as soon as the local site is running, do:

```sh
linkchecker http://localhost:8000
```

The ```linkchecker``` output will display the broken link and there location
in the site.


## Submitting changes

1. Install Git Review

    ```sh
    #recent version of git-review  (>=1.28.0 is required)
    sudo pip3 install git-review 
    ```

2. Write commit message

    ```sh
    # track all the new changes
    git add .

    # Write the commit message
    git commit --signoff
    ```

3. Push changes for review to Gerrit

    ```sh
    # first time only
    git review -s

    # then to push use
    git review
    ```