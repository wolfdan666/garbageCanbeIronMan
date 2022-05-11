# 技术

## 看work with go并动手写每一行代码
110分钟 + 90分钟

下次从这里开始:

https://mkaz.blog/working-with-go/sorting/

在测试go写文件的时候，发现自己用remote-wsl...竟然在编辑windows共享目录的文件，仅仅只是用了wsl的go开发环境

然后导致go写出来的文件权限全是777，网上看了一下解决方案，有点麻烦，只写`~/.vscode-server/server-env-setup`这个文件，还不能成功，

所以决定还是用remote-wsl编辑wsl上的文件，把我学习的代码mv到wsl文件系统上面去，这里需要`chmod 777 /root`, 否则windows文件写不进去

并且还不能在软链接的目录下mv，否则还是写不进去，要在外面mv

```zsh
 ⚡ 05/11|18:07:23  /  chmod 777 /root 
 ⚡ 05/11|18:11:30  /  cd -           
~/code/go_learn
 ⚡ 05/11|18:11:34  go_learn  mv workWithGo ../code/
mv: cannot move 'workWithGo' to '../code/': Permission denied
 ✘ ⚡ 05/11|18:11:49  go_learn  pwd
/root/code/go_learn
 ⚡ 05/11|18:11:53  go_learn  ll
total 0
drwxrwxrwx 1 root root 4.0K May  7 18:17 gopl.io
drwxrwxrwx 1 root root 4.0K May 10 21:22 workWithGo
 ⚡ 05/11|18:12:05  go_learn  cd ..
 ⚡ 05/11|18:12:20  code  pwd
/root/code
 ⚡ 05/11|18:12:22  code  mv go_learn/workWithGo ./
mv: setting attribute 'system.wsl_case_sensitive' for 'system.wsl_case_sensitive': Invalid argument
mv: setting attribute 'system.wsl_case_sensitive' for 'system.wsl_case_sensitive': Invalid argument
 ⚡ 05/11|18:12:38  code  ll
total 0
drwxr-xr-x 1 root root 4.0K May 10 14:55 estor-server
drwxr-xr-x 1 root root 4.0K May 11 17:33 estorgosdk
lrwxrwxrwx 1 root root   42 May 10 20:57 go_learn -> /mnt/c/Users/shanl/Documents/Code/go_learn
drwxrwxrwx 1 root root 4.0K May 10 21:22 workWithGo
 ⚡ 05/11|18:13:50  code  
```

到外面就正常了
```zsh
 ⚡ 05/11|18:15:53  workWithGo  umask
022
 ⚡ 05/11|18:15:56  workWithGo  touch test
 ⚡ 05/11|18:16:01  workWithGo  ll
total 0
drwxrwxrwx 1 root root 4.0K May 11 18:05 code
-rw-r--r-- 1 root root    0 May 11 18:16 tes
```

# 运动
10分钟

# 总结
加2个自信币和3.5个自信点 加1.5个支线未来币