+++
title= "OnUsingAsciidoctorPDF"
date = "2020-03-14T08:22:36+08:00"
description = "OnUsingAsciidoctorPDF"
keywords = ["Technology"]
categories = ["Technology"]
+++
Records the steps for generating the chinese language version of linstor documentation. 

### Source Code Preparation
Clone the whole source tree from github:     

```
$ git clone https://github.com/LINBIT/linbit-documentation.git
```
For I want to generate chinese version, I need `asciidoctorr-pdf-cjk-kai_gen_gothic` to be installed and also its fonts, so I changed the Dockerfile for generating the docker images which could support chinese language pdf generation:    

```
# vim Dockerfile
FROM debian:buster
MAINTAINER Roland Kammerer <roland.kammerer@linbit.com>
RUN groupadd --gid 1000 makedoc
RUN useradd -u 1000 -g 1000 makedoc
RUN apt-get update && apt-get install -y make inkscape ruby po4a patch
RUN gem install --pre asciidoctor-pdf
RUN gem install --pre asciidoctor-pdf-cjk
RUN gem install asciidoctor-pdf-cjk-kai_gen_gothic
RUN asciidoctor-pdf-cjk-kai_gen_gothic-install
USER makedoc
# make dockerimage
# sudo docker images | grep linbit
linbit-documentation   latest              118d89f74702        47 hours ago        902MB
```
Generate the po files:    

```
# cd linbit-documentation-master
# chmod 777 -R *
# make UG9-pot-docker
# cd UG9/en
# ls *.pot
about-linstor.pot              administration-manual.pot      docker-linstor.pot    features.pot          internals.pot            lvm.pot        opennebula-linstor.pot  pacemaker.pot        recent-changes.pot   xen.pot
about.pot                      benchmark.pot                  docker.pot            fundamentals.pot      kubernetes.pot           man-pages.pot  opennebula.pot          proxmox-linstor.pot  rhcs.pot
administration-drbdmanage.pot  build-install-from-source.pot  drbdmanage-more.pot   gfs.pot               latency.pot              more-info.pot  openstack-linstor.pot   proxmox.pot          throughput.pot
administration-linstor.pot     configure.pot                  drbd-users-guide.pot  install-packages.pot  linstor-users-guide.pot  ocfs2.pot      openstack.pot           proxy.pot            troubleshooting.pot
```
The pot (portable object template) which holds the tranlatable items. I wrote a simple python script which uses baidu api for translating them into chinese:    

```
# cd UG9/en
# vim autotrans.py
import re
import polib
import sys
import translators as ts

# pot files should be passed in argv, e.g python autotrans.py example.pot
print("#######################################################################3")
print("#######################################################################3")
print("#######################################################################3")
print("################# Processing "+ sys.argv[1] + "###############################")
print("#######################################################################3")
print("#######################################################################3")
print("#######################################################################3")
print("#######################################################################3")
po = polib.pofile(sys.argv[1])

for entry in po:
    # If contains '\n', then ignore translation
    print(entry.msgid)
    if "\n" in entry.msgid:
        print("********************Ignore translation***********************")
        entry.msgstr = entry.msgid
    else:
        # Normal texts should be translated. 
        entry.msgstr = ts.baidu(entry.msgid, 'auto', 'zh-CN')
    print(entry.msgstr)
    # msgid is the origin one, while msgstr should be translated one. 
    # << some_text_here >> should be selected out
    msgstr_links = []
    msgid_links = []
    # origin ones
    msgid_links = re.findall(r"(\<\<*.*?\>\>)", entry.msgid)
    # translated ones, needs to be recover using origin ones.
    msgstr_links = re.findall(r"(\<\<*.*?\>\>)", entry.msgstr)
    if len(msgstr_links)>0:
        ### only run in matched at least once
        for i in range(0, len(msgstr_links)):
            entry.msgstr = entry.msgstr.replace(msgstr_links[i], msgid_links[i])

po.save('./cn/' + sys.argv[1])
```
Use a shell wrapped commands for tranlating all of the pot files:    

```
# for i in `ls *.pot`
do
python3 autotrans.py $i
done
```
The tranlation goes one-by-one, until all of the pot files has been translated.   

Notice: the auto translation got many errors, you have to correct them manually.  

### Building steps
I use ja language for start, cause japanese version has been translated:     

```
# cd UG9
# cp -r ja cn
# cd cn/
# rm -f *.po
# cp ../cn/*.pot .
```
Write a simple shell script for changing name from `pot` to `po`:    

```
# vim rename.sh
for i in `ls *.pot`
do
	name=`echo $i | awk -F '.' {'print $1'}`
	mv $i $name.po
done
# chmod 777 rename.sh
# ./rename.sh
# rm -f rename.sh
# ls *.po
about-linstor.po              administration-manual.po      docker-linstor.po    features.po          internals.po            lvm.po        opennebula-linstor.po  pacemaker.po        recent-changes.po   xen.po
about.po                      benchmark.po                  docker.po            fundamentals.po      kubernetes.po           man-pages.po  opennebula.po          proxmox-linstor.po  rhcs.po
administration-drbdmanage.po  build-install-from-source.po  drbdmanage-more.po   gfs.po               latency.po              more-info.po  openstack-linstor.po   proxmox.po          throughput.po
administration-linstor.po     configure.po                  drbd-users-guide.po  install-packages.po  linstor-users-guide.po  ocfs2.po      openstack.po           proxy.po            troubleshooting.po
```
Add `cn` to `UG-top.mk`:     

```
- languages = en ja
+ languages = en ja cn
```

Change `UG-build.mk` for adding `cn` selection:    

```
# diff UG-build.mk  UG-build-back.mk 
77c77
< 	if [ -d $(FONTDIR) ] && [ "$(lang)" != "ja" ] && [ "$(lang)" != "cn" ]; then \
---
> 	if [ -d $(FONTDIR) ] && [ "$(lang)" != "ja" ]; then \
83c83
< 	if [ -d $(FONTDIR) ] && [ "$(lang)" != "ja" ] && [ "$(lang)" != "cn" ]; then \
---
> 	if [ -d $(FONTDIR) ] && [ "$(lang)" != "ja" ]; then \
```
Change the Makefile for generating cn language pack:    

```
# vim UG9/cn/Makefile
ASCIIDOCTOR_ADD_OPTIONS=-r asciidoctor-pdf-cjk -r asciidoctor-pdf-cjk-kai_gen_gothic -a pdf-style=KaiGenGothicCN
lang=cn

include ../../UG-build-po.mk
include ../../UG-build.mk
```
Add the `default-theme.yml` under the `UG9/cn` folder, and changes its fonts to CN related fonts:     

```
# docker run --rm --entrypoint /bin/sh linbit-documentation:latest -c "cat /var/lib/gems/2.5.0/gems/asciidoctor-pdf-1.5.3/data/themes/default-theme.yml" > default-theme.yml
# vim default-theme.yml
font:
  catalog:
    # Noto Serif supports Latin, Latin-1 Supplement, Latin Extended-A, Greek, Cyrillic, Vietnamese & an assortment of symbols
    Noto Serif:
      normal: KaiGenGothicCN-Regular.ttf
      bold: KaiGenGothicCN-Bold.ttf
      italic: KaiGenGothicCN-Regular-Italic.ttf
      bold_italic: KaiGenGothicCN-Bold-Italic.ttf
    # M+ 1mn supports ASCII and the circled numbers used for conums
    M+ 1mn:
      normal: KaiGenGothicCN-Regular.ttf
      bold: KaiGenGothicCN-Bold.ttf
      italic: KaiGenGothicCN-Regular-Italic.ttf
      bold_italic: KaiGenGothicCN-Bold-Italic.ttf
page:
..........
```
Change the patch files under `UG9/cn`:    

```
# mv UG9/cn/drbd-users-guide.adoc-ja.patch UG9/cn/drbd-users-guide.adoc-cn.patch
# vim UG9/cn/drbd-users-guide.adoc-cn.patch
@@ -1,4 +1,23 @@
 :doctype: article
+:lang: ja
+:chapter-label:
+:toc-title: 目录
+:preface-title: 前言标题
+:appendix-caption: 附录标题
+:caution-caption: 注意
+:example-caption: 示例标题
+:figure-caption: 图形标题
+:important-caption: 重要
+:last-update-label: 最終更新
+:listing-caption: 列表标题
+:manname-title: 人名 
+:note-caption: 注記
+:preface-title: 前言标题
+:table-caption: 表标题
+:tip-caption: 要点标题
+:untitled-label: 未命名标签
+:version-label: 版本标签
+:warning-caption: 警告
 :source-highlighter: bash
 :listing-caption: Listing
 :icons: font
```
Now change the permission for all of the folder, thus docker could have write priviledge to current folder:     

```
# chmod a+w -R *
#  make UG9-pdf-finalize-docker lang=cn
```
By running `make UG9-pdf-finalize-docker` could generate chinese version's pdf, By running ` make UG9-html-finalize-docker lang=cn ` will generate chinese version's htmls.  


