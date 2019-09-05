# shell脚本设置git代理

```js
#!/bin/bash
proxy_url='socket://127.0.0.1:8086'
echo 'Please choose your operation:'
echo '1: add git http proxy'
echo '2: delete git http proxy'
read num

proxy(){
	if [[ 1 == $num ]]; then
		git config --global http.proxy $proxy_url
		git config --global https.proxy $proxy_url
		return 1
	elif [[ 2 == $num ]]; then
		git config --global --unset http.proxy $proxy_url
		git config --global --unset https.proxy $proxy_url
		return 2
	fi
}

proxy

if [ 1 == $? ]; then
	echo "git proxy was setted to $proxy_url !"
else
	echo 'git proxy was deleted!'
fi
```
