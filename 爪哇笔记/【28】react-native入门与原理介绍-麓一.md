# 课程目标

p6：

* 会使用 RN，了解RN同类别的产品，了解移动端的主要技术方案，有一定的跨端开发经验，踩过一些坑

p6+~p7：

* 知道如何与native进行数据交互，知道ios与安卓jsbridge实现原理
* 知道移动端webview和基础能力，包括但不限于：webview资源加载优化方案；webview池管理、独立进程方案；native路由等
* 能够给出完整的前后端对用户体系的整体技术架构设计，满足多业务形态用户体系统一。考虑跨域名、多组织架构、跨端、用户态开放等场景

其它：

* 把react以及跨端相关的知识点，一起串一下
  * 在窥探react- native原理的同时，总结reac 框架上的一些知识，同时了解一些跨端编译方面的知识

# 知识要点

## 移动端跨平台开发方案的演进

### 当前热点

在 2021 JavaScript Rising Star： [链接](https://risingstars.js.org/2021/en#section-mobile)

* **React Native**
* lonic
* **Expo**
* Quasar
* Flipper
* Flutter

### 演进历史

* Webview
* ReactNative（Weex）
* Flutter
* RN + Fabric

## 端与跨端

* 端：数据获取、状态管理、页面渲染。（FE三板斧）
* 跨端：虚拟机、渲染引擎、原生交互、开发环境。

### 1. 数据获取

还是老三样：fetch \ axios \ XHR

### 2. 状态管理

React中的state模式

### 3. 页面渲染

vDom -> yoga -> iOS / android / DOM APIs -> iOS / android / Web

### 4. 虚拟机

#### RN

* JSC - Objective-c / JS
* 到原生层，用了大量的bridge

#### Flutter

* JIT(dev) + AOT(prod)
* Skia
* 生态问题

### 5. 渲染引擎

yoga

### 6. 原生交互

JSBridge

### 7. 开发环境

## RN的原理

### React的设计理念

在运行时开发者能够处理 React JSX 的核心基础其实在于 **React 的设计理念**，React 将自身能力充分解耦，并提供给社区接入关键环节。这里我们需要先进行一些 React 原理解析。

React的整体设计理念可以分为三个部分：

* React Core
* React Renderer
* Reconciler

在这里我们需要了解的是：

自定义renderer --- 宿主配置**hostConfig** --- React reconciler --- react core

```js
HostConfig.getPublicInstance
HostConfig.getRootHostContext
HostConfig.getChildHostContext
HostConfig.prepareForCommit
HostConfig.resetAfterCommit
HostConfig.createInstance
HostConfig.appendInitialChild
HostConfig.finalizeInitialChildren
HostConfig.prepareUpdate
HostConfig.shouldSetTextContent
HostConfig.shouldDeprioritizeSubtree
HostConfig.createTextInstance
HostConfig.scheduleDeferredCallback
HostConfig.cancelDeferredCallback
HostConfig.setTimeout
HostConfig.clearTimeout
HostConfig.noTimeout
HostConfig.now
HostConfig.isPrimaryRenderer
HostConfig.supportsMutation
HostConfig.supportsPersistence
HostConfig.supportsHydration
// -------------------
//      Mutation
//     (optional)
// -------------------
HostConfig.appendChild
HostConfig.appendChildToContainer
HostConfig.commitTextUpdate
HostConfig.commitMount
HostConfig.commitUpdate
HostConfig.insertBefore
HostConfig.insertInContainerBefore
HostConfig.removeChild
HostConfig.removeChildFromContainer
HostConfig.resetTextContent
HostConfig.hideInstance
HostConfig.hideTextInstance
HostConfig.unhideInstance
HostConfig.unhideTextInstance
// -------------------
//     Persistence
//     (optional)
// -------------------
HostConfig.cloneInstance
HostConfig.createContainerChildSet
HostConfig.appendChildToContainerChildSet
HostConfig.finalizeContainerChildren
HostConfig.replaceContainerChildren
HostConfig.cloneHiddenInstance
HostConfig.cloneUnhiddenInstance
HostConfig.createHiddenTextInstance
// -------------------
//     Hydration
//     (optional)
// -------------------
HostConfig.canHydrateInstance
HostConfig.canHydrateTextInstance
HostConfig.getNextHydratableSibling
HostConfig.getFirstHydratableChild
HostConfig.hydrateInstance
HostConfig.hydrateTextInstance
HostConfig.didNotMatchHydratedContainerTextInstance
HostConfig.didNotMatchHydratedTextInstance
HostConfig.didNotHydrateContainerInstance
HostConfig.didNotHydrateInstance
HostConfig.didNotFindHydratableContainerInstance
HostConfig.didNotFindHydratableContainerTextInstance
HostConfig.didNotFindHydratableInstance
HostConfig.didNotFindHydratableTextInstance
```

附：

Taro 的包：https://github.com/NervJS/taro/tree/next/packages/taro-react

native的包：https://github.com/facebook/react/blob/main/packages/react-native-renderer/src/ReactNativeHostConfig.js

Dom 的包：https://github.com/facebook/react/blob/main/packages/react-dom/src/client/ReactDOMHostConfig.js

ART 的包：https://github.com/facebook/react/blob/main/packages/react-art/src/ReactARTHostConfig.js

### RN原理

#### 注册与发布

`AppRegistry`是所有React Native应用的JS入口。应用的根组件应当通过`AppRegistry.registerComponent`方法注册自己，然后原生系统才可以加载应用的代码包并且在启动完成之后通过调用`AppRegistry.runApplication`来真正运行应用。

```js
AppRegistry.registerComponent(appName, () => App);
```

#### Libraries/ReactNative/AppRegistry.js

```js
registerComponent(
    appKey: string,
    componentProvider: ComponentProvider,
    section?: boolean,
): string {
    let scopedPerformanceLogger = createPerformanceLogger();
    runnables[appKey] = {
        componentProvider,
        run: (appParameters, displayMode) => {
            const concurrentRootEnabled =
                  appParameters.initialProps?.concurrentRoot ||
                  appParameters.concurrentRoot;
            /***********************/
            renderApplication(
                componentProviderInstrumentationHook(
                    componentProvider,
                    scopedPerformanceLogger,
                ),
                appParameters.initialProps,
                appParameters.rootTag,
                wrapperComponentProvider && wrapperComponentProvider(appParameters),
                appParameters.fabric,
                showArchitectureIndicator,
                scopedPerformanceLogger,
                appKey === 'LogBox',
                appKey,
                coerceDisplayMode(displayMode),
                concurrentRootEnabled,
            );
        },
    };
    if (section) {
        sections[appKey] = runnables[appKey];
    }
    return appKey;
}

runApplication(
    appKey: string,
    appParameters: any,
    displayMode?: number,
): void {
    if (appKey !== 'LogBox') {
        const logParams = __DEV__
        	? '" with ' + JSON.stringify(appParameters)
        	: '';
        const msg = 'Running "' + appKey + logParams;
        infoLog(msg);
        BugReporting.addSource(
            'AppRegistry.runApplication' + runCount++,
            () => msg,
        );
    }
    invariant(
    	runnables[appKey] && runnables[appKey].run,
        `"${appKey}" has not been registered. This can happen if:\n` +
        '* Metro (the local dev server) is run from the wrong folder. ' +
        'Check if Metro is running, stop it and restart it in the current project.\n' +
        "* A module failed to load due to an error and `AppRegistry.registerComponent` wasn't called.",
    );

    SceneTracker.setActiveScene({name: appKey});
    runnables[appKey].run(appParameters, displayMode);
}
```

#### Libraries/ReactNative/renderApplication.js

```js
function renderApplication<Props: Object>(
    RootComponent: React.ComponentType<Props>,
    initialProps: Props,
    rootTag: any,
    WrapperComponent?: ?React.ComponentType<any>,
    fabric?: boolean,
    showArchitectureIndicator?: boolean,
    scopedPerformanceLogger?: IPerformanceLogger,
    isLogBox?: boolean,
    debugName?: string,
    displayMode?: ?DisplayModeType,
    useConcurrentRoot?: boolean,
) {
    invariant(rootTag, 'Expect to have a valid rootTag, instead got ', rootTag);

    const performanceLogger = scopedPerformanceLogger ?? GlobalPerformanceLogger;

    let renderable = (
        <PerformanceLoggerContext.Provider value={performanceLogger}>
            <AppContainer
                rootTag={rootTag}
                fabric={fabric}
                showArchitectureIndicator={showArchitectureIndicator}
                WrapperComponent={WrapperComponent}
                initialProps={initialProps ?? Object.freeze({})}
                internal_excludeLogBox={isLogBox}
            >
                <RootComponent {...initialProps} rootTag={rootTag} />
            </AppContainer>
		</PerformanceLoggerContext.Provider>
	);

	if (__DEV__ && debugName) {
        const RootComponentWithMeaningfulName = getCachedComponentWithDebugName(
            `${debugName}(RootComponent)`,
        );
        renderable = (
            <RootComponentWithMeaningfulName>
            	{renderable}
            </RootComponentWithMeaningfulName>
        );
    }

	performanceLogger.startTimespan('renderApplication_React_render');
	performanceLogger.setExtra('usedReactFabric', fabric ? '1' : '0');
	if (fabric) {
        require('../Renderer/shims/ReactFabric').render(
            renderable,
            rootTag,
            null,
            useConcurrentRoot,
        );
    } else {
        /*****************************/
        require('../Renderer/shims/ReactNative').render(renderable, rootTag);
    }
	performanceLogger.stopTimespan('renderApplication_React_render');
}
```

#### Libraries/Renderer/implementations/ReactNativeRenderer-dev.js

```js
// 22976
function render(element, containerTag, callback) {
    var root = roots.get(containerTag);

    if (!root) {
        // TODO (bvaughn): If we decide to keep the wrapper component,
        // We could create a wrapper for containerTag as well to reduce special casing.
        root = createContainer(containerTag, LegacyRoot, false, null, false);
        roots.set(containerTag, root);
    }

    updateContainer(element, root, null, callback); // $FlowIssue Flow has hardcoded values for React DOM that don't work with RN

    return getPublicRootInstance(root);
}
// updateContainer
// scheduleUpdateOnFiber
// performSyncWorkOnRoot
// renderRootSync
// workLoopSync
// performUnitOfWork
// completeWork
// -HostComponent-createInstance
// -> 一直走到 createInstance

function createInstance(
	type,
    props,
    rootContainerInstance,
    hostContext,
    internalInstanceHandle
) {
    var tag = allocateTag();
    var viewConfig = getViewConfigForType(type);
    
    {
        for (var key in viewConfig.validAttributes) {
            if (props.hasOwnProperty(key)) {
                ReactNativePrivateInterface.deepFreezeAndThrowOnMutationInDev(
                    props[key]
                );
            }
        }
    }

    var updatePayload = create(props, viewConfig.validAttributes);
    /**************************************/
    ReactNativePrivateInterface.UIManager.createView(
        tag, // reactTag
        viewConfig.uiViewClassName, // viewName
        rootContainerInstance, // rootTag
        updatePayload // props
    );
    var component = new ReactNativeFiberHostComponent(
        tag,
        viewConfig,
        internalInstanceHandle
    );
    precacheFiberNode(internalInstanceHandle, tag);
    updateFiberProps(tag, props); // Not sure how to avoid this cast. Flow is okay if the component is defined
    // in the same file but if it's external it can't see the types.

    return component;
}
```

#### Renderer

### 一起实现一个render

其它的可参考资料：

https://www.awesome-react-native.com/

# 补充知识点