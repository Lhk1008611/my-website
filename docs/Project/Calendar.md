---
sidebar_position: 1
---

# 行事历系统开发文档

## 一、需求调研

### 1.1 项目背景

在日常工作和生活中，用户需要高效管理个人任务、会议、待办事项等时间敏感活动。传统的纸质或简单数字工具无法灵活支持重复事件、多日历分类、可视化拖拽调整等需求。因此，本项目旨在开发一个基于 Web 的现代行事历系统，提供直观的日历视图、灵活的事件管理和良好的用户体验。

### 1.2 用户角色

- **普通用户**：注册、登录、管理自己的行事历和事件。

### 1.3 功能需求

- **用户认证**：注册、登录、登出、个人信息修改、账号注销。
- **行事历管理**：创建多个行事历（工作、个人、家庭等）、设置默认日历、修改日历名称/颜色/可见性、删除日历。
- **事件管理**：
  - 创建/编辑/删除事件（标题、描述、起止时间、全天事件、颜色、地点、优先级、状态）。
  - 支持重复事件（每日、每周、每月、每年），可设置重复结束日期。
  - 支持删除重复事件的单个实例或整个系列。
  - 支持拖拽移动事件、调整事件时长。
- **日历视图**：月视图、周视图、日视图、年视图（传统表格样式）。
- **懒加载**：根据当前视图时间范围动态请求事件，减轻服务器压力。
- **界面风格**：简洁、美观、响应式，使用 Bootstrap 5 组件。

### 1.4 非功能需求

- 性能：事件加载响应时间 < 300ms（常规数据量）。
- 安全：用户数据隔离，基于会话的身份验证，防止 CSRF 攻击。
- 可维护性：前后端代码结构清晰，遵循 MVC 模式。
- 跨平台：支持主流浏览器（Chrome, Firefox, Edge）。

## 二、系统设计

### 2.1 总体架构

采用前后端分离但耦合于 Laravel 的混合模式：

- 后端：Laravel 提供 RESTful API 和前端入口视图（`app.blade.php`）。
- 前端：Vue 3 构建 SPA，通过 Vite 编译，嵌入 Laravel 视图。
- 认证：基于 Laravel Sanctum SPA 认证，使用 `auth:sanctum` 中间件保护路由。

### 2.2 数据库设计

#### 表结构

- **users**（Laravel 自带）

- **calendars**

  | 字段                               | 类型         | 说明                            |
  | :--------------------------------- | :----------- | :------------------------------ |
  | id                                 | bigint       | 主键                            |
  | user_id                            | bigint       | 所属用户                        |
  | name                               | varchar(100) | 日历名称                        |
  | description                        | text         | 描述                            |
  | color                              | varchar(7)   | 主题色                          |
  | is_default                         | boolean      | 是否默认日历                    |
  | visibility                         | tinyint      | 可见性（1仅自己，2共享，3公开） |
  | created_at, updated_at, deleted_at | timestamp    | 时间戳                          |

- **calendar_events**

  | 字段                               | 类型         | 说明                                                         |
  | :--------------------------------- | :----------- | :----------------------------------------------------------- |
  | id                                 | bigint       | 主键                                                         |
  | calendar_id                        | bigint       | 所属日历                                                     |
  | title                              | varchar(255) | 事件标题                                                     |
  | description                        | text         | 描述                                                         |
  | start_time                         | datetime     | 开始时间（UTC）                                              |
  | end_time                           | datetime     | 结束时间（UTC）                                              |
  | all_day                            | boolean      | 是否全天                                                     |
  | status                             | tinyint      | 状态（1待办，2进行中，3已完成，4已取消）                     |
  | priority                           | tinyint      | 优先级（1低，2中，3高）                                      |
  | location                           | varchar(255) | 地点                                                         |
  | color                              | varchar(7)   | 自定义颜色                                                   |
  | rrule                              | json         | 重复规则（如 `{"freq":"weekly","interval":1,"byweekday":["MO"]}`） |
  | exdates                            | json         | 排除的日期列表（存储 ISO 日期字符串）                        |
  | rrule_until                        | datetime     | 冗余字段，用于快速查询（从 rrule.until 提取）                |
  | created_at, updated_at, deleted_at | timestamp    | 时间戳                                                       |

#### 索引

- `calendars.user_id`
- `calendar_events.calendar_id`
- `calendar_events.start_time`, `end_time`
- 复合索引：`(calendar_id, start_time, end_time)`
- `calendar_events.rrule_until`

### 2.3 前端状态管理

使用 Pinia 管理用户认证状态（`user`、`isAuthenticated`）和全局加载状态。

### 2.4 组件划分

- **AuthCard**：登录/注册卡片容器。
- **Login** / **Register**：表单组件。
- **Layout**：后台主布局（顶部导航栏 + 路由视图）。
- **Calendar**：FullCalendar 日历视图，集成新增/编辑事件模态框、删除确认。
- **Manage**：行事历管理页面（左右布局，包含 CalendarList 和 CalendarEventList）。
- **CalendarList**：日历列表表格（搜索、分页、批量删除）。
- **CalendarEventList**：事件列表表格（搜索、分页、批量删除）。
- **Profile**：个人信息页面（只读展示 + 修改资料模态框 + 注销账号）。

### 2.5 核心业务逻辑

- **重复事件处理**：
  - 前端使用 FullCalendar 的 `rrule` 插件，事件对象包含 `rrule` 属性（如 `{ freq: 'weekly', interval: 1, byweekday: ['MO'], until: '2026-12-31' }`）。
  - 后端存储 `rrule` 为 JSON，并维护 `rrule_until` 字段用于快速查询未来重复事件。
  - 删除单个实例时，将排除日期加入 `exdates` 数组，后端在查询时（`getRangeEvents`）动态过滤。
  - 拖拽重复事件实例时，前端执行“排除原实例 + 创建新普通事件”的逻辑。
- **懒加载**：FullCalendar 的 `events` 函数根据当前视图的 `start` 和 `end` 参数请求 `/api/calendar_event/range`，后端返回该时间范围内的事件实例（包括重复展开的实例）。

## 三、技术选型

### 3.1 后端

- **PHP 8.2+**：语言基础。
- **Laravel 11**：框架，提供路由、ORM、中间件、会话管理。
- **MySQL**：关系数据库。
- **Redis**：可选，用于缓存和会话。
- **Composer**：依赖管理。

### 3.2 前端

- **Vue 3**（Composition API）：响应式 UI 框架。
- **Vite**：构建工具，与 Laravel 集成（`laravel-vite-plugin`）。
- **Bootstrap 5** + **Bootstrap Icons**：样式与图标。
- **FullCalendar v6**：日历核心组件，包含插件：
  - `@fullcalendar/vue3`
  - `@fullcalendar/daygrid`
  - `@fullcalendar/timegrid`
  - `@fullcalendar/interaction`
  - `@fullcalendar/rrule`（前端重复规则解析）
- **Pinia**：状态管理。
- **Axios**：HTTP 客户端（配置 `withCredentials: true`）。
- **Vue Router**：前端路由（History 模式）。

### 3.3 开发工具

- **PhpStorm**
- **Git** 版本控制
- **npm** 包管理

## 四、系统开发

### 4.1 环境搭建

1. 安装 PHP、Composer、Node.js、MySQL。
2. 创建 Laravel 项目：`composer create-project laravel/laravel calendar-system`
3. 安装前端依赖：`npm install`，并配置 Vite。
4. 配置 `.env` 数据库连接。
5. 生成应用密钥：`php artisan key:generate`。

### 4.2 后端开发

#### 4.2.1 认证模块

- 使用 Laravel 内置 `Auth` 控制器（或自定义 `AuthController`）。
- 登录路由放在 `web` 中间件组中，使用 `Auth::attempt()` 并调用 `session()->regenerate()`。
- 登出：`Auth::logout()` + 失效会话。
- 个人信息修改：验证密码后更新用户表。

#### 4.2.2 日历模块

- 创建 `CalendarController`，实现 CRUD：
  - `index`：分页、搜索（按名称）。
  - `store`：新增日历，若 `is_default` 为 true 则清除其他默认标志。
  - `destroy`：支持批量删除（检查是否为默认日历，禁止删除默认日历）。
- 模型关联：`Calendar belongsTo User`，`hasMany CalendarEvent`。

#### 4.2.3 事件模块

- 创建 `CalendarEventController`：
  - `getRangeEvents`：根据 `start`、`end` 返回默认日历下时间范围内的事件（包括重复展开）。使用 `where('start_time', '<', $end)->where('end_time', '>', $start)` 优化重叠查询。
  - `store` / `update`：验证输入，处理 `rrule` 字段（JSON），若 `rrule` 有 `until` 则同步更新 `rrule_until`。
  - `destroy`：支持批量删除。
  - `excludeOccurrence`：接收 `date` 参数，将其添加到 `exdates` 数组。
- 重复事件展开：由于前端使用 FullCalendar 的 `rrule` 插件，后端无需展开，只需原样返回 `rrule` 字段。
  - 排除实例：由于前端 FullCalendar 的 `rrule` 插件可以识别 `exdate`，因此后端不对重复事件展开，仅存储规则；排除实例通过前端在渲染时过滤，即根据 `exdates` 数组，不显示对应实例
  - 排除实例也可以由后端完成：对于重复事件，可以使用 PHP RRULE 库（如 `rlanvin/php-rrule`）生成实例，并过滤掉 `exdates`

### 4.3 前端开发

#### 4.3.1 页面结构

- **App.vue**：根组件，包含路由视图。
- **router/index.js**：定义路由（`/` 主页，`/dashboard` 后台布局，子路由 `calendar`、`profile`、`manage`）。
- **布局组件**：`Layout.vue` 包含顶部导航栏和 `router-view`。

#### 4.3.2 FullCalendar 集成

- 安装依赖：`npm install @fullcalendar/vue3 @fullcalendar/core @fullcalendar/daygrid @fullcalendar/timegrid @fullcalendar/interaction @fullcalendar/rrule`
- 在 `Calendar.vue` 中配置：
  - `events` 函数异步请求 `/api/calendar_event/range`，返回事件数组。
  - 对于重复事件，返回对象需包含 `rrule` 属性（格式符合 FullCalendar 要求）和 `duration`（非全天事件）。
  - 实现 `eventDrop`、`eventResize` 回调，向后端同步修改。
  - 实现 `select` 和 `dateClick` 打开新增事件模态框。
  - 实现 `eventClick` 打开编辑模态框。
- 重复事件实例的删除：通过 `exdate` 机制，前端调用 `/api/calendar_event/{id}/exclude` 添加排除日期，然后重新获取事件渲染到行事历上

#### 4.3.3 状态管理（Pinia）

- `auth` store：`user`、`isAuthenticated`、`fetchUser`、`logout`。
- 在 `App.vue` 的 `onMounted` 中调用 `fetchUser()` 恢复登录状态。

#### 4.3.4 样式与主题

- 使用 Bootstrap 5 的全局样式，自定义 CSS 覆盖 FullCalendar 工具栏、时间轴等。
- 响应式设计：在移动设备上调整导航栏和表格布局。

### 4.4 测试与部署

- **单元测试**：PHPUnit 测试模型关系、验证规则。
- **端到端测试**：手动测试登录、增删改查、重复事件、拖拽等。
- **部署**：
  - 配置生产环境 `.env`（`APP_ENV=production`，`APP_DEBUG=false`）。
  - 运行 `npm run build` 生成静态资源。
  - 使用 Nginx 或 Apache 指向 Laravel 的 `public` 目录。
  - 设置计划任务（如需要）和队列（可选）。

## 五、开发中遇到的问题与解决方案

| 问题                                 | 解决方案                                                                       |
| :----------------------------------- |:---------------------------------------------------------------------------|
| FullCalendar 时区导致事件偏移 8 小时 | 后端存储 UTC，前端 FullCalendar 默认使用本地时区，新增事件时将本地时间转为 UTC 字符串（`toISOString()`）发送。 |
| 模态框关闭后残留遮罩层               | 监听 `hidden.bs.modal` 事件，调用 `modal.dispose()` 并手动移除 `.modal-backdrop`。      |
| 删除重复事件实例后，页面仍显示       | 使用 `exdates` 数组记录排除日期，前端在 `events` 函数中过滤掉被排除的实例（根据 `exdates` 列表）。          |
| 拖拽重复事件实例时其他实例闪烁       | 改为“排除原实例 + 创建新普通事件”策略，避免拖拽重复事件实例本身。                                        |
| 表格数据变化导致页面高度抖动         | 固定表格容器高度，设置 `overflow-y: auto`，分页组件固定在底部。                                  |
| Sanctum 认证  | 路由使用 `auth:sanctum` 中间件，前端 Axios 保持 `withCredentials: true`。               |

## 六、未来扩展建议

- 支持日历共享（与他人协作）。
- 增加事件提醒（邮件、浏览器通知）。
- 导入/导出 iCalendar（.ics）文件。
- 移动端原生应用封装（如通过 Capacitor）。
- 增加数据统计报表（如每月事件数量分布）。

## 七、附录

### 7.1 环境变量示例（.env）

```ini
APP_NAME="行事历系统"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=calendar
DB_USERNAME=root
DB_PASSWORD=

SESSION_DRIVER=file
SESSION_LIFETIME=120
```



### 7.2 前端 Axios 配置（axios.js）

```javascript
import axios from 'axios';

export const api = axios.create({
    baseURL: '/api',
    withCredentials: true,
    headers: { 'X-Requested-With': 'XMLHttpRequest' }
});
```



### 7.3 关键 npm 脚本

```json
"scripts": {
    "dev": "vite",
    "build": "vite build"
}
```