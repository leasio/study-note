# 课程目标

p6：

* 掌握react-router使用方法
* 能够使用react-router设计开发react应用

p6+~p7：

* 理解react-router关键源码实现
* 理解react-router和vue-rouer的实现差异，针对面试提出的问题能举一反三

# 知识要点

## react-router基本使用

### （1）路由配置

jsx用法：

```jsx
import { render } from "react-dom";
import {
    BrowserRouter,
    Routes,
    Route,
} from "react-router-dom";
// import your route components too

render(
    <BrowserRouter>
        <Routes>
            <Route path="/" element={<App />}>
                <Route index element={<Home />} />
                <Route path="teams" element={<Teams />}>
                    <Route path=":teamId" element={<Team />} />
                    <Route path="new" element={<NewTeamForm />} />
                    <Route index element={<LeagueStandings />} />
                </Route>
            </Route>
        </Routes>
    </BrowserRouter>,
    document.getElementById("root")
);
```

config+hooks用法：

```tsx
import { render } from "react-dom";
import { useRoutes, BrowserRouter } from 'react-router-dom'

function Routes() {
    return useRoutes([
        {
            path: '/',
            element: <App>,
            children: [
                {
                    path: '/,
                    element: <Home />,
                },
                {
                    path: '/teams',
                    element: <Teams />,
                    children: [
                        {
                            path: ':tamId',
                            element: <Team />
                        }
                    ]
                }
            ]
        },
    ])
}
                
render(
	<BrowserRouter>
        <Routes />
    </BowserRouter>,
    document.getElementById("root")
)
```

### （2）页面跳转

jsx用法：

```tsx
<Link to="/">To</Link>
```

hooks用法：

```tsx
const location = useLocation()

location.push("/")
```

## 搭建基于react-router的应用

```tsx
import * as React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter } from "react-router-dom";
import Routes from "./routes";
import "./index.css";
import { AuthProvider } from "./auth";

ReactDOM.hydrate(
    <React.StrictMode>
        <AuthProvider>
            <BrowserRouter>
                <Routes />
            </BrowserRouter>
        </AuthProvider>
    </React.StrictMode>,
    document.getElementById("root")
);
```

### 面试1：react-router的登录校验实现思路

鉴权

```tsx
// auth.tsx
import { createContext, useContext, useEffect, useState } from "react";
import { Link, Navigate, useLocation } from 'react-router-dom';

interface AuthContext {
    user: any;
    login: (user: string) => void;
    logout: () => void;
}

const AuthContext = createContext<AuthContext>(null!);

export function AuthProvider({ children }: { children: React.ReactNode }) {
    const [user, setUser] = useState<any>(null);
    const [loading, setLoading] = useState(true);
    const getUserInfo = async () => {
        const user = await fetch("/api/userinfo").then(res => res.json());
        setUser(user.data);
        setLoading(false);
    }
    const login = async (newUser: string) => {
        await fetch('/api/login');
        await getUserInfo();
        return true;
    };

    const logout = async () => {
        await fetch('/api/logout');
        setUser(null);
        return true;
    };

    useEffect(() => {
        getUserInfo()
    }, [])
  
    const value = { user, login, logout };

    return loading ? 'loading...' : <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function RequireAuth({ children }: { children: JSX.Element }) {
    let auth = useAuth();
    let location = useLocation();

    if (!auth.user) {
        return <Navigate to="/login" state={{ from: location }} replace />;
    }

    return children;
}

export function useAuth() {
    return useContext(AuthContext);
}

export function useAuthStatus() {
    const auth = useAuth();

    if (!auth.user) {
        return <Link to="/login">登录</Link>
    }

    return <Link to="/logout">退出登录</Link>
}
```

```tsx
// routes.tsx
import { useRoutes } from "react-router"
import { RequireAuth } from "./auth"
import Layout from "./layouts/Layout"
import About from "./pages/About"
import Account from "./pages/Account"
import Home from "./pages/Home"
import Login from "./pages/Login"
import Logout from "./pages/Logout"

export const routes = [
    {
        path: '/',
        name: '首页',
        element: <Home />,
        nav: true,
    },
    {
        path: '/about',
        name: '关于我们',
        element: <About />,
        nav: true,
    },
    {
        path: '/login',
        name: '登录',
        element: <Login />,
    },
    {
        path: '/logout',
        name: '退出登录',
        element: <Logout />
    },
    {
        path: '/account',
        name: '我的账户',
        element: <Account />,
        auth: true,
        nav: true,
    }
]

function Routes () {
    const r = useRoutes([
        {
            path: '/',
            element: <Layout />,
            children: routes.map(e => {
                if (e.auth) {
                    e.element = (
                        <RequireAuth>
                            {e.element}
                        </RequireAuth>
                    )
                }
                return e;
            }),
        }
    ]);
    return r;
}

export default Routes;
```



### 面试2：react-router懒加载的实现思路

React.lazy

```tsx
import * as React from "react";
import ReactDOM from "react-dom";
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Layout from "./layouts/Layout"

const Account = React.lazy(() => import("./pages/Account"));

ReactDOM.hydrate(
    <React.StrictMode>
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Layout />}>
                    <Route path="/account" element={
                        <React.Suspense fallback={(<div>loading...</div>)}>
                            <Account />
                        </React.Suspense>
                    } />
                </Route>
            </Routes>
        </BrowserRouter>
    </React.StrictMode>
);
```

### 面试3：在react-router v6中如何实现离开页面前确认

navigator.block

```ts
// usePrompt.ts
import { useCallback, useContext, useEffect, useState } from 'react';
import { UNSAFE_NavigationContext, useLocation, useNavigate } from 'react-router-dom';
import type { History, Transtition } from 'history';

export function usePrompt(when: boolean): (boolean | (() => void))[] {
    const navigate = useNavigate();
    const location = useLocation();
    const [showPrompt, setShowPrompt] = useState(false);
    const [lastLocation, setLastLocation] = useState<any>(null);
    const [confirmedNavigation, setConfirmedNavigation] = useState(false);
    
    const cancelNavigation = useCallback(() => {
        setShowPrompt(false);
    }, [])
    
    const handleBlockedNavigation = useCallback(
        (naxtLocation) => {
            if (
                !confirmedNavigation &&
                naxtLocation.location.pathname !== location.pathname
            ) {
                setShowPrompt(true);
                setLastLocation(nextLocation);
                return false;
            }
            return true;
        }, 
        [confirmedNavigation]
    );
    
    const confirmNavigation = useCallback(() => {
        setShowPrompt(false);
        setConfirmedNavigation(true);
    }, []);
    
    useEffect(() => {
        if (confirmedNavigation && lastLocation) {
            navigate(lastLocation.location.pathname);
        }
    }, [confirmedNavigation, lastLocation]);
    
    const navigator = useContext(UNSAFE_NavigationContext)
    	.navigator as History;
    
    useEffect(() => {
        if (!when) return;
        
        const unblock = navigator.block((tx: Transition) => {
            const autoUnblockingTx = {
                ...tx,
                retry() {
                    unblock();
                    tx.retry();
                }
            };
            
            handleVlockedNavigation(autoUnblockingTx);
        });
        
        return unblock;
    }, [navigator, handleBlockedNavigation, when]);
    
    return [showPrompt, confirmNavigation, cancelNavigation];
}
```

```tsx
// 弹框组件
import { Button, Modal } from 'react-bootstrap'

interface DialogBoxProps {
    showDialog: boolean
    cancelNavigation: any
    confirmNavigation: any
}

const DialogBox: React.FC<DialogBoxProps> = ({
    showDialog,
    cancelNavigation,
    confirmNavigation,
}) => {
    return (
        <Modal show={showDialog}>
            <Modal.Header>
                <Modal.Title>警告</Modal.Title>
            </Modal.Header>
            <Modal.Body>
                <b>确认离开页面吗</b>
                <br /> 离开页面后您填写的内容将全部丢失
            </Modal.Body>
            <Modal.Footer>
                <Button variant="primary" onClick={cancelNavigation}>
                    取消
                </Button>
                <Button variant="danger" onClick={confirmNavigation}>
                    确认
                </Button>
            </Modal.Footer>
        </Modal>
    )
}
export default DialogBox
```

```tsx
// 页面调用
import { useState } from "react";
import { usePrompt } from "../../hooks/usePrompt";
import DialogBox from "../../components/Dailog";

export default () => {
    const [showDialog, setShowDialog] = useState<boolean>(false)
    const [showPrompt, confirmNavigation, cancelNavigation] =
          usePrompt(showDialog)

    const handleChange = (event: any) => {
        setShowDialog(true)
    }
  
    return (
        <div>
            <DialogBox
                // @ts-ignore
                showDialog={showPrompt}
                confirmNavigation={confirmNavigation}
                cancelNavigation={cancelNavigation}
            />
            <input onChange={handleChange} />
        </div>
    )  
}
```

### 面试4：如何在服务端处理react-router

StaticRouter

```tsx
import * as React from "react";
import ReactDOMServer from "react-dom/server";
import { StaticRouter } from "react-router-dom/server";
import Routes from "./routes";

export function render(url: string) {
    return ReactDOMServer.renderToString(
        <React.StrictMode>
            <StaticRouter location={url}>
                <Routes />
            </StaticRouter>
        </React.StrictMode>
    );
}
```

### 面试5：react-router有哪些路由类型，及其实现原理和差异

* BrowserRouter
* HashRouter
* HistoryRouter

（原理见下面“核心源码解析”）

### 面试6：为什么要分react-router、react-router-dom，react-router-native这样实现的好处是什么

（详见下面“核心源码解析”）

### 面试7：react-router和vue-router有哪些异同

（详见下面“react-router和vue-router的差异”）

## 核心源码解析

我们经常使用的BrowserRouter和HashRouter主要依赖三个包：react-router-dom、react-router、history。

* **react-router**提供react路由的核心实现，是跨平台的。
* **react-router-dom**提供了路由在web端的具体实现，与之同级的还有react-router-native，提供react-native端的路由实现。
* **history**是一个对浏览器history操作封装，抹平浏览器history和hash的操作差异，提供统一的location对象给react-router-dom使用。

下面从最简单的例子，进入react-router源码解析：

```jsx
ReactDOM.render(
    <BrowserRouter>
        <App />
    </BrowserRouter>,
    document.getElementById('root')
)
```

BrowserRouter实现：

```jsx
// react-router-dom/index.tsx : 142
export function BrowserRouter({
    basename,
    children,
    window,
}: BrowserRouterProps) {
    let historyRef = React.useRef<BrowserHistory>();
    if (historyRef.current == null) {
        // 如果之前没有创建过，则创建history对象，createBrowserHistory是history包中实现的方法
        historyRef.current = createBrowserHistory({ window });
    }
    
    let history = historyRef.current;
    let [state, setState] = React.useState({
        action: history.action,
        location: history.location,
    });
    // history变化时重新渲染
    React.useLayoutEffect(() => history.listen(setState), [history]);
    
    return (
        <Router
            basename={basename}
            children={children}
            location={state.location}
            navigationType={state.action}
            navigator={history}
        />
    );
}
```

BrowserRouter包装了<Router />，并创建了history对象，监听history变化。

Router实现：

```jsx
export function Router({
    basename: basenameProp = "/",
    children = null,
    location: locationProp,
    navigationType = NavigationType.Pop,
    navigator,
    static: staticProp = false,
}: RouterProps): React.ReactElement | null {
  
    let basename = normalizePathname(basenameProp);
    let navigationContext = React.useMemo(
        () => ({ basename, navigator, static: staticProp }),
        [basename, navigator, staticProp]
    );
    
    if (typeof locationProp === "string") {
        locationProp = parsePath(locationProp);
    }
    
    let {
        pathname = "/",
        search = "",
        hash = "",
        state = null,
        key = "default",
    } = locationProp;
    
    let location = React.useMemo(() => {
        let trailingPathname = stripBasename(pathname, basename);
        
        if (trailingPathname == null) {
            return null;
        }
        
        return {
            pathname: trailingPathname,
            search,
            hash,
            state,
            key,
        };
        // 当location中有如下变化时重新⽣成location对象，触发⻚⾯重新渲染
 }, [basename, pathname, search, hash, state, key]);
    if (location == null) {
        return null;
    }
    // const LocationContext = React.createContext<LocationContextObject>(null!);
    /**
     创建了全局的context，⽤于存放history对象
    */
    return (
        <NavigationContext.Provider value={navigationContext}>
            <LocationContext.Provider
                children={children}
                value={{ location, navigationType }}
            />
        </NavigationContext.Provider>
    );
}
```

Router处理了history对象，对其用NavigationContext包裹，使得下层子组件都可以访问到这个history。

Routes实现：

```jsx
export function Routes({
    children,
    location,
}: RoutesProps): React.ReactElement | null {
    return useRoutes(createRoutesFromChildren(children), location);
}

export function useRoutes(
	routes: RouteObject[],
    locationArg?: Partial<Location> | string
): React.ReactElement | null {
    let { matches: parentMatches } = React.useContext(RouteContext);
    let routeMatch = parentMatches[parentMatches.length - 1];
    let parentParams = routeMatch ? routeMatch.params : {};
    let parentPathname = routeMatch ? routeMatch.pathname : "/";
    let parentPathnameBase = routeMatch ? routeMatch.pathnameBase : "/";
    let parentRoute = routeMatch && routeMatch.route;
    let locationFromContext = useLocation();
    let location;
    if (locationArg) {
        let parsedLocationArg =
            typeof locationArg === "string" ? parsePath(locationArg) : locationArg;
        
        location = parsedLocationArg;
    } else {
        location = locationFromContext;
    }
    let pathname = location.pathname || "/";
    let remainingPathname =
        parentPathnameBase === "/"
    		? pathname
    		: pathname.slice(parentPathnameBase.length) || "/";
    let matches = matchRoutes(routes, { pathname: remainingPathname });
    
    return _renderMatches(
        matches &&
        	matches.map((match) =>
            	Object.assign({}, match, {
                	params: Object.assign({}, parentParams, match.params),
                	pathname: joinPaths([parentPathnameBase, match.pathname]),
                	pathnameBase:
                		match.pathnameBase === "/"
                			? parentPathnameBase
                			: joinPaths([parentPathnameBase, match.pathnameBase]),
            	})
            ),
        parentMatches
    );
}

export function createRoutesFromChildren(
	children: React.ReactNode
): RouteObject[] {
    let routes: RouteObject[] = [];
    
    React.Children.forEach(children, (element) => {
        if (!React.isValidElement(element)) {
            return;
        }
        
        if (element.type === React.Fragment) {
            routes.push.apply(
                routes,
                createRoutesFromChildren(element.props.children)
            );
            return;
        }
        let route: RouteObject = {
            caseSensitive: element.props.caseSensitive,
            element: element.props.element,
            index: element.props.index,
            path: element.props.path,
        };
        
        if (element.props.children) {
            route.children = createRoutesFromChildren(element.props.children);
        }
        
        routes.push(route);
    });
    
    return routes;
}
```

Routes通过Route子组件生成路由列表，通过location中的pathname匹配组件并渲染。

通过以上代码，基本理解了react-router如何感知history中的pathname变化，并渲染对应组件。但具体是如何操作history变化的呢？

在回到最上面的createBrowserHistory和history.listen方法，看看history对象是怎么被创建和改变的：

```jsx
// history/packages/history/index.ts: 364
export function createBrowserHistory(
	options: BrowserHistoryOptions = {}
): BrowserHistory {
    let { window = document.defaultView! } = options;
    let globalHistory = window.history;
      
    // 获取当前history中的index和location对象
    function getIndexAndLocation(): [number, Location] { }
    
    let blockedPopTx: Transition | null = null;
  
    // 处理返回上⼀⻚
    function handlePop() {}
    
    // 监听浏览器的popState事件
    window.addEventListener(PopStateEventType, handlePop);
  
    // 操作history对象
    function applyTx(nextAction: Action) {
        action = nextAction;
        [index, location] = getIndexAndLocation();
        listeners.call({ action, location });
    }
    // push操作
    function push(to: To, state?: any) {}
    
    // replace操作
    function replace(to: To, state?: any) {}
    
    // 前进或后退n⻚
    function go(delta: number) {}
    
    let history: BrowserHistory = {
        get action() {
            return action;
        },
        get location() {
            return location;
        },
        createHref,
        push,
        replace,
        go,
        back() {
            go(-1);
        },
        forward() {
            go(1);
        },
        listen(listener) {
            return listeners.push(listener);
        },
        block(blocker) {
            let unblock = blockers.push(blocker);
            
            if (blockers.length === 1) {
                window.addEventListener(BeforeUnloadEventType, promptBeforeUnload);
            }
            
            return function () {
                unblock();
                if (!blockers.length) {
                    window.removeEventListener(BeforeUnloadEventType, promptBeforeUnload);
                }
            };
        },
    };
    
    return history;
}
```

createBrowserHistory创建了一个标准的history对象，以及对history对象操作的各方法，且操作变更后，通过listen方法将变更结果回调给外部。

getIndexAndLocation实现：

```jsx
function getIndexAndLocation(): [number, Location] {
    let { pathname, search, hash } = window.location;
    let state = globalHistory.state || {};
    return [
        state.idx,
        readOnly<Location>({
            pathname,
            search,
            hash,
            state: state.usr || null,
            key: state.key || "default",
        }),
    ];
}
```

push实现：

```jsx
function push(to: To, state?: any) {
    let nextAction = Action.Push;
    let nextLocation = getNextLocation(to, state);
    function retry() {
        push(to, state);
    }
    
    if (allowTx(nextAction, nextLocation, retry)) {
        let [historyState, url] = getHistoryStateAndUrl(nextLocation, index + 1);
        
        try {
            // 操作浏览器的history
            globalHistory.pushState(historyState, "", url);
        } catch (error) {
            window.location.assign(url);
        }
        // 处理history对象并回调
        applyTx(nextAction);
        /**
          function applyTx(nextAction: Action) {
          	action = nextAction;
          	[index, location] = getIndexAndLocation();
          	listeners.call({ action, location });
          }
        */
    }
}
```

问：我们在代码中都是const location = useLocation(); location.push("/")这样的方式使用push的，那上面这个push方法到底是怎么跟useLocation关联的呢？

还记得Router中有这么一段代码吗？

```jsx
const LocationContext = React.createContext<LocationContextObject>(null!);
```

我们的history对象创建后会被Router注入进一个LocationContext的全局上下文中。

useLocation实际就是包裹了这个上下文对象。

```jsx
export function useLocation(): Location {
    return React.useContext(LocationContext).location;
}
```

总结一下，BrowserRouter核心实现包含三部分：

* 创建history对象，提供对浏览器history对象的操作。
* 创建Router组件，将创建好的history对象注入全局上下文。
* Routes组件，遍历子组件生成路由表，根据当前全局上下文history对象中的pathname匹配当前激活的组件并渲染。

HashRouter和BrowserRouter原理类似，只是监听的浏览器原生history从pathname变成hash，这里不再赘述。

## react-router和vue-router的差异

### （1）路由类型

React：

* borwserRouter
* hashRouter
* memoryRouter

Vue：

* history
* hash
* abstract

memoryRouter和abstract作用类似，都是在不支持浏览器的环境中充当fallback。

### （2）使用方式

#### 路由拦截的实现不同

​		vue router提供了全局守卫、路由守卫、组件守卫供我们实现路由拦截。

​		react router没有提供类似vue router的守卫供我们使用，不过我们可以在组件渲染过程中自己实现路由拦截。如果是类组件，我们可以在componentWillMount或者getDerivedStateFromProps中通过props.history实现路由拦截；如果是函数式组件，在函数方法中通过props.history或者useHistory返回的history对象实现路由拦截。

### （3）实现差异

#### hash模式的实现不同

​		react router的hash模式，是基于window.location.hash(window.location.replace)和hashchange实现的。当通过push方式跳转页面时，直接修改window.location.hash，然后渲染页面；当通过replace方式跳转页面时，会先构建一个修改hash以后的临时url，然后使用这个临时url通过window.location.replace的方式替换当前url，然后渲染页面；当激活历史记录导致hash发生变化时，触发hashchange事件，重新渲染页面。

​		vue router的hash模式，是先通过pushState(replaceState)在浏览器中新增（修改）历史记录，然后渲染页面。当激活某个历史记录时，触发popstate事件，重新渲染页面。如果浏览器不支持pushState，才会使用window.location.hash(window.location.replace)和hashchange实现。

#### history模式不支持pushState的处理方式不同

​		使用react router时，如果history模式下不支持pushState，会通过重新加载页面（window.locaton.href = href）的方式实现页面跳转。

​		使用vue router时，如果history模式下不支持pushState，会根据fallback配置项来进行下一步处理。如果fallback为true，回退到hash模式；如果fallback为false，通过重新加载页面的方式实现页面跳转。

#### 懒加载实现过程不同

​		vue router在路由懒加载过程中，会先去获取懒加载页面对应的js文件。等懒加载页面对应的js文件加载并执行完毕，才会开始渲染懒加载页面。

​		react router在路由懒加载过程中，会先去获取懒加载页面对应的js文件，然后渲染loading页面。等懒加载页面对应的js文件加载并执行完毕，触发更新，渲染懒加载页面。
