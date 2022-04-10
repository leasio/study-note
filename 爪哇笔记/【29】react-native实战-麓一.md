# 课程目标

p6：

* 会使用 RN，了解RN同类别的产品，了解移动端的主要技术方案，有一定的跨端开发经验，踩过一些坑

p6+~p7：

* 知道如何与native进行数据交互，知道ios与安卓jsbridge实现原理
* 知道移动端webview和基础能力，包括但不限于：webview资源加载优化方案；webview池管理、独立进程方案；native路由等
* 能够给出完整的前后端对用户体系的整体技术架构设计，满足多业务形态用户体系统一。考虑跨域名、多组织架构、跨端、用户态开放等场景

# 知识要点

## 部署相关

### SDK

* Node.js 12+
* 打包
  * metro的打包工具，Facebook出品，类似于webpack，需要启动对应的项目开发，xcode/vscode
  * expo-cli，社区的。原生应用复杂一些，简化了整个的开发流程，能够快速地找到这个expo应用，扫码就能够快速地在真机上进行预览
  * expo-cli更全面的一个重构，帮助我们在web端，同时进行展示
  * 将react开发，集成了一个大的套件

### react-native，特点

* 有自己的标签，比如`<View />`，`<Text />`
* 样式必须使用camelCase，没有className，取而代之的是styleSheet
* 文本必须标明文本节点
* 所有的布局元素，都是`flex-box`，一般也不用直接声明是`display: flex`
* image必须要给定宽高
* flex默认的方向变成了column
* 滚动需要`scrollView`
* `transform`有一个特殊的配置方式
* 像素
  * `PixelRatio.getPixelSizeForLayoutSize(200)`
  * `PixelRatio.get() ==> 2.75`
  * RN的数值的计算方式，是按照你的大小/2.75这种方式来计算的
* 首行缩进上，没有`text-indent`，需要自己用字符来实现
* 表单：`e.target.value` -> `e.nativeEvent.text`

```js
import React from 'react';
import {
    Text,
    View,
    StyleSheet,
    Image,
    ScrollView,
    PixelRatio,
    Dimensions,
    TextInput,
} from 'react-native';

const styles = StyleSheet.create({
    container: {
        backgroundColor: '#eee',
        flexDirection: 'column',
    },
    title: {
        alignSelf: 'center',
        fontSize: 20,
        backgroundColor: '#00f5ff',
        width: 390,
    },
    title1: {
        alignSelf: 'center',
        fontSize: 20,
        transform: [{scale: 1.5}, {rotate: '-10deg'}],
    },
    img: {
        width: 200,
        height: 300,
    },
});

export default function App() {
    const [state, setState] = React.useState('');
    return (
        <View style={styles.container}>
            <Text>{PixelRatio.getPixelSizeForLayoutSize(200)}</Text>
            <Text>{PixelRatio.get()}</Text>
            <Text>{Dimensions.get('window').width}</Text>
            <TextInput onChange={e => setState(e.nativeEvent.text)} />
            <Text>{state}</Text>
            <Text style={styles.title}>Hello world</Text>
            <Text style={styles.title1}>Hello world</Text>
            <Image style={styles.img} source={require('./assets/1.webp')} />
            <ScrollView>
                {new Array(1000).fill(0).map((item, index) => (
                    <Text>teaching you {index} times</Text>
                ))}
            </ScrollView>
        </View>
    );
}
```

## 一个app至少包含哪几个部分

### 1. 界面

* 路由 *
* 开屏动画
* icon
* 像素方案 *
* 多媒体——图片、视频
* 弹窗、抽屉。。。

### 2.接口

* 状态管理
* 数据接口

### 3. 事件

* onPress
* ...

## 常用的一些包描述

### 1. navigation

```js
"@react-navigation/bottom-tabs": "^6.0.9",
"@react-navigation/material-top-tabs": "^6.0.6",
"@react-navigation/native": "^6.0.6",
"@react-navigation/native-stack": "^6.2.5",
"@react-navigation/stack": "^6.0.11",
"react-navigation": "^4.4.4",
```

### 2. icon

```js
"react-native-vector-icons": "^9.0.0",
  
//
  ○ react-native-vector-icons
  ○ react-native-iconfont-cli (转出可选，svg / iconfont)
  ○ react-native-svg
  ○ react-native-svg-url
```

### 3. redux

```js
"react-redux": "^7.2.6",
"redux": "^4.1.2",
"redux-thunk": "^2.4.1"
```

### 4. 自动刷新列表

`<FlatList />`

```js
"react-native-refresh-list-view": "^1.0.12",
```

### 5. charts

```js
"react-native-charts-wrapper": "^0.5.7",
```

## 解决方案

传统的**23寸的显示器**，**1920\*1080**分辨率的；——72像素每英寸ppi（Pixel Per Inch）

iphone——retina——各种手机的显示屏。1英寸的距离上，可以显示更多的点。144ppi；

——显示器的各个厂商就规定了一个值。

dpr：设备像素比；

### 图片方案

#### 1px方案

先放大，200% -> transform(scale(0.5))

svg

#### rem方案

rem是什么：根节点的font-size

flexible

```js
var rem = document.documentElement.clinetWidth / 10;
documentElement.style.fontSize = rem + "px";
// 1rem 变成了 视⼝的1/10;
// postcss  -> px2rem;
```

#### viewpoint

## 坑点

### 路由

* 安装，需要安装多个组件

### 刘海屏

* 安卓怎么办？

```js
<SafeAreaView style={styles.AndroidSafeArea}>
</SafeAreaView>

AndroidSafeArea: {
    flex: 1,
    backgroundColor: 'white',
    paddingTop: Platform.OS === 'android' ? StatusBar.currentHeight : 0,
},
```

## 实战

### 路由先行

### 构建APP.js，使用StatusBar

```js
import React, {Fragment} from 'react';
import {Text, View, StatusBar} from 'react-native';
import Router from './router';

export default function App() {
    return (
        <Fragment>
            <StatusBar
                barStyle={'dark-content'}
                backgroundColor="transparent"
                hidden={false}
            />
            <Router />
        </Fragment>
    );
}
```

### 编写Router文件，构建堆栈导航器

```jsx
import React from 'react';
import {View, Text} from 'react-native';

import {NavigationContainer} from '@react-navigation/native';
import {createStackNavigator} from '@react-navigation/stack';

import Bottom from '../pages/bottom/index';
import Detail from '../pages/detail/index';
import Search from '../pages/search/index';

const Stack = createStackNavigator();

export default function Router() {
    return (
        <NavigationContainer>
            {/* 堆栈导航器, 构建导航信息 */}
            <Stack.Navigator
                screenOptions={{
                    headerTitleAlign: 'center',
                }}
            >
                <Stack.Screen name="Bottom" component={Bottom} />
                <Stack.Screen name="Detail" component={Detail} />
                <Stack.Screen name="Search" component={Search} />
            </Stack.Navigator>
        </NavigationContainer>
    );
}
```

### Bottom跳转Detail，了解一下堆栈导航器

```js
import {Text, TouchableWithoutFeedback} from 'react-native';
import React, {Component} from 'react';

export default function Bottom(props) {
    return (
        <TouchableWithoutFeedback
            onPress={() => {
                props.navigation.navigate({name: 'Detail'});
            }}>
            <Text>bottom</Text>
        </TouchableWithoutFeedback>
    );
}
```

### 丰富路由内容

```js
import React from 'react';
import {View, Text, Platform} from 'react-native';

import {NavigationContainer} from '@react-navigation/native';
import {
    CardStyleInterpolators,
    createStackNavigator,
} from '@react-navigation/stack';

import Bottom from '../pages/bottom/index';
import Detail from '../pages/detail/index';
import Search from '../pages/search/index';

const Stack = createStackNavigator();

export default function Router() {
    return (
        <NavigationContainer>
            {/* 堆栈导航器, 构建导航信息 */}
            <Stack.Navigator
                screenOptions={{
                    headerTitleAlign: 'center',
                        headerStyle: {
                            ...Platform.select({
                                android: {
                                    elevation: 0,
                                    borderBottomWidth: 0,
                                },
                                ios: {
                                    shadowOpacity: 0,
                                },
                            }),
                        },
                        headerBackTitleVisible: false,
                        headerTitleStyle: {
                            color: '#000',
                        },
                        // 屏幕过度⽅式
                        cardStyleInterpolator: CardStyleInterpolators.forHorizontalIOS,
                }}>
                <Stack.Screen
                    name="Bottom"
                    options={{
                        title: '⾸⻚',
                    }}
                    component={Bottom}
                />
                <Stack.Screen name="Detail" component={Detail} />
                <Stack.Screen name="Search" component={Search} />
            </Stack.Navigator>
        </NavigationContainer>
    );
}
```

### Bottom底部导航栏

```js
import {
    Text,
    TouchableWithoutFeedback,
    TouchableHighlight,
    StyleSheet,
} from 'react-native';
import React, {Component} from 'react';
import {createBottomTabNavigator} from '@react-navigation/bottom-tabs';
import Icon from 'react-native-vector-icons/FontAwesome';

import Home from './subPages/home';
import DashBoard from './subPages/dash';
import News from './subPages/news';
import Self from './subPages/self';
import {getFocusedRouteNameFromRoute} from '@react-navigation/native';

const Tab = createBottomTabNavigator();

const styles = StyleSheet.create({
    icon: {
        width: 24,
        height: 24,
    },
    headerRight: {
        paddingRight: 15,
    },
});

export default function Bottom(props) {
    const headerRight = () => {
        // const {route} = props;
        // const routeName = getFocusedRouteNameFromRoute(route) ?? 'News';
        // if (routeName === 'News') {
        return (
            <TouchableHighlight
                style={styles.headerRight}
                onPress={() => props.navigation.navigate({name: 'Search'})}>
                <Icon name="search" color={'#724035'} size={14} />
            </TouchableHighlight>
        );
        // } else {
        //   return null;
        // }
    };
    return (
        <Tab.Navigator>
            <Tab.Screen
                name="Home"
                component={Home}
                options={{
                    headerShown: true,
                    title: '⾸⻚',
                    tabBarLabel: '⾸⻚',
                    tabBarIcon: ({color, size}) => (
                        <Icon name="home" color={color} size={size} />
                    ),
                }}
            />
            <Tab.Screen
                name="DashBoard"
                component={DashBoard}
                options={{
                    headerShown: true,
                    title: '控制台',
                    tabBarLabel: '控制台',
                    tabBarIcon: ({color, size}) => (
                        <Icon name="dashboard" color={color} size={size} />
                    ),
                }}
            />
            <Tab.Screen
                name="News"
                component={News}
                options={{
                    headerShown: true,
                    headerRight: headerRight,
                    title: '新闻',
                    tabBarLabel: '新闻',
                    tabBarIcon: ({color, size}) => (
                        <Icon name="newspaper-o" color={color} size={size} />
                    ),
                }}
            />
            <Tab.Screen
                name="Self"
                component={Self}
                options={{
                    headerShown: true,
                    title: '我的',
                    tabBarLabel: '我的',
                    tabBarIcon: ({color, size}) => (
                        <Icon name="user" color={color} size={size} />
                    ),
                }}
            />
        </Tab.Navigator>
    );
}
```

### Detail详情页面的开发

```js
import {Text, View, StyleSheet, Dimensions, Image} from 'react-native';
import React, {useEffect} from 'react';

export default function Detail({navigation, route}) {
    useEffect(() => {
        navigation.setOptions({
            headerTitle: route.params.title,
        });
    }, [navigation, route.params.title]);
    
    return (
        <View style={styles.container}>
            <Image
                style={styles.image}
                source={{
                    uri: route.params.uri,
                }}
            />
            
            <View style={styles.detail}>
                <Text style={styles.detail_text}>
                    &#12288;&#12288;{route.params.desc}
                </Text>
            </View>
        </View>
    );
}
const styles = StyleSheet.create({
    title: {
        alignSelf: 'center',
    },
    title_text: {
        fontSize: 18,
        color: '#000000',
        margin: 12,
    },
    image: {
        width: Dimensions.get('window').width,
        height: 200,
    },
    detail: {
        alignSelf: 'flex-start',
    },
    detail_text: {
        fontSize: 16,
        color: '#333333',
        margin: 6,
    },
});
```

#### 引入redux，处理store

```js
import {createStore, applyMiddleware} from 'redux';
import thunk from 'redux-thunk';
import {newsList} from './index';
const defaultState = {
    list: [
        {
            author: '滑头⻤⼤⼈',
            title: 'nage客户端开发做得好好的为什么离职了',
            desc: '拿到这份offer的时候，我就想，如果不是领导⾮要赶我⾛，我是⽆论如何都不会挪窝的，但没想到⾃⼰会先投降，做⼀个逃兵，毕竟⾃⼰从校招进来也不容易',
            uri: 'https://p9-juejin.byteimg.com/tos-cn-ik3u1fbpfcp/86da66548d1d45608dde745415e7ff98~tplv-k3u1fbpfcp-zoom-cropmark:1304:1304:1304:734.awebp?',
        },
    ],
};

const reducer = (state = defaultState, action) => {
    switch (action.type) {
        case 'LOADED': {
            return {list: action.payload};
        }
        case 'LOADING': {
            return defaultState;
        }
        default:
            return state;
    }
};

export const fetchList = () => (dispatch, getState) => {
    dispatch({type: 'LOADING'});
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(newsList);
        }, 300);
    }).then(payload => dispatch({type: 'LOADED', payload}));
};

export const store = createStore(reducer, applyMiddleware(thunk));
```

### index.js

```jsx
import React, {Fragment} from 'react';
import {Text, View, StatusBar} from 'react-native';
import Router from './router';

import {Provider} from 'react-redux';
import {store} from './data/store';

export default function App() {
    return (
        <Provider store={store}>
            {/* 界⾯上⽅的导航栏，包括 Wi-Fi、电池等信息，设置背景颜⾊、透明度等信息 */}
            <StatusBar
                barStyle={'dark-content'}
                backgroundColor="transparent"
                hidden={false}
            />
            <Router />
        </Provider>
    );
}
```

# 补充知识点

## 一些资料

[React Native](https://github.com/crazycodeboy/RNStudyNotes)

[图标库](https://oblador.github.io/react-native-vector-icons/)

[第三方](https://www.awesome-react-native.com/#Components-UI)

[仿-天眼app](https://github.com/MarnoDev/react-native-eyepetizer)

[30天学React Native](https://github.com/fangwei716/30-days-of-react-native)

## react全家桶

### react、react-dom、react-reconciler、scheduler

### 状态管理

redux / mobx --- vuex

redux中间件：redux-thunk redux-saga

### 组件集

antd / rc-tree ... --- element ui / element plus

### router

react-router react-router-dom --- vue-router

### hooks

react-use

webpack --- webpack / vite

### http API

axios / fetch / xhr