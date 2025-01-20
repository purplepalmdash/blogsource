+++
title= "mdbookworkflow"
date = "2025-01-20T23:46:32+08:00"
description = "mdbookworkflow"
keywords = ["Technology"]
categories = ["Technology"]
+++
Steps for publishing to github pages using mdbook.    


```
$ mkdir lxcDesktop
$ cd lxcDesktop
$ mkbook init
Do you want a .gitignore to be created? (y/n)
y
What title would you like to give the book? 
lxc-desktop
2025-01-20 22:36:48 [INFO] (mdbook::book::init): Creating a new book with stub content

All done, no errors...
$ tree -a
.
├── book
├── book.toml
├── .gitignore
└── src
    ├── chapter_1.md
    └── SUMMARY.md

3 directories, 4 files
$ mkdir .github
$ cd .github
$ mkdir workflows
$ cd workflows
$ vim PublishMySite.yml
```
Content for `PublishMySite.yml`:    

```
name: PublishMySite

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Build markdown files to a static site.
      - name: Setup mdBook
        uses: peaceiris/actions-mdbook@v1
        with:
          mdbook-version: "latest"
      - run: mdbook build . --dest-dir ./book # --dest-dir is relative to <dir>

      # Publish the static site to gh-pages branch.
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN}}
          publish_dir: ./book
          publish_branch: gh-pages
```
Back to repository:    

```
git init
git add .
git commit -m "init"
```
meanwhile, on github, do following steps:    

```
GitHub > New Repository

GitHub > Repository > Settings > Actions > General >

Actions permissions: Allow all actions and reusable workflows
Workflow permissions: Read and write permissions
Click Save
```

Commit to remote branch:    

```
git remote add origin git@github.com:purplepalmdash/lxcDesktop.git
git branch -M main
git pull --rebase origin main
git push origin main
```
On github, do following:     

```
GitHub > Repository > Settings > Pages > Branch > gh-pages > Click Save
```

The result is shown as in following picture:     

![/images/2025_01_20_23_52_16_1233x385.jpg](/images/2025_01_20_23_52_16_1233x385.jpg)


