---

title: "Manage-You-Java-Version"
date: 2017-10-14
categories: Java Version Tools 

layout: post
comments: true

---

### 干货如下

添加以下脚本到~/.bash_profile配置文件中：

```
function setjdk() {
    export JAVA_HOME=`/usr/libexec/java_home -v $@`
}
```

输入命令

```
setjdk 1.8
```




