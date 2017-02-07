参考文档：https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant.html
执行vagrant update时报错：
```
failed generating SSL artifacts
```
发现是Vagrantfile中的这个脚本执行出错：
```
 ./../../lib/init-ssl-ca ssl
```
问题最终定位到脚本的最后一行：
```
openssl req -x509 -new -nodes -key "$OUTDIR/ca-key.pem" -days 10000 -out "$OUTFILE" -subj "/CN=kube-ca"
```
发现在windows下执行必须修改为```-subj "//CN=kube-ca"```(增加一个斜杠)。  
这样修改后手工执行上述openssl脚本不再报错。同目录下另一个init-ssl也要改。  
手工删除两个目录，否则报错：
```
$ rm -rf ./-p && rm -rf ssl
```
Vagrantfile中含有ruby脚本，该脚本在windows下执行时提示能执行点(.)。修改成下列的样子（前面增加了sh）：
```
system("mkdir -p ssl && sh ./../../lib/init-ssl-ca ssl") or abort ("failed generating SSL artifacts")
```
Vagrantfile中还有几个以点(.)开始的命令前面也要加上sh。