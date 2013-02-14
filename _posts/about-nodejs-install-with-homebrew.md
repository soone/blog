title: 关于用homebrew安装nodejs无法link的解决方法
date: 2013-02-08 00:29:46
tags: [mac, homebrew, nodejs, npm]
categories: [nodejs, mac]
---
为了使用hexo，装nodejs是必须的，索性我的mac已经装了homebrew这个包管理神器了，有了这个东西确实装起软件来方便很多。且看我如何用它装nodejs。

*其实只需要一步*
{% codeblock %}
brew install nodejs
{% endcodeblock %}

**且慢有状况**，报了下面这样一个信息：
{% codeblock %}
Warning: Could not link node. Unlinking...
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
You can try again using `brew link node'
{% endcodeblock %}

这时候虽然安装完成了，但是你直接在终端执行node命令的时候，会出现以下的信息
{% codeblock %}
-bash node command not found
{%endcodeblock %}

然后仔细看刚才的错误提示，brew建议我们执行
{% codeblock %}
brew link node
#google了一下有建议执行这样命令的
brew link -f node
{% endcodeblock %}

照做。。。然后是“外甥打灯笼--照旧”  
google发现stackoverflow.com上有人有类似问题，在第二个答案找到了比较靠谱的解决方法，不过也不完美哦,
{% blockquote http://stackoverflow.com/questions/12607155/error-the-brew-link-step-did-not-complete-successfully%}
You need to remove the npm package manually.  
first unlink node: brew link -n node  
remove npm folder: rm -R /usr/local/Cellar/node/0.8.10/lib/node_modules/npm  
link again: brew link node  
there will be a soft link to the new location of npm  
{% endblockquote %}

第一条命令是删除link，然后第二条命令要求把这个目录下的东西，关键就在这里，如果/usr/local/Cellar/node/0.8.10/lib/node_modules/目录下有npm这个目录，第三条命令还是会执行失败，但是如果你执行了第二条，那么第三条虽然成功了npm却同样找不到，因为npm目录已经被删除了，所以解决方法是，第二条不要执行rm命令，而是使用mv把npm目录暂时先移走，然后执行第三条命令，成功后再把npm移回来，这样就都ok了。  
最后一步就是做个软链到/usr/local/bin/npm下，这样整个问题就完美解决了  
完整步骤如下：  
{% codeblock %}
brew link -n node  
mv /usr/local/Cellar/node/0.8.15/lib/node_modules/npm ~/npm  
brew link node
mv ~/npm /usr/local/Cellar/node/0.8.15/lib/node_modules/npm  
ln -s /usr/local/Cellar/node/0.8.15/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm  
{% endcodeblock %}

All Done. Enjoy That!!  
