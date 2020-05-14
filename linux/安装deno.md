按官方文档使用 curl，如果没安装 curl 的话，先安装curl：
```sh
sudo apt install curl
```

安装deno：
```sh
curl -fsSL https://deno.land/x/install/install.sh | sh
```

安装成功提示：
```
Deno was installed successfully to /home/xxx/.deno/bin/deno
Manually add the directory to your $HOME/.bash_profile (or similar)
  export DENO_INSTALL="/home/xxx/.deno"
  export PATH="$DENO_INSTALL/bin:$PATH"
```

这个 “or similar” 好贴心！这个`.bash_profile`还真没有，有个 “similar” 的文件叫 `.bashrc`

```
cd ~
vim .bashrc 
```

再把提示里的那两句话粘贴到最下面就行：
```
export DENO_INSTALL="/home/xxx/.deno"
export PATH="$DENO_INSTALL/bin:$PATH"
```

然后，重新打开终端，已经可以使用deno命令了：

```sh
deno --version
```

# 参考
[deno_install](https://github.com/denoland/deno_install)