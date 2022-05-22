# plate

## createPlugins

可以通过该函数一次插入所有的插件，返回一个插件集合

第一个参数：设置插件，接受一个插件的数组

第二个参数：设置插件对应的UI 组件，可以使用createPlateUI()，

## PlatePlugin

`key`: 插件唯一的key，必要、唯一。命名约定：element以：` ELEMENT_`开头;mark以：`MARK_`开头

`componmnet`: react组件用来渲染element\leaf



## createPluginFactory

