<div align="center">

# 影视器材预约系统 SGFB-Rent

<img src="SGFBLogo.png" alt="SGFB Logo" width="160" />

**中德品牌学部 · 摄影器材在线预约与管理平台**

Vue 3 + Spring Boot + MySQL · Docker 一键部署

</div>

---

## 📖 项目简介

SGFB-Rent 是一套面向高校院系的**摄影/影视器材在线预约与管理系统**。学生可在前台填写个人信息并选择相机、镜头、稳定器等设备完成预约；管理员通过后台进行设备管理、订单审核、状态流转、时间段配置以及记录导出。

系统通过时间段冲突校验、设备状态与订单状态自动同步、定时逾期检测等机制，保证器材借还流程的准确与高效。

> 当前版本：**v2.0.0** · 作者：Kirbyuuu (Jiaqi Yu)

---

## ✨ 功能特性

### 🧑‍🎓 前台（学生端，免登录）

- **预约申请**：填写姓名 / 学号 / 手机号，分步选择相机、镜头、其他设备，选择预约与归还时间后提交
- **时间冲突校验**：提交前自动校验所选设备在目标时间段是否被占用，避免一物多借
- **公告弹窗**：进入预约页时展示管理员发布的 Markdown 公告，需确认「我已知悉」
- **预约甘特图**：以甘特图直观展示近期（默认 ±5 天）所有预约的时间分布与设备占用
- **响应式布局**：移动端自动切换为抽屉式导航，手机端可用

### 🛠 后台（管理员端，需登录）

- **预约单详情**：分页查看所有订单，按时间段 / 状态筛选，支持编辑、删除、状态流转
- **设备管理**：设备的增删改查，删除前校验是否被在用订单引用
- **预约时间管理**：按星期几配置可预约时间段（如周一~周五上下午、周末仅上午），前台据此生成可选时间点
- **Excel 导出**：按日期范围一键导出预约记录为 `.xlsx`（基于 Apache POI）
- **登录鉴权**：基于 token 的简单鉴权，前端路由守卫拦截未授权访问

### ⚙️ 后台自动化机制

- **设备状态联动**：订单状态变更时，自动把关联设备库存同步为对应状态（借出 / 逾期 / 归还）
- **定时逾期检测**：每 10 分钟扫描「已借出」订单，超过归还时间自动标记为「逾期未还」并同步设备状态
- **订单号自生成**：格式为 `yyyyMMdd + 4 位序号`（如 `202604030001`），按日递增

---

## 🧰 技术栈

| 层级 | 技术选型 |
| --- | --- |
| 前端框架 | Vue 3.4（Composition API + `<script setup>`） |
| 构建工具 | Vite 5 |
| UI 组件库 | Element Plus 2.5 + Element Plus Icons |
| 路由 | Vue Router 4（History 模式） |
| 样式 | Tailwind CSS 4 + 自定义 SCSS 风格 |
| 甘特图 | `@adesin-fr/vue-ganttastic`、`gantt-schedule-timeline-calendar` |
| Markdown 渲染 | marked |
| 后端框架 | Spring Boot 2.7.18（Java 11） |
| ORM | MyBatis-Plus 3.5.5（含分页插件） |
| 数据库 | MySQL 8.0 |
| 工具库 | Lombok、Apache POI 5.2.3（Excel 导出） |
| 部署 | Docker + Docker Compose、Nginx（前端静态托管 + 反向代理） |

---

## 📂 项目结构

```
SGFB_Rent/
├── index.html                  # 前端 HTML 入口
├── package.json                # 前端依赖与脚本
├── vite.config.js              # Vite 配置（含 /api 代理到 8080）
├── SGFBLogo.png                # 学部 Logo
├── RENT_SYS.sql                # 数据库初始化脚本（建表 + 示例数据）
├── docker-compose.yml          # 容器编排：backend + frontend
├── nginx.conf                  # 前端 Nginx 配置（静态托管 + 反向代理 + Gzip）
├── Dockerfile.frontend         # 前端多阶段构建（Node 构建 → Nginx 运行）
│
├── src/                        # 前端源码
│   ├── main.js                 # 应用入口
│   ├── App.vue                 # 布局（侧边栏 + 路由出口 + 移动端抽屉）
│   ├── router/index.js         # 路由表 + 登录守卫
│   ├── components/
│   │   └── TimeSlotPicker.vue  # 时间段选择器组件
│   └── views/
│       ├── Login.vue               # 管理员登录
│       ├── BorrowApplication.vue   # 预约申请（学生前台）
│       ├── BorrowGantt.vue         # 预约甘特图
│       ├── RentDetails.vue         # 预约单详情（管理）
│       ├── EquipmentManagement.vue # 设备管理
│       └── TimeSlotManagement.vue  # 预约时间管理
│
└── backend/                    # 后端源码
    ├── pom.xml
    ├── Dockerfile              # 后端多阶段构建（Maven 构建 → JRE 运行）
    └── src/main/
        ├── java/com/sgfb/rent/
        │   ├── RentApplication.java          # 启动类
        │   ├── config/MybatisPlusConfig.java # 分页插件配置
        │   ├── controller/                  # REST 控制器
        │   │   ├── LoginController.java      #   登录
        │   │   ├── DeviceController.java     #   设备
        │   │   ├── RentRecordController.java #   预约记录（含甘特图、Excel 导出）
        │   │   ├── TimeSlotController.java   #   时间段
        │   │   └── AnnounceController.java   #   公告
        │   ├── entity/                      # 实体（MyBatis-Plus 注解）
        │   ├── mapper/                      # Mapper 接口
        │   └── service/                     # 业务层
        │       ├── UserService.java
        │       ├── DeviceService.java        # 含时间冲突检测
        │       ├── RentRecordService.java    # 含订单状态联动、定时逾期检测
        │       ├── TimeSlotService.java      # 含时间点生成
        │       └── AnnounceService.java
        └── resources/
            ├── application.yml               # 本地配置
            └── application-docker.yml        # Docker 配置（环境变量注入）
```

---

## 🗄 数据模型

数据库 `RENT_SYS` 包含 5 张表：

| 表名 | 说明 | 关键字段 |
| --- | --- | --- |
| `DEVICE_LIST` | 摄影器材列表 | `id`、`NAME`、`BRAND`、`TYPE`、`STATUS` |
| `RENT_LIST` | 器材预约记录表（主表） | `id`(订单号)、`NAME`/`NUM`/`TEL`(借用人)、`CAMARA`/`LENS`/`OTHER`(外键→DEVICE_LIST)、`BRWTIME`/`RTNTIME`(预约/归还)、`STATUS`、`REMARK` |
| `USER_SYS` | 管理员用户表 | `USERNAME`、`PASSWORD` |
| `TIME_SLOT` | 预约时间段配置表 | `day_of_week`、`period_index`、`time_range_start/end`、`enabled` |
| `RENT_ANC` | 公告表 | `ANNOUNCE`（Markdown 文本） |

### 状态码约定

**设备状态 `DEVICE_LIST.STATUS`**

| 值 | 含义 |
| --- | --- |
| 0 | 正常库存 |
| 1 | 已预约借出 |
| 2 | 逾期未还 |
| 3 | 设备故障 |

**订单状态 `RENT_LIST.STATUS`**

| 值 | 含义（前端语义） | 联动设备状态 |
| --- | --- | --- |
| 0 | 已预约 reserved | → 1 已预约借出 |
| 1 | 已归还 returned | → 0 正常库存 |
| 2 | 逾期未还 overdue | → 2 逾期未还 |
| 3 | 已借出 borrowed | → 1 已预约借出 |
| 4 | 预约未取 unpicked | → 0 正常库存 |

---

## 🚀 快速开始

### 环境要求

- **Node.js** ≥ 18
- **JDK** 11（推荐 Eclipse Temurin）
- **Maven** 3.6+
- **MySQL** 8.0
- （可选）Docker & Docker Compose

### 1️⃣ 初始化数据库

```bash
# 登录 MySQL 后执行
mysql -u root -p < RENT_SYS.sql
```

脚本会创建 `RENT_SYS` 数据库下的全部表并插入示例数据（含若干测试器材、预约记录、默认管理员账号）。

### 2️⃣ 启动后端

```bash
cd backend
# 修改 src/main/resources/application.yml 中的数据库连接（url / username / password）
mvn spring-boot:run
# 后端默认运行在 http://localhost:8080
```

### 3️⃣ 启动前端

```bash
# 回到项目根目录
npm install
npm run dev
# 前端默认运行在 http://localhost:3000，/api 自动代理到 8080
```

浏览器访问 `http://localhost:3000` 即可。

> 生产预览：`npm run build` 后 `npm run start:prod` 可在 3000 端口预览构建产物。

---

## 🐳 Docker 部署

项目提供 `docker-compose.yml`，一键拉起前后端两个服务。

### 前提

- 宿主机已运行 MySQL 8.0，并已导入 `RENT_SYS.sql`
- Docker 引擎已安装

### 启动

```bash
# 1. 按需修改 docker-compose.yml 中的数据库连接环境变量
#    SPRING_DATASOURCE_URL / USERNAME / PASSWORD
# 2. 构建并启动
docker compose up -d --build
```

启动后：

- 前端：`http://localhost`（Nginx 80 端口）
- 后端：`http://localhost:8080`

### 架构说明

- **前端容器**：多阶段构建，`node:18-alpine` 执行 `vite build`，产物拷贝到 `nginx:alpine`。Nginx 负责 SPA 静态托管、`/api/` 反向代理到 `backend:8080`、Gzip 压缩与静态资源缓存。
- **后端容器**：多阶段构建，`maven:3.9-eclipse-temurin-11` 打包，`eclipse-temurin:11-jre` 运行。使用非 root 用户 `sgfb`，JVM 内存限制 `-Xmx512m -Xms256m`，激活 `docker` profile 通过环境变量注入数据库连接。
- 两容器通过 `sgfb-network` 桥接网络互通；后端经 `host.docker.internal` 访问宿主机 MySQL。

---

## 🔌 API 接口概览

所有接口前缀 `/api`，响应统一为 `{ success: boolean, data/message, ... }` 结构。

### 登录

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | `/api/login` | 管理员登录，返回 token 与用户信息 |

### 设备

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/devices` | 获取全部设备 |
| GET | `/api/devices/by-type/{type}` | 按类型查询设备 |
| GET | `/api/devices/available` | 按类型 + 时间段查询可用设备（含冲突过滤） |
| POST | `/api/devices` | 新增设备 |
| PUT | `/api/devices/{id}` | 更新设备状态 |
| PUT | `/api/devices/{id}/info` | 更新设备信息 |
| DELETE | `/api/devices/{id}` | 删除设备（在用时拒绝） |

### 预约记录

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/rent-records` | 分页查询订单（支持按日期、状态筛选） |
| GET | `/api/rent-records/gantt` | 甘特图数据（默认 ±5 天） |
| POST | `/api/rent-records` | 创建预约（含时间冲突校验） |
| PUT | `/api/rent-records/{id}` | 更新订单（含时间冲突校验） |
| PUT | `/api/rent-records/{id}/status` | 更新订单状态（联动设备状态） |
| DELETE | `/api/rent-records/{id}` | 删除订单（已预约状态同步恢复设备库存） |
| GET | `/api/rent-records/export` | 按日期范围导出 Excel |

### 时间段

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/time-slots` | 获取全部时间段配置 |
| GET | `/api/time-slots/enabled` | 仅获取启用的时间段 |
| GET | `/api/time-slots/day/{dayOfWeek}` | 按星期查询 |
| GET | `/api/time-slots/points/{dayOfWeek}` | 生成可选时间点（5 分钟粒度） |
| POST | `/api/time-slots` | 新增/更新时间段 |
| DELETE | `/api/time-slots/{id}` | 删除时间段 |

### 公告

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/api/announce` | 获取公告内容（Markdown） |
| POST | `/api/announce` | 更新公告 |

---

## 🔒 路由权限

| 路由 | 页面 | 是否需要登录 |
| --- | --- | --- |
| `/borrow-application` | 预约申请 | 否（学生前台） |
| `/borrow-gantt` | 预约甘特图 | 否 |
| `/login` | 登录 | 否 |
| `/rent-details` | 预约单详情 | ✅ |
| `/equipment-management` | 设备管理 | ✅ |
| `/time-slot-management` | 预约时间管理 | ✅ |

前端通过 `vue-router` 全局守卫 + `localStorage` 中的 `token` 判断登录态，未登录访问受保护路由将跳转至 `/login`。

---

## 🧩 核心业务逻辑

### 时间冲突检测

`DeviceService.isDeviceAvailable` 在创建/更新预约时被调用：拉取所有「活跃订单」（状态为 已预约 / 逾期 / 已借出），判断目标设备是否在目标时间段内与已有订单时间重叠。重叠判定采用区间相交公式：`start1 < end2 && end1 > start2`。

### 订单与设备状态联动

`RentRecordService` 在订单状态变更、设备字段变更时，按规则同步关联设备的库存状态（见上文状态码表），保证订单视图与设备库存视图一致。

### 定时逾期检测

`@Scheduled(fixedRate = 600000)`（每 10 分钟）扫描所有「已借出」(status=3) 订单，若归还时间早于当前时间，则自动流转为「逾期未还」(status=2) 并联动设备状态。

---

## 📝 默认账号

`RENT_SYS.sql` 中预置的管理员账号（仅用于本地测试，**生产环境务必修改密码**）：

| 用户名 | 密码 |
| --- | --- |
| admin | UNAV@1LABLE |

---

## ⚠️ 安全注意事项

- `application.yml` 与 `docker-compose.yml` 中包含数据库明文连接信息，**生产部署前请改为环境变量或密钥管理方案**，切勿直接提交真实凭据。
- 当前登录为明文密码比对，未使用加密哈希与 Session/JWT，仅适用于受控校园内网环境。如需公网部署，建议补充密码加密、登录限流、HTTPS 等加固措施。
- 后端控制器使用 `@CrossOrigin(origins = "*")`，生产环境应收敛为可信来源。

---

## 📜 脚本说明

| 命令 | 说明 |
| --- | --- |
| `npm run dev` | 启动前端开发服务器（端口 3000，热更新） |
| `npm run build` | 构建前端生产产物到 `dist/` |
| `npm run preview` | 本地预览构建产物 |
| `npm run start:prod` | 以 3000 端口对外预览生产构建 |
| `mvn spring-boot:run`（backend 目录） | 启动后端 |
| `mvn clean package -DskipTests`（backend 目录） | 打包可执行 jar |
| `docker compose up -d --build` | 一键容器化部署 |

---

<div align="center">

**Developed By Kirbyuuu** · 中德品牌学部

</div>
