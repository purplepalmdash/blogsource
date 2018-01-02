---
categories: ["vim"]
comments: true
date: 2014-07-12T00:00:00Z
title: Hightlight Jade File
url: /2014/07/12/hightlight-jade-file/
---

Install `vim-jade` via:    

```
    Bundle 'digitaltoad/vim-jade'

```
Then in vim type `:BundleInstall` this will automatically install the plugin of vim-jade.     
Enable the Highlight of jade file via:    

```
au BufNewFile,BufRead,BufReadPost *.jade set filetype=jade

```
Now everytime you open jade file, it will be automatically be highlighted.    
