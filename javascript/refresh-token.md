```js
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
});

// 存储刷新 token 的 Promise 避免多次刷新
let isRefreshing = false;
let refreshSubscribers = [];

// 添加拦截器
api.interceptors.response.use(
  response => response,
  async error => {
    const originalRequest = error.config;

    // 检测是否是 token 过期
    if (error.response && error.response.status === 401) {
      if (!isRefreshing) {
        isRefreshing = true;

        try {
          // 刷新 token 的逻辑
          const refreshToken = localStorage.getItem('refreshToken');
          const { data } = await axios.post('https://api.example.com/auth/refresh', { token: refreshToken });

          // 更新 token
          localStorage.setItem('accessToken', data.accessToken);
          api.defaults.headers.common['Authorization'] = `Bearer ${data.accessToken}`;

          // 通知所有订阅者重新发起请求
          refreshSubscribers.forEach(callback => callback(data.accessToken));
          refreshSubscribers = [];
          isRefreshing = false;
        } catch (refreshError) {
          // 刷新 token 失败，重定向到登录页或处理逻辑
          isRefreshing = false;
          refreshSubscribers = [];
          return Promise.reject(refreshError);
        }
      }

      // 添加到刷新队列，等待 token 刷新完成
      return new Promise(resolve => {
        refreshSubscribers.push(token => {
          originalRequest.headers['Authorization'] = `Bearer ${token}`;
          resolve(api(originalRequest));
        });
      });
    }

    // 如果是其他错误，直接返回
    return Promise.reject(error);
  }
);

export default api;
```