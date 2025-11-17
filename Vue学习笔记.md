文件按照功能划分比按照类型划分 (html, css, js)更高效

header main footer

![](assets/Vue学习笔记/file-20251116170524362.png)

使用子组件的流程，导入组件，components中注册组件，在template中使用

难点1：

isDone的完整传递过程

父组件传出done属性，子组件接收done属性。

但是子组件不可以直接将其应用在checkbox的属性中，因为子组件只应该读父组件传的props，不应该修改父组件传的props，要保持单向流动。

因此子组件复制一份done的值到isDone

这样就可以指定:checked = "isDone"



