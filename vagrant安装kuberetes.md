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