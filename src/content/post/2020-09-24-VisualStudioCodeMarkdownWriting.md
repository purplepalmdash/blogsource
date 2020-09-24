+++
title= "VisualStudioCodeMarkdownWriting"
date = "2020-09-24T09:54:27+08:00"
description = "VisualStudioCodeMarkdownWriting"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Installation
Install microsoft visual studio code via:    

```
$ sudo vim /etc/pacman.conf
[archlinuxcn]
#The Chinese Arch Linux communities packages.
SigLevel = Never
Server   = http://repo.archlinuxcn.org/$arch
$ sudo pacman -Sy
$ sudo pacman -S visual-studio-code-bin
```

### Install Plugin
Install via:    

![/images/2020_09_24_09_58_02_326x285.jpg](/images/2020_09_24_09_58_02_326x285.jpg)

Install prince(for converting to pdf)via:    

```
$ sudo pacman -S prince-bin
```
Configure the stylesheet for prince's generated output:    

```
$ cat ~/.mume/style.less
/* Please visit the URL below for more information: */
/*   https://shd101wyy.github.io/markdown-preview-enhanced/#/customize-css */ 

.markdown-preview.markdown-preview {
  // modify your style here
  // eg: background-color: blue;  
  &.prince {
    @page {
      //size: A4 landscape
      size: A4 PortraitModes
    }
    @font-face {
      font-family: sans-serif;
      font-style: normal;
      font-weight: normal;
      src: url("/usr/share/fonts/adobe-source-han-sans/SourceHanSansCN-Regular.otf")
  }

  @font-face {
      font-family: sans-serif;
      font-style: normal;
      font-weight: bold;
      src: url("/usr/share/fonts/adobe-source-han-sans/SourceHanSansCN-Bold.otf")
  }

  @font-face {
      font-family: sans-serif;
      font-style: italic;
      font-weight: normal;
      src: url("/usr/share/fonts/adobe-source-code-pro/SourceCodePro-It.otf")
      //src: url("/usr/share/fonts/adobe-source-han-sans/SourceHanSansCN-Regular.otf")
  }

  @font-face {
      font-family: sans-serif;
      font-style: italic;
      font-weight: bold;
      src: url("/usr/share/fonts/adobe-source-code-pro/SourceCodePro-BoldIt.otf")
      //src: url("/usr/share/fonts/adobe-source-han-sans/SourceHanSansCN-Bold.otf")
  }
  }
}
```
Or edit the css via `ctrl+shift+p` and Run `Markdown Preview Enhanced: Customize Css`, Insert the above code.   
