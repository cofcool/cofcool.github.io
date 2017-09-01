---
layout: post
category : Tech
title : Shadowsocks更新PAC文件失败问题
tags : [shadowsocks]
---
{% include JB/setup %}

更新失败原因为,Shadowsocks把GFWList地址编码在代码里了，可以通过修改代码的方式来更改改地址。也可手动下载`GFWList`文件。

@github/VincentSit写了一个脚本可实现该功能，具体情况可查看
https://gist.github.com/VincentSit/b5b112d273513f153caf23a9da112b3a

shell代码：
```shell
#!/bin/bash
# update_gfwlist.sh
# Author : VincentSit
# Copyright (c) http://xuexuefeng.com
#
# Example usage
#
# ./whatever-you-name-this.sh
#
# Task Scheduling (Optional)
#
#	crontab -e
#
# add:
# 30 9 * * * sh /path/whatever-you-name-this.sh
#
# Now it will update the PAC at 9:30 every day.
#
# Remember to chmod +x the script.


GFWLIST="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"
PROXY="127.0.0.1:1080"
USER_RULE_NAME="user-rule.txt"

check_module_installed()
{
	pip list | grep gfwlist2pac &> /dev/null

	if [ $? -eq 1 ]; then
		echo "Installing gfwlist2pac."

		pip install gfwlist2pac
	fi
}

update_gfwlist()
{
	echo "Downloading gfwlist."

	curl -s "$GFWLIST" --fail --socks5-hostname "$PROXY" --output /tmp/gfwlist.txt

	if [[ $? -ne 0 ]]; then
		echo "abort due to error occurred."
    exit 1
	fi

	cd ~/.ShadowsocksX || exit 1

	if [ -f "gfwlist.js" ]; then
		mv gfwlist.js ~/.Trash
	fi

	if [ ! -f $USER_RULE_NAME ]; then
		touch $USER_RULE_NAME
	fi

	/usr/local/bin/gfwlist2pac \
    --input /tmp/gfwlist.txt \
    --file gfwlist.js \
    --proxy "SOCKS5 $PROXY; SOCKS $PROXY; DIRECT" \
    --user-rule $USER_RULE_NAME \
    --precise

  rm -f /tmp/gfwlist.txt

  echo "Updated."
}

check_module_installed
update_gfwlist
```

在macOS上，如果`main.py`报错，可把`pip`替换为`pip2`。

另外，如果想要自定义规则，修改完`user-rule.txt`之后，需要更新PAC把该修改写入GFWList 规则列表来生效。

⚠️ ***注意***：

这个脚本已经不需要了，请自行去下载 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG/releases)。
