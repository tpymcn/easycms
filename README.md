# ThinkPHP 8 内容发布系统 (CMS)

基于 ThinkPHP 8 框架开发的 MVC 内容发布系统，使用 SQLite 数据库，零外部数据库依赖，开箱即用。前台模板集成自杂志网站风格，支持响应式布局与移动端导航。

## 功能模块

### 系统设置

- **基本设置**：网站名称、网站地址、网站关键词、网站描述（textarea）、网站 Logo、备案信息、版权信息、联系方式、在线客服 URL
- **高级设置**：网站状态（开关按钮，1 开启 / 0 关闭）、关闭原因
- 网站关闭后前台所有页面显示维护提示页（HTTP 503），后台仍可正常访问

### 文章分类管理

- 分类的增删改查，支持排序、启用/禁用
- 分类别名（slug）用于前台 URL：`/category/{slug}`
- 删除分类前检查是否有关联文章

### 文章发布

- 文章的发布、编辑、删除，支持排序、启用/禁用
- **富文本编辑器**：自建零依赖编辑器（contenteditable + execCommand），支持加粗/斜体/列表/引用/对齐/链接/图片上传等
- **缩略图上传**：支持 jpg/png/gif/webp，限 5MB，自动存储到 `public/uploads/YYYYMMDD/`
- **文章密码加密**：设置密码后前台详情页仅显示标题，需输入密码验证后才显示内容（password_hash 存储，Session 标记验证状态）
- **URL 别名（slug）**：文章详情页通过 slug 访问（`/detail/{slug}`），无 slug 时自动回退到 ID
- **浏览量统计**：详情页访问自动累加浏览量

### 友情链接管理

- 链接的增删改查，支持排序、启用/禁用
- 仅在前台首页底部显示已启用的友情链接

### 前台展示

- 首页文章列表分页（每页 12 篇）
- 分类筛选（通过 slug 访问 `/category/{slug}`）
- 全站搜索（搜索参数使用 `q`，不使用 `s`）
- 文章详情页：面包屑导航、相关推荐、返回顶部
- 移动端右侧滑出导航面板、顶部下拉搜索面板
- Footer 粘性底部（内容不足一屏时 footer 贴底）
- 站点关闭拦截页（closed.html）

![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405246-7d3e00c0249e347.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405247-ac8189e87bad674.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405243-7fb5c29b7a46470.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405248-ab687efce59d199.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405250-1f11e727e643b37.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405248-ab687efce59d199.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405252-9215559ca9cb8ef.png)
![演示图片](https://tpym.cn/wp-content/uploads/2026/09/1788405253-2fbe42edb5446c7.png)
## 技术栈

| 项目 | 说明 |
|------|------|
| 框架 | ThinkPHP 8.x |
| 数据库 | SQLite（文件 `sqlite.db`，表前缀 `cms_`） |
| 前台模板 | 集成自定义杂志风格模板（含 CSS/JS/图片资源） |
| 后台模板 | 自建管理后台（SVG 图标，无外部图标库依赖） |
| 后台路径 | `/manage` |
| PHP 版本 | >= 8.0 |

## 数据表结构

| 表名 | 说明 |
|------|------|
| `cms_admin` | 管理员账号（username, password, login_time, login_ip） |
| `cms_setting` | 系统设置（group_name, key_name, value, title, sort） |
| `cms_category` | 文章分类（parent_id, name, slug, description, sort, status） |
| `cms_article` | 文章（category_id, title, slug, summary, content, thumbnail, source, author, views, password, sort, status） |
| `cms_link` | 友情链接（name, url, sort, status） |

## 安装步骤

### 1. 安装依赖

在项目根目录执行：

```bash
composer install
```

### 2. 初始化数据库

执行安装命令，自动创建数据表并插入默认数据：

```bash
php think cms:install
```

该命令会：
- 创建 SQLite 数据库文件（项目根目录 `sqlite.db`）
- 创建 5 张数据表（cms_admin / cms_setting / cms_category / cms_article / cms_link）
- 插入默认管理员账号（admin / admin123）
- 插入默认系统设置（11 项，含联系方式和在线客服 URL）
- 插入默认分类和示例文章
- 插入默认友情链接
- 为已有数据库补充新增字段（ALTER TABLE，已存在则静默跳过）

> 重复执行 `php think cms:install` 是安全的，已有数据不会被覆盖。

### 3. 配置 Web 服务器

**Nginx 配置参考：**

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/wwwroot/public;
    index index.php index.html;

    # 伪静态：所有请求转发到 index.php
    location / {
        rewrite ^(.*)$ /index.php?s=/$1 last;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~* \.(js|css|jpg|png|gif|webp|woff|ico)$ {
        expires 30d;
    }
}
```

> Nginx 伪静态必须使用 `rewrite ^(.*)$ /index.php?s=/$1 last;`，否则路由无法正确解析。

**Apache 配置：**

项目 `public/` 目录已包含 `.htaccess` 文件，确保 `mod_rewrite` 已启用即可。

### 4. SQLite 文件权限

确保 `sqlite.db` 文件对 Web 用户可读写：

```bash
chmod 664 sqlite.db
chown www:www sqlite.db
```

### 5. 访问系统

- **前台首页**：`http://your-domain/`
- **后台管理**：`http://your-domain/manage`
- **默认管理员账号**：`admin`
- **默认密码**：`admin123`

> 请首次登录后及时修改密码！后台右上角下拉菜单可修改密码。

## 目录结构

```
wwwroot/
├── app/                            # 应用目录
│   ├── command/
│   │   ├── Install.php             # 数据库安装命令（建表 + 默认数据）
│   │   └── ResetPassword.php       # 重置管理员密码命令
│   ├── controller/
│   │   ├── Index.php               # 前台控制器（首页/详情/分类/密码验证/站点关闭拦截）
│   │   └── manage/                 # 后台控制器
│   │       ├── Base.php            # 后台基类（认证检查）
│   │       ├── Index.php           # 仪表盘
│   │       ├── Login.php           # 登录/退出/修改密码
│   │       ├── Setting.php         # 系统设置
│   │       ├── Category.php        # 分类管理
│   │       ├── Article.php         # 文章管理 + 图片上传
│   │       └── Link.php            # 友情链接管理
│   ├── model/                      # 数据模型
│   │   ├── Admin.php
│   │   ├── Setting.php             # 含 getAllSettings() / getSetting()
│   │   ├── Category.php            # 含 getList() / getActiveList()
│   │   ├── Article.php             # 含 getList() 分页查询
│   │   └── Link.php                # 含 getActiveList()
│   ├── middleware/
│   │   └── AdminAuth.php            # 后台认证中间件
│   ├── BaseController.php
│   ├── common.php                  # 公共函数（含模板过滤器 defval）
│   └── ...
├── config/                         # 配置目录
│   ├── database.php                # 数据库配置（SQLite，可切换 MySQL）
│   └── ...
├── route/
│   └── app.php                     # 路由定义（前台 + 后台，均带 completeMatch）
├── view/                           # 视图模板
│   ├── index/                      # 前台视图
│   │   ├── index.html              # 首页（文章列表 + 移动端导航/搜索 + 友链展示）
│   │   ├── detail.html             # 文章详情（密码验证 + 面包屑 + 相关推荐）
│   │   └── closed.html             # 站点关闭提示页
│   └── manage/                     # 后台视图
│       ├── layout.html             # 后台布局（侧边栏 SVG 图标 + 右上角下拉菜单 + 修改密码弹窗）
│       ├── login/
│       │   └── login.html          # 登录页
│       ├── index/
│       │   └── index.html          # 仪表盘（统计卡片 + 最近文章）
│       ├── article/
│       │   ├── article_list.html   # 文章列表（加密标识）
│       │   └── edit.html            # 文章编辑（富文本编辑器 + 密码输入 + 缩略图上传）
│       ├── category/
│       │   └── index.html          # 分类管理（列表 + 弹窗表单）
│       ├── link/
│       │   └── index.html           # 友情链接管理（列表 + 弹窗表单）
│       └── setting/
│           └── index.html           # 系统设置（textarea/开关/文本框条件渲染）
├── public/                         # Web 根目录
│   ├── index.php                   # 应用入口
│   ├── .htaccess                   # Apache 重写规则
│   ├── uploads/                    # 上传文件目录（按日期分目录）
│   └── static/                     # 静态资源
│       ├── css/                     # 前台样式
│       ├── js/                      # 前台脚本
│       ├── images/                 # 图片资源
│       ├── picture/                # 内容图片
│       └── fonts/                  # 字体文件
├── runtime/                        # 运行时目录（自动生成）
├── vendor/                         # Composer 依赖
├── composer.json
├── .env                            # 环境配置
└── think                           # 命令行入口
```

## 常用命令

```bash
# 安装依赖
composer install

# 初始化数据库（建表 + 默认数据，可重复执行）
php think cms:install

# 重置管理员密码（交互模式，会提示输入新密码）
php think cms:reset-password

# 重置指定管理员密码
php think cms:reset-password admin

# 直接设置新密码（非交互模式，适合脚本调用）
php think cms:reset-password admin --password=newpass123

# 清理缓存（修改路由或模板后建议执行）
rm -rf runtime/temp/* runtime/route*
```

## 常见问题

### Q: 安装后访问 404？

确保 Web 服务器配置正确：
1. `public/` 为文档根目录
2. Nginx 伪静态规则为 `rewrite ^(.*)$ /index.php?s=/$1 last;`
3. 清理缓存：`rm -rf runtime/temp/* runtime/route*`

### Q: 忘记管理员密码怎么办？

通过命令行重置密码，无需登录后台：

```bash
# 交互模式（会提示输入新密码，输入时不回显）
php think cms:reset-password

# 指定用户名重置
php think cms:reset-password admin

# 直接指定新密码（非交互模式）
php think cms:reset-password admin --password=newpass123
```

重置后即可使用新密码登录 `http://your-domain/manage`。

> 密码要求至少 6 位。交互模式下需要输入两次以确认。

### Q: 数据库在哪？

SQLite 数据库文件位于项目根目录 `sqlite.db`，首次运行 `php think cms:install` 后自动创建。确保 Web 用户对该文件有读写权限（`chmod 664`）。

### Q: 如何切换为 MySQL？

修改 `.env` 文件中的 `DB_DRIVER=mysql`，并在 `config/database.php` 的 mysql 配置中填入数据库信息，然后重新运行 `php think cms:install`。

### Q: 搜索功能不生效？

前台搜索参数使用 `q`（如 `/?q=关键词`），不能使用 `s`，因为 `s` 与 ThinkPHP 的 PATH_INFO 参数冲突。

### Q: 文章上传的图片在哪？

上传的图片存储在 `public/uploads/YYYYMMDD/` 目录下（按日期分目录），通过 `/uploads/YYYYMMDD/文件名` 访问。

### Q: 如何更新到新版本？

1. 覆盖代码文件
2. 执行 `composer install` 更新依赖
3. 执行 `php think cms:install`（会自动补充新增字段和设置项，不影响已有数据）
4. 清理缓存：`rm -rf runtime/temp/* runtime/route*`

### Q: 后台侧边栏图标显示异常？

后台使用内联 SVG 图标，无外部图标库依赖，确保浏览器支持 SVG 渲染即可（所有现代浏览器均支持）。

## 开发说明

### 模板过滤器

在 `app/common.php` 中定义了自定义模板过滤器：
- `defval`：替代 PHP 保留字 `default`，用于模板中设置默认值，如 `{$item.value|defval=''}`

### 路由注意事项

- 后台路由均使用 `->completeMatch(true)` 防止前缀匹配冲突
- 前台分类路由为 `category/:slug`，支持 slug 和 ID 两种访问方式
- 文章详情路由为 `detail/:slug`，同样支持 slug 和 ID

### PHP 8.0 兼容性

- `default` 是 PHP 8 保留字，不可做函数名，项目中使用 `defval` 替代
- `uniqid()` 第一个参数必须为 string 类型
- `Model::update()` 在 ThinkPHP 8 中需至少 1 个参数，自增浏览量使用 `Model::where()->inc()->update()`

## License

本系统基于 ThinkPHP 8 框架开发，可供个人学习和项目使用。
