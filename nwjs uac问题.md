---
title: nwjs uac问题
tags: [nwjs, uac, node-webkit]
categories: [nwjs]
---
### uac
[uac](http://baike.baidu.com/view/750250.htm), 平常我们见的小盾牌就是它了, 如果我们程序用了需要管理员权限的东西, 那么我们需要加入管理员权限需求. nwjs本身是不需要uac的，但是我们用的第三方插件可能用到了管理员权限, 这个时候我们需要加入uac.
<!--more-->

### 加入uac
在manifest文件里面要加几行代码
```
<security>
  <requestedPrivileges>
    <requestedExecutionLevel level='requireAdministrator' uiAccess='true' />
  </requestedPrivileges>
</security>
```

### nwjs加入uac
编译的时候生成的nw.exe.manifest文件是动态生成的, 如果我们在src/content/nw/src下直接修改nw.exe.manifest文件， 并不能起作用。编译的时候控制uac是某个配置文件里面写的, 具体大约是
```
F:\node-webkit-dev\depot_tools\python276_bin\python.exe gyp-win-tool link-with-manifests environment.x86 True nw.exe "F:\node-webkit-dev\depot_tools\python276_bin\python.exe gyp-win-tool link-wrapper environment.x86 False link.exe /nologo /OUT:nw.exe @nw.exe.rsp" 1 mt.exe rc.exe "obj\content\nw.nw.exe.intermediate.manifest" obj\content\nw.nw.exe.generated.manifest ..\..\content\nw\src\nw.exe.manifest..\..\build\win\compatibility.manifest                       
```

### ~~取巧~~
~~上面代表了什么意思，要去解析太麻烦, 但是我们可以看到最后生成了nw.nw.exe.generated.manifest文件, 这个文件是最后生成uac的文件. 所以我们在编译完成之后, 修改这个文件，然后再次编译就好了（增量编译, 这个文件不会再次生成）~~

上面错了：这样做了之后, 生成的exe的确带了uac的标签, 但是实际上启动之后, 有几个进程直接不能起来, 也就是说这个exe去启动子进程的时候, 子进程没有admin权限. 所以错了.

当然还有另一种取巧的方法：直接通过mt.exe修改nw.exe

nw.exe.mainfest 修改权限
```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
	<trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
		<security>
			<requestedPrivileges>
				<requestedExecutionLevel level="requireAdministrator" uiAccess="false"></requestedExecutionLevel>
			</requestedPrivileges>
		</security>
	</trustInfo>
</assembly>
```
```
mt.exe –manifest nw.exe.manifest -outputresource:nw.exe;
```


关于[mt.exe](http://msdn.microsoft.com/en-us/library/bb384691.aspx)








Good luck!
