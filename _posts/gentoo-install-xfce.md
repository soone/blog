title: 在gentoo上安装xfce遇到的一些问题
date: 2013-02-14 17:48:56
tags: 
- gentoo
- xfce
- emerge
- tilda
categories: 
- gentoo
---
以前就听说过tilda，后来一直没有时间好好研究研究，那天在n4上的浏览器上看到的tilda浏览记录，就想想要不在gentoo上先尝试一把。  
不过我的gentoo还没有装桌面，那就从装桌面开始。
{% codeblock X服务器配置指南 http://www.gentoo.org/doc/zh_cn/xorg-config.xml %}
emerge -pv xorg-server  
emerge xorg-server
{% endcodeblock %}
运行上面的安装命令，总是得到以下的问题显示
{% codeblock %}
The following USE changes are necessary to proceed:  
#required by x11-drivers/xf86-video-vmware-12.0.2-r1, required by x11-base/xorg-drivers-1.13[video_cards_vmware], required by x11-base/xorg-server-1.13.0-r1[xorg], required by x11-drivers/xf86-video-glint-1.2.8>=x11-libs/libdrm-2.4.40 libkms  
#required by x11-drivers/xf86-video-vmware-12.0.2-r1, required by x11-base/xorg-drivers-1.13[video_cards_vmware], required by x11-base/xorg-server-1.13.0-r1[xorg], required by x11-drivers/xf86-video-glint-1.2.8 =media-libs/mesa-9.0 xa  

 * IMPORTANT: 6 news items need reading for repository 'gentoo'.  
 * Use eselect news to read news items.  
{% endcodeblock %}
看提示是需要安装libdrm和mesa，不过安装过后还是一样出现这个提示，从google大神那得到一个提示说是要先做以下操作:
{% codeblock %}
cd /etc/init.d  
ln -s net.lo net.eth1  
rc-update add net.eth1 default
{% endcodeblock %}
再运行上面装xorg的命令，可惜还是得到这样的提示
{% codeblock %}
The following USE changes are necessary to proceed:  
#required by x11-drivers/xf86-video-vmware-12.0.2-r1, required by x11-base/xorg-drivers-1.13[video_cards_vmware], required by x11-base/xorg-server-1.13.0-r1[xorg], required by x11-drivers/xf86-video-glint-1.2.8>=x11-libs/libdrm-2.4.40 libkms  
#required by x11-drivers/xf86-video-vmware-12.0.2-r1, required by x11-base/xorg-drivers-1.13[video_cards_vmware], required by x11-base/xorg-server-1.13.0-r1[xorg], required by x11-drivers/xf86-video-glint-1.2.8=media-libs/mesa-9.0 xa  

Use --autounmask-write to write changes to config files (honoring CONFIG_PROTECT).  
{% endcodeblock %}
不过这次最后的提示不一样了，说需要加--autounmask-write的参数去修改一些config文件。于是乎第三次运行以下命令
{% codeblock %}
emerge xorg-server --autounmask-write
{% endcodeblock %}
这回是另一个提示
{% codeblock %}
The following USE changes are necessary to proceed:  
#required by x11-drivers/xf86-video-vmware-12.0.2-r1, required by x11-base/xorg-drivers-1.13[video_cards_vmware], required by x11-base/xorg-server-1.13.0-r1[xorg], required by x11-drivers/xf86-video-glint-1.2.8>=x11-libs/libdrm-2.4.40 libkms  
#required by x11-drivers/xf86-video-vmware-12.0.2-r1, required by x11-base/xorg-drivers-1.13[video_cards_vmware], required by x11-base/xorg-server-1.13.0-r1[xorg], required by x11-drivers/xf86-video-glint-1.2.8=media-libs/mesa-9.0 xa  

Autounmask changes successfully written. Remember to run dispatch-conf.
{% endcodeblock %}
根据提示再运行
{% codeblock %}
dispatch-conf
{% endcodeblock %}
按u选择使用新的文件替换，最后才可以正常的运行
{% codeblock %}
emerge xorg-server --autounmask-write
{% endcodeblock %}

