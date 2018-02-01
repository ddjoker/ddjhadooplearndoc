1.  error setting certificate verify locations

    今天在新电脑用git clone的时候出现一个错误，导致无法克隆成功，错误大体描述为“error setting certificate verify locations”，翻译为错误设置证书验证位置，为此我在网上寻找了很多方法都没有解决，也查看了本地\Git\mingw64\ssl\certs 下面确实有证书。

    最终找到解决方法，发现这个错误是系统证书的问题，系统判断到这个行为会造成不良影响，所以进行了阻止，只要设置跳过SSL证书验证就可以了，用命令 ：
    `git config --global http.sslVerify false`
    设置完成后继续clone，就可以了。
2. 设置代理
    ```
    git config --global http.proxy http://127.0.0.1:1080
    git config --global https.proxy https://127.0.0.1:1080
    ```