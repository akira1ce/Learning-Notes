重定向需要配置在 routes 下。

```JavaScript
    {
      name: '监控管理',
      path: '/monitor',
      menu: { flatMenu: true },
      component: '@/layouts/monitorLayout',
      routes: [
        /** redirect */
        {
          path: '/monitor',
          redirect: '/monitor/monitorView',
        },
        {
          path: '/monitor/EventMgt',
          redirect: '/monitor/EventMgt/realTimeEvent',
        },
        {
          name: '监控查看',
          path: '/monitor/monitorView',
          component: '@/pages/Monitor/MonitorView',
          icon: 'DesktopOutlined',
        },
        {
          name: '事件管理',
          path: '/monitor/EventMgt',
          icon: 'FileTextOutlined',
          routes: [
            {
              name: '实时事件',
              path: '/monitor/EventMgt/realTimeEvent',
              component: '@/pages/Monitor/EventMgt/RealTimeEvent',
            },
            { name: '历史事件', path: '/monitor/EventMgt/historyEvent', component: '@/pages/Monitor/EventMgt/HistoryEvent' },
            { name: '事件管理', path: '/monitor/EventMgt/eventMgt', component: '' },
          ],
        },
        {
          name: '工具管理',
          path: '/monitor/toolMgt',
          component: '@/pages/Monitor/ToolMgt',
          icon: 'ToolOutlined',
        },
        {
          name: '监控覆盖率',
          path: '/monitor/monitorOver',
          component: '@/pages/Monitor/MonitorOver',
          icon: 'EditOutlined',
        },
        {
          name: 'chatOps',
          path: '/monitor/chatOps',
          component: '@/pages/ChatOps',
          menuRender: false,
          hideInMenu: true,
        },
      ],
    },
```

