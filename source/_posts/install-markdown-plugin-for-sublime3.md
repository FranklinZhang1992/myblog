---
title: Sublime3 安装Markdown插件
date: 2017-09-14 21:49:34
tags: Sublime3
---

This blog introduces two popular Markdown plugins in Sublime and describes how to install these two plugins for Sublime3.

### [MarkdownEditing](https://github.com/SublimeText-Markdown/MarkdownEditing)

MarkdownEditing provides a decent Markdown color scheme (light and dark) with more **robust** syntax highlighting and useful Markdown editing features for Sublime Text. 3 flavors are supported: Standard Markdown, **GitHub flavored Markdown**, MultiMarkdown.

### [OmniMarkupPreviewer](https://github.com/timonwong/OmniMarkupPreviewer)

<!--more-->

OmniMarkupPreviewer renders markups into htmls and send it to web browser in the backgound, which enables a live preview. Besides, OmniMarkupPreviewer provide support for exporting result to html file as well.

## How to install the plugins

Before we start to install these two plugins, we need to install [Package Control](https://packagecontrol.io/installation). This is the very first step because this plugin is necessary for Sublime to intall/manage plugins.

### Install Package Control

- Open Sublime console (View -> Show Console)

- Paste below codes into the input box and type **enter**.

```
import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

### Install MarkdownEditing

- Enter Package Control. (Preferences -> Package Control)

![Package Control](/assets/img/install_markdown_plugin_for_sublime3/package_control.PNG)

- Choose **package instal** and type in **enter**, there will be a delay because Sublimes to request for remote repository.

![choose install package from list](/assets/img/install_markdown_plugin_for_sublime3/choose_install_package_from_list.PNG)

- When you see the plugin list, type in **MarkdownEditing** and **enter**, then the plugin installation begins. The plugin won't take effect until you restart Sublime.

![plugin list](/assets/img/install_markdown_plugin_for_sublime3/plugin_list.PNG)

### Install OmniMarkupPreviewer

- Enter Package Control. (Preferences -> Package Control)

![Package Control](/assets/img/install_markdown_plugin_for_sublime3/package_control.PNG)

- Choose **package instal** and type in **enter**, there will be a delay because Sublimes to request for remote repository.

![choose install package from list](/assets/img/install_markdown_plugin_for_sublime3/choose_install_package_from_list.PNG)

- When you see the plugin list, type in **OmniMarkupPreviewer** and **enter**, then the plugin installation begins. The plugin won't take effect until you restart Sublime.

![OmniMarkupPreviewer in plugin list](/assets/img/install_markdown_plugin_for_sublime3/OmniMarkupPreviewer_in_list.PNG)
