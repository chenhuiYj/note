
## 一、场景问题描述
中后台项目常见权限场景：
1. 页面级权限控制（未登录拦截、无权限拦截）
2. 按钮级权限控制（不同角色显示/隐藏操作按钮）
3. Token 过期、登录态全局管理
4. 接口 401 / 403 统一拦截跳转登录
5. 多角色、动态菜单、路由权限控制

## 二、核心解题思路
1. 采用 **RBAC 权限模型**：用户 -> 角色 -> 权限码
2. 全局统一管理登录态、Token、角色、权限列表（Context / Redux / Zustand）
3. 封装高阶路由守卫组件，统一做页面访问拦截
4. 封装权限按钮组件，通过权限码精准控制元素渲染
5. Axios 全局拦截器统一处理 Token 携带、过期失效
6. 退出登录清空本地缓存与全局状态

## 三、完整代码实现

### 1. 全局权限上下文 AuthContext.jsx
```jsx
import { createContext, useContext, useReducer, useEffect } from 'react';

const initialState = {
  token: localStorage.getItem('token') || null,
  userInfo: null,
  roles: [],
  permissions: [],
  isLoading: true
};

const authReducer = (state, action) => {
  switch (action.type) {
    case 'SET_USER':
      return {
        ...state,
        token: action.payload.token,
        userInfo: action.payload.userInfo,
        roles: action.payload.roles,
        permissions: action.payload.permissions,
        isLoading: false
      };
    case 'LOGOUT':
      return { ...initialState, token: null, isLoading: false };
    default:
      return state;
  }
};

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialState);

  useEffect(() => {
    const initUser = async () => {
      if (!state.token) {
        dispatch({ type: 'LOGOUT' });
        return;
      }
      try {
        const mockUserInfo = {
          userId: 1,
          username: '管理员',
          roles: ['admin'],
          permissions: ['user:list', 'user:add', 'user:edit', 'user:del']
        };
        dispatch({
          type: 'SET_USER',
          payload: { ...mockUserInfo, token: state.token }
        });
      } catch (err) {
        dispatch({ type: 'LOGOUT' });
      }
    };
    initUser();
  }, [state.token]);

  const login = (userData) => {
    localStorage.setItem('token', userData.token);
    dispatch({ type: 'SET_USER', payload: userData });
  };

  const logout = () => {
    localStorage.removeItem('token');
    dispatch({ type: 'LOGOUT' });
  };

  return (
    <AuthContext.Provider value={{ ...state, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```
2. 路由守卫组件 AuthRoute.jsx（页面权限）
```jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

/**
 * 页面权限拦截组件
 * @param {*} children 页面组件
 * @param {string} requirePerms 当前页面所需权限码
 */
export const AuthRoute = ({ children, requirePerms }) => {
  const { token, permissions, isLoading } = useAuth();

  if (isLoading) return <div>加载中...</div>;

  // 未登录 → 跳转登录
  if (!token) {
    return <Navigate to="/login" replace />;
  }

  // 无页面权限 → 跳转403
  if (requirePerms && !permissions.includes(requirePerms)) {
    return <Navigate to="/403" replace />;
  }

  return children;
};
```
3. 权限按钮组件 AuthButton.jsx（按钮权限）
```jsx
import { useAuth } from './AuthContext';

/**
 * 按钮级权限控制
 * @param {string} code 权限码
 */
export const AuthButton = ({ code, children }) => {
  const { permissions } = useAuth();
  // 包含对应权限码才渲染
  return permissions.includes(code) ? children : null;
};
4. 登录页 Login.jsx
jsx
import { useNavigate } from 'react-router-dom';
import { useAuth } from './AuthContext';

export default function Login() {
  const navigate = useNavigate();
  const { login } = useAuth();

  const handleLogin = () => {
    // 模拟后端返回登录信息
    const res = {
      token: 'admin-token-20250001',
      userInfo: { username: '超级管理员' },
      roles: ['admin'],
      permissions: ['user:list', 'user:add', 'user:edit', 'user:del']
    };
    login(res);
    navigate('/user');
  };

  return (
    <div style={{ padding: '30px' }}>
      <h3>系统登录</h3>
      <button onClick={handleLogin}>模拟管理员登录</button>
    </div>
  );
}
```
5. 业务页面 UserPage.jsx
```jsx
import { useAuth } from './AuthContext';
import { AuthButton } from './AuthButton';

export default function UserPage() {
  const { logout } = useAuth();

  return (
    <div style={{ padding: '30px' }}>
      <h3>用户管理页面</h3>

      {/* 权限按钮 */}
      <AuthButton code="user:add">
        <button style={{ margin: '0 8px' }}>新增用户</button>
      </AuthButton>

      <AuthButton code="user:edit">
        <button style={{ margin: '0 8px' }}>编辑用户</button>
      </AuthButton>

      <AuthButton code="user:del">
        <button>删除用户</button>
      </AuthButton>

      <div style={{ marginTop: '30px' }}>
        <button onClick={logout}>退出登录</button>
      </div>
    </div>
  );
}
```
6. 403 无权限页面 Forbidden.jsx
```jsx
export default function Forbidden() {
  return (
    <div style={{ padding: '30px' }}>
      <h3>403 暂无访问权限</h3>
    </div>
  );
}
```
7. 路由配置 router/index.jsx
```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { AuthRoute } from '../AuthRoute';
import Login from '../Login';
import UserPage from '../UserPage';
import Forbidden from '../Forbidden';

const router = createBrowserRouter([
  { path: '/login', element: <Login /> },
  { path: '/403', element: <Forbidden /> },
  {
    path: '/user',
    element: (
      <AuthRoute requirePerms="user:list">
        <UserPage />
      </AuthRoute>
    )
  }
]);

export default function AppRouter() {
  return <RouterProvider router={router} />;
}
```
8. 全局入口 main.jsx
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { AuthProvider } from './AuthContext';
import AppRouter from './router';

ReactDOM.createRoot(document.getElementById('root')).render(
  <AuthProvider>
    <AppRouter />
  </AuthProvider>
);
```
9. Axios 请求统一拦截
```jsx
import axios from 'axios';

// 请求拦截：统一携带 Token
axios.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截：401 403 自动登出跳转
axios.interceptors.response.use(
  res => res,
  err => {
    const status = err.response?.status;
    if (status === 401 || status === 403) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(err);
  }
);
```
## 四、面试标准回答

权限整体分为三层控制：

接口层：axios 拦截器统一注入 Token，捕获 401、403 状态码，清空登录态并强制跳转登录页；

页面层：通过封装路由守卫高阶组件，校验登录态和页面权限码，实现未登录拦截、无权限跳转 403；

按钮层：封装权限通用组件，根据当前用户权限码列表，控制按钮、操作区域的渲染与否；

全局使用 Context 管理登录态、角色、权限数组，刷新页面自动恢复登录信息，退出登录清空本地缓存与全局状态，满足中后台多角色、细粒度权限管控需求。