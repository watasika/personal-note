# univer开发分享

## UI层面的开发

### 利用Preact实现一个组件

1. 声明type或interface

```tsx
export interface XXXProps {
    height: string;
}
```

2. 从univerjs中引入封装的preact的Component类并继承（这里从base-ui中引入），**需要实现render抽象方法以及一个initialize方法，contract构造函数直接继承** ，具体查看[源文件](https://github.com/dream-num/univer/blob/dev/packages/base-ui/src/Framework/Preact.tsx) 

```tsx
import { Component, VNode } from '@univerjs/base-ui';
export class XXX extends Component<XXXProps>{
    initialize(props: XXXProps) {
        // 这里做一些初始化的事
    }
    render(): VNode {
        // 渲染函数
        return(<div>aaaa</div>)
    }
}
```
3. 挂载到页面上：
   - 通过全局上下文获得`_sheetUIPlugin  `
   - 获取实例管理器 `getComponentManager()` ，注册方法进行注册
   - `setSlot` 指定插入的位置

```tsx
this._sheetUIPlugin = plugin.getGlobalContext().getPluginManager().getRequirePluginByName<SheetUIPlugin>(SHEET_UI_PLUGIN_NAME);
...
this._sheetUIPlugin.getComponentManager().register(DataBindingModal.name, DataBindingModal);
        this._sheetUIPlugin.setSlot('main', {
            name: DATA_BINDING_PLUGIN_NAME + DataBindingModal.name,
            component: {
                name: DataBindingModal.name,
                props: {
                    getComponent: (ref: DataBindingModal) => this.getComponent(ref),
                },
            },
        });
```

###  其他方式实现一个组件

1. 声明type或interface

```tsx
export interface XXXProps {
    height: string;
}
```

2. 直接export导出一个函数，其中return了一个虚拟DOM树（VNode）。此处没有渲染函数（render），实际上并不能挂载到页面上。即该组件只用于声明，最终被其他文件导入并渲染。

```tsx
export const XXX = (props: XXXProps) => {
    return (
        <div style="display: flex;"></div>
    );
};
```

###  通过canvas画布进行渲染

> ***最终没有使用该部分的方法，所以其中一部分是靠推断出来的，不一定对，具体文件见[image插件](https://github.com/dream-num/univer/blob/dev/packages/sheets-plugin-image/src/OverGridImagePlugin.tsx)*** 
>
> **如果是通过内置命令创建的插件文件夹，其下的目录结构是这样的：** 
>
> │  index.ts # 插件出口
> │  main.ts # 插件调试预览入口
> │  preact.d.ts # preact 声明文件
> │  SheetPlugin.tsx # 插件核心导出类（实际名称会根据插件名称替换）
> │
> ├─Controller # 根据Model改变view状态；如果是preact，那就只涉及一些公共业务逻辑，修改view还是在preact的view里
> │      index.ts # 出口
> │      SpreadsheetController.ts # 具体的数据更新类（实际名称会根据插件名称替换）
> │
> ├─Basics # 基础工具函数、常量、接口（可选）
> │  ├─Const # 常量
> │  │      index.ts # 出口
> │  │
> │  ├─Enum # 枚举
> │  │      index.ts # 出口
> │  │
> │  ├─Interfaces # 接口
> │  │      index.ts # 出口
> │  │
> │  └─Shared # 公共方法
> │         index.ts # 出口
> │
> ├─Locale # 国际化
> │      en.ts # 英文国际化翻译文件
> │      index.ts # 出口
> │      zh.ts # 中文国际化翻译文件
> │
> ├─Model # 数据 CRDT
> │  │
> │  ├─Action # 由Command触发（可选）
> │  │      index.ts # 出口
> │  │
> │  ├─Apply # 修改数据（可选）
> │  │      index.ts # 出口
> │  │
> │  └─SpreadsheetModel # 各个Controller对应的数据模型
> │         index.ts # 出口
> │
> ├─Types # 声明文件
> │      index.d.ts # 具体的声明文件
> │
> └─View # 视图，dom和canvas的生成和管理
>  ├─Render # canvas 组件
>  │      index.ts # 出口
>  │
>  └─UI # DOM 组件
>         SpreadsheetButton.tsx # 具体的DOM组件（实际名称会根据插件名称替换）
>         SpreadsheetButton.module.less # SpreadsheetButton组件的样式文件（实际名称会根据插件名称替换）
>
> 
>
> **render文件夹下控制canvas组件的绘制，在项目中带有render字样的，大部分是canvas组件相关，与之相对，UI则是DOM组件相关。 ** 

1. 可基于base-render提供的基本组件进行扩展，至少包含构造函数

```tsx
import { Picture } from '@univerjs/base-render';
export class OverImageShape extends Picture {
    constructor() {
        // 这是构造函数
    }
    // 大部分继承的基本组件都有draw方法，部分是抽象函数需要具体实现
    protected _draw（）{}
}
```

2. 如果想要渲染上述的组件，需要在整个插件的入口文件（一般是XXXPlugin.tsx，在插件文件夹src根目录下）的 `onMounted` 声明周期中渲染。由于整个插件类是继承自Plugin基类的，基类有一个方法`pushToObserve` ，它可以把各种监视器推入一个闭包，相当于在全局注册一个事件。然后在生命周期中实例化自己这个插件类，这样的话势必会触发构造函数，但是构造函数里面其实不需要做什么，这样就完成渲染了。那么如何确定渲染的位置呢？

```tsx
onMounted(): void {
        this.pushToObserve('onChangeImageSize'); // 注册监视器
        this._render = new OverImageRender(this);
    }
```

3. 注册事件之后，如何调用？最开始注册的时候，这个事件只有名字等一些属性，不包含实现。在我们的render组件中，我们需要一个具体的触发点去调用`getObserver` 拿到对应的事件（在上下文的plugin中），链式调用`notifyObservers` 通知全局，这样就完成一个事件触发。同时这里确定渲染的位置，在mainScene上，有个`addObject` 方法，可以把当前实例挂载在这个类上。

```tsx
addOverImage(property: OverGridImageProperty): void {
        const shape = new OverImageShape(property);
		  // 这里挂载事件触发方式
        shape.on(EVENT_TYPE.PointerDown, () => {
            this._activeShape = shape;
            this._plugin.showOverImagePanel();
            this._plugin.getObserver('onActiveImage')!.notifyObservers(shape.getProperty());// 这里触发onActiveImage事件
        });
		  // 这里进行实际操作
        this._mainScene.addObject(shape, OverImageRender.LAYER_Z_INDEX);
    }
```

## 事件的挂载

### 在主表（render范围）进行事件挂载

1. 一般我们挂载事件在plugin文件的入口做，或者在controller里面做，但其实任何地方做都可以，因为拿到全局的上下文是一件比较容易的事。我们首先通过全局上下文拿到`_sheetPlugin` ，尽量在初始化的时候做，比如在onMounted生命周期（如果有的话），或者在构造函数中。

   ```tsx
   import { SheetPlugin } from '@univerjs/base-sheets';
   import { PLUGIN_NAMES } from '@univerjs/core';
   ...
   // 在构造函数中，拿到当前的plugin，从plugin身上拿到全局的上下文，进而获取SheetPlugin
   constructor(plugin: DataBindingPlugin) {
           this._plugin = plugin;
           this._sheetPlugin = plugin.getUniver().getCurrentUniverSheetInstance().context.getPluginManager().getRequirePluginByName<SheetPlugin>(PLUGIN_NAMES.SPREADSHEET);
   
       	  // 挂载事件
           this.installObservers();
       }
   ```

2. 从`_sheetPlugin` 身上拿到`mainComponent` ，这个就是所有单元格范围，不包括title部分，在其上可以调用`onPointerDownObserver` 的`add`方法，添加一个事件监视器，并在触发的时候调用回调参数。之所以能在`mainComponent` 中拿到，是因为它本质上是`Spreadsheet` 类，继承自祖先类[BaseObject](https://github.com/dream-num/univer/blob/dev/packages/base-render/src/BaseObject.ts) ，当然它身上还有其他事件方法可以挂载。

```tsx
installObservers() {
        const mainComponent = this._sheetPlugin.getMainComponent();
        mainComponent.onPointerDownObserver.add((evt: IPointerEvent | IMouseEvent) => {
            console.log('点击', evt);
            evt.preventDefault();
        });
    }
```

3. 成功挂载之后，可以在回调函数里面写想要做的事。这边是获取到了plugin，在plugin中返回了它的render类，拿render类上的方法去渲染canvas

> 该分享可能已过时