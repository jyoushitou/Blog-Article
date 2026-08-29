---
slug: Blog_meta
title: 博客框架
description: 20260828
date: 2026-08-28
lastmod: 2026-08-28
tags: [test,框架]
summary: 部署了博客
---


# 收到——保留"微服务骨架",但把它压缩到最小可交付

你希望保留微服务形态,完全可以。我们把范围缩到**2 个服务 + 1 个网关**,仍然是一个真正的微服务架构,但不会让你陷入 10 个服务的泥潭。

---

## 一、最终架构(极简微服务)

```
浏览器 / curl
      │
      │ HTTP (JSON)
      ▼
┌─────────────────────────────────────┐
│  C++ 网关 (端口 8080)                │
│  · HTTP 服务器                        │
│  · 路由: /api/* → 转发到 Rust 服务    │
│  · /health → 检查后端存活             │
└──────────────┬──────────────────────┘
               │ HTTP (内网调用)
               ▼
┌─────────────────────────────────────┐
│  Rust 博客服务 (端口 8081)           │
│  · /api/articles → 扫描 content/*.md │
│  · /api/articles/:slug → 返回详情    │
│  · /health → 返回 ok                 │
│  · content/ 目录存放 Markdown        │
└─────────────────────────────────────┘
```

### 这是微服务,因为:

- ✅ 服务独立部署(Rust 服务可以单独跑,网关可以单独跑)
- ✅ 服务间通过 HTTP API 通信(不是共享内存/代码)
- ✅ 统一入口(所有请求走网关)
- ✅ 健康检查(网关能感知后端存活)
- ✅ 每个服务有自己的生命周期(重启/替换不影响对方)

### 这不是微服务,因为:

- ❌ 没有注册中心 → 用配置文件写死地址(最简服务发现)
- ❌ 没有配置中心 → 环境变量 + 配置文件
- ❌ 没有链路追踪 → 网关加个日志就是"追踪"
- ❌ 没有数据库 → Markdown 文件就是存储

**这些"没有"都是故意砍掉的,让骨架足够小。** 等骨架跑通,再逐个加(Consul→追踪→DB→gRPC)。

---

## 二、Rust 博客服务(复用之前的代码 + 加 /health)

```rust server/src/main.rs
use axum::{extract::Path, http::StatusCode, routing::get, Json, Router};
use serde::Serialize;
use std::fs;
use std::path::PathBuf;

#[derive(Serialize, Clone)]
struct ArticleMeta {
    slug: String,
    title: String,
    date: String,
    tags: Vec<String>,
}

#[derive(Serialize)]
struct ArticleDetail {
    meta: ArticleMeta,
    content: String,
}

#[derive(Serialize)]
struct Health {
    status: String,
    service: String,
}

fn parse_frontmatter(raw: &str) -> (Option<String>, Option<String>, Vec<String>) {
    let mut title = None;
    let mut date = None;
    let mut tags = Vec::new();

    if let Some(rest) = raw.strip_prefix("---") {
        if let Some(end_pos) = rest.find("---") {
            let fm = &rest[..end_pos];
            for line in fm.lines() {
                if let Some(value) = line.strip_prefix("title:") {
                    title = Some(value.trim().trim_matches('"').to_string());
                }
                if let Some(value) = line.strip_prefix("date:") {
                    date = Some(value.trim().trim_matches('"').to_string());
                }
                if let Some(value) = line.strip_prefix("tags:") {
                    tags = value
                        .trim()
                        .trim_start_matches('[')
                        .trim_end_matches(']')
                        .split(',')
                        .map(|s| s.trim().trim_matches('"').to_string())
                        .filter(|s| !s.is_empty())
                        .collect();
                }
            }
        }
    }
    (title, date, tags)
}

fn scan_articles() -> Vec<ArticleMeta> {
    let content_dir = PathBuf::from("../content");
    let mut articles = Vec::new();

    if let Ok(entries) = fs::read_dir(&content_dir) {
        for entry in entries.flatten() {
            let path = entry.path();
            if path.extension().and_then(|e| e.to_str()) != Some("md") {
                continue;
            }
            let slug = path.file_stem().and_then(|s| s.to_str()).unwrap_or("").to_string();
            if let Ok(raw) = fs::read_to_string(&path) {
                let (title, date, tags) = parse_frontmatter(&raw);
                articles.push(ArticleMeta {
                    slug,
                    title: title.unwrap_or_else(|| "Untitled".to_string()),
                    date: date.unwrap_or_else(|| "1970-01-01".to_string()),
                    tags,
                });
            }
        }
    }
    articles.sort_by(|a, b| b.date.cmp(&a.date));
    articles
}

async fn list_articles() -> Json<Vec<ArticleMeta>> {
    Json(scan_articles())
}

async fn get_article(Path(slug): Path<String>) -> Result<Json<ArticleDetail>, StatusCode> {
    let path = PathBuf::from(format!("../content/{}.md", slug));
    let raw = fs::read_to_string(&path).map_err(|_| StatusCode::NOT_FOUND)?;
    let (title, date, tags) = parse_frontmatter(&raw);
    let content = raw
        .strip_prefix("---")
        .and_then(|rest| rest.find("---").map(|pos| &rest[pos + 3..]))
        .unwrap_or(&raw)
        .to_string();
    Ok(Json(ArticleDetail {
        meta: ArticleMeta {
            slug,
            title: title.unwrap_or_else(|| "Untitled".to_string()),
            date: date.unwrap_or_else(|| "1970-01-01".to_string()),
            tags,
        },
        content,
    }))
}

async fn health() -> Json<Health> {
    Json(Health {
        status: "ok".into(),
        service: "blog-server".into(),
    })
}

#[tokio::main]
async fn main() {
    let app = Router::new()
        .route("/api/articles", get(list_articles))
        .route("/api/articles/:slug", get(get_article))
        .route("/health", get(health));

    let listener = tokio::net::TcpListener::bind("127.0.0.1:8081").await.unwrap();
    println!("blog-server listening on {}", listener.local_addr().unwrap());
    axum::serve(listener, app).await.unwrap();
}
```

```toml server/Cargo.toml
[package]
name = "blog-server"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

---

## 三、C++ 网关(核心微服务组件)

用 C++ 写一个 HTTP 反向代理:接收浏览器请求,转发给 Rust 服务。**不需要引入 boost/asio,用 libcurl 做客户端,用 C++ 标准库写一个极简 HTTP 服务器**——或者更简单:使用 `cpp-httplib`(单头文件库,非常轻)。

```cpp gateway/CMakeLists.txt
cmake_minimum_required(VERSION 3.14)
project(gateway)

set(CMAKE_CXX_STANDARD 17)

# 下载 cpp-httplib 单头文件
include(FetchContent)
FetchContent_Declare(
    httplib
    URL https://github.com/yhirose/cpp-httplib/archive/v0.18.0.tar.gz
)
FetchContent_MakeAvailable(httplib)

add_executable(gateway src/main.cpp)
target_link_libraries(gateway PRIVATE httplib)
```

```cpp gateway/src/main.cpp
#include <httplib.h>
#include <nlohmann/json.hpp>
#include <iostream>
#include <string>

// 服务地址(最简服务发现:配置文件替代注册中心)
const std::string BLOG_SERVER_HOST = "127.0.0.1";
const int BLOG_SERVER_PORT = 8081;

int main() {
    httplib::Server svr;

    // 健康检查端点:网关自身 + 后端健康状况
    svr.Get("/health", [](const httplib::Request& req, httplib::Response& res) {
        nlohmann::json result;
        result["gateway"] = "ok";

        httplib::Client client(BLOG_SERVER_HOST, BLOG_SERVER_PORT);
        if (auto backend = client.Get("/health")) {
            if (backend->status == 200) {
                result["backend"] = "ok";
            } else {
                result["backend"] = "degraded";
            }
        } else {
            result["backend"] = "down";
        }

        res.set_content(result.dump(), "application/json");
    });

    // 转发所有 /api/* 请求到 Rust 博客服务
    svr.Get(R"(/api/(.*))", [](const httplib::Request& req, httplib::Response& res) {
        std::string backend_path = "/api/" + req.matches[1];

        httplib::Client client(BLOG_SERVER_HOST, BLOG_SERVER_PORT);
        if (auto backend = client.Get(backend_path)) {
            res.status = backend->status;
            res.set_content(backend->body, "application/json");
        } else {
            res.status = 502;
            res.set_content(R"({"error": "backend unreachable"})", "application/json");
        }
    });

    std::cout << "[gateway] listening on 8080\n";
    svr.listen("127.0.0.1", 8080);
    return 0;
}
```

如果你不想用 FetchContent 下载依赖,直接手动下载这两个单头文件放到 `gateway/` 目录:

```bash
# 在 gateway/ 目录下
mkdir include
curl -o include/httplib.h https://raw.githubusercontent.com/yhirose/cpp-httplib/v0.18.0/httplib.h
curl -o include/json.hpp https://raw.githubusercontent.com/nlohmann/json/v3.11.3/single_include/nlohmann/json.hpp
```

然后 CMakeLists 简化为:

```cmake
cmake_minimum_required(VERSION 3.14)
project(gateway)
set(CMAKE_CXX_STANDARD 17)
add_executable(gateway src/main.cpp)
target_include_directories(gateway PRIVATE include)
```

编译:

```bash
mkdir build && cd build
cmake ..
cmake --build .
./gateway
```

---

## 四、验证整个微服务闭环

### 启动顺序:

```bash
# 终端 1:启动 Rust 博客服务
cd server && cargo run
# 输出: blog-server listening on 127.0.0.1:8081

# 终端 2:启动 C++ 网关
cd gateway/build && ./gateway
# 输出: [gateway] listening on 8080

# 测试
curl http://localhost:8080/health
# 输出: {"gateway":"ok","backend":"ok"}

curl http://localhost:8080/api/articles
# 请求被网关转发到 Rust 服务,返回文章列表

curl http://localhost:8080/api/articles/hello-world
# 返回文章详情
```

### 微服务即时可见的演示效果:

1. **杀掉 Rust 服务**(Ctrl+C),再请求:
   ```
   curl http://localhost:8080/health
   # {"gateway":"ok","backend":"down"}  ← 网关还活着,感知到了后端挂了
   cm -shcm curl http://localhost:8080/api/articles
   # {"error": "backend unreachable"}  ← 网关返回 502,但网关没崩溃
   ```
2. **重启 Rust 服务**,请求自动恢复——这展示了微服务"独立生命周期"的核心价值。

---

## 五、这个骨架已经覆盖的微服务知识点

| 微服务核心概念 | 在这个项目中的体现                                    |
| -------------- | ----------------------------------------------------- |
| 服务拆分       | 网关(C++)和业务服务(Rust)是两个独立进程               |
| 服务间通信     | HTTP/REST(后续可升级为 gRPC)                          |
| 统一入口       | 网关是唯一对外暴露点                                  |
| 健康检查       | `/health` 端点 + 网关聚合健康状态                     |
| 故障隔离       | 后端挂了网关不受影响,返回 502                         |
| 服务发现(最简) | 配置文件写死地址(后续换 Consul)                       |
| 水平扩展(潜质) | 你可以启动两个 Rust 服务在 8081/8082,改网关配置做轮询 |

**这就是"基础微服务"——完整但极小。** 你说"基础的微服务还是要有的",现在你有了。

---

## 六、之后的扩展路径(每次只加一块)

```
✅ 现在: 网关 + 博客服务 + health + 转发
⬜ 下一步: 加注册中心(Consul/etcd)——网关从注册中心发现 Rust 服务地址
⬜ 再下一步: 内部通讯改 gRPC + Protobuf(你已有 proto 设计,直接实现)
⬜ 再下一步: 加数据库(sqlx + MySQL),文章存 DB
⬜ 再下一步: 加认证服务(单独的 Rust 服务,发 JWT)
⬜ 再下一步: 加第二业务服务(比如待办/留言),通过网关转发
⬜ 最后: OTel 追踪 + Prometheus 指标
```

每加一块都是在现有骨架上做增量,不会推翻重来。

---

## 七、目录结构汇总

```
blog-micro/
├── content/            # Markdown 文章
│   └── hello-world.md
├── server/             # Rust 博客服务(8081)
│   ├── Cargo.toml
│   └── src/main.rs
├── gateway/            # C++ 网关(8080)
│   ├── CMakeLists.txt
│   ├── include/        # httplib.h, json.hpp
│   └── src/main.cpp
└── README.md           # 写明启动方式和架构图
```

---

## 八、今天就能走通

| 步骤 | 动作                                   | 时间               |
| ---- | -------------------------------------- | ------------------ |
| 1    | 建 content/,写第一篇 md                | 5 分钟             |
| 2    | 建 server/,复制 Rust 代码,`cargo run`  | 15 分钟            |
| 3    | 建 gateway/,下载两个头文件,编译运行    | 15 分钟            |
| 4    | curl 测试 `/health` 和 `/api/articles` | 5 分钟             |
| 5    | 关掉 Rust,看网关报 502,重启恢复        | 2 分钟(微服务演示) |

**一小时后,你有一个真正可演示的微服务博客系统。** 虽然小,但架构形态是真实的。

要不要我把上面的内容整理成一个完整的 README 模板,方便你初始化项目时直接用?

# 你的 MySQL 微服务绝对没白写——它在第二阶段是核心，现在只是“先跑通界面再接入”

先把你的担忧拆掉：

| 阶段                     | 你的 MySQL 微服务的角色                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| **现在（第一阶段）**     | 暂不接入，先用文件驱动把博客界面跑通。但你的微服务代码**继续存在、继续可用**                    |
| **第二阶段（1~2 周后）** | Rust 博客服务通过 gRPC 调你的微服务，文章数据存 MySQL。这是真正的微服务闭环                     |
| **第三阶段（面试展示）** | 你可以在简历上写“C++ 微服务（gRPC + MySQL）+ Rust 业务服务”的完整架构，面试官很难不问你这个设计 |

**“先文件后数据库”不是抛弃微服务，而是为了让你在调试前端时不被 gRPC 卡住。** 微服务接入只需要 1~2 天就能完成，你现在直接跳进去反而会卡死。

况且你想想：**你自己写好了 MySQL 微服务 = 别人可能要装 MySQL + 自己写连接代码，而你只用 gRPC 一行调用就能查数据。** 这就是你比其他学生领先的地方。

---

## 现在开始：完整的第一阶段（文件驱动博客）

### 1. 项目结构

```
blog/
├── content/                  # Markdown 文件（就是你的"数据库"）
│   ├── hello-world.md
│   └── microservice.md
│
├── server/                   # Rust 后端
│   ├── Cargo.toml
│   └── src/main.rs
│
└── web/                      # Vue 前端（AI 生成）
```

### 2. 创建 `content/hello-world.md`

```markdown content/hello-world.md
---
title: Hello World
date: 2026-07-01
tags: [rust, blog]
---

# 这是我的第一篇博客

用 **Markdown** 写内容就是爽。

## 前端展示效果

这里应该渲染成标题、正文、高亮。
```

另一个：

```markdown content/microservice.md
---
title: C++ 微服务开发笔记
date: 2026-07-02
tags: [cpp, grpc]
---

# 我写的 MySQL 微服务

核心点：
- gRPC 接口
- 连接池
- 读写分离
```

### 3. `server/Cargo.toml`

```toml server/Cargo.toml
[package]
name = "blog-server"
version = "0.1.0"
edition = "2021"

[dependencies]
axum = "0.7"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

### 4. `server/src/main.rs`（完整代码）

```rust server/src/main.rs
use axum::{
    extract::{Path, State},
    http::StatusCode,
    routing::get,
    Json, Router,
};
use serde::Serialize;
use std::collections::HashMap;
use std::fs;
use std::sync::Arc;

#[derive(Serialize, Clone)]
struct ArticleMeta {
    slug: String,
    title: String,
    date: String,
    tags: Vec<String>,
}

#[derive(Serialize)]
struct ArticleDetail {
    meta: ArticleMeta,
    content: String,
}

struct AppState {
    articles: Vec<ArticleMeta>,
}

fn parse_frontmatter(raw: &str) -> (Option<String>, Option<String>, Vec<String>) {
    let mut title = None;
    let mut date = None;
    let mut tags = Vec::new();

    if let Some(rest) = raw.strip_prefix("---") {
        if let Some(end_pos) = rest.find("---") {
            let fm = &rest[..end_pos];
            for line in fm.lines() {
                if let Some(value) = line.strip_prefix("title:") {
                    title = Some(value.trim().trim_matches('"').to_string());
                }
                if let Some(value) = line.strip_prefix("date:") {
                    date = Some(value.trim().trim_matches('"').to_string());
                }
                if let Some(value) = line.strip_prefix("tags:") {
                    tags = value
                        .trim()
                        .trim_start_matches('[')
                        .trim_end_matches(']')
                        .split(',')
                        .map(|s| s.trim().trim_matches('"').to_string())
                        .filter(|s| !s.is_empty())
                        .collect();
                }
            }
        }
    }
    (title, date, tags)
}

fn scan_articles() -> Vec<ArticleMeta> {
    let content_dir = "../content";
    let mut articles = Vec::new();
    if let Ok(entries) = fs::read_dir(content_dir) {
        for entry in entries.flatten() {
            let path = entry.path();
            if path.extension().and_then(|e| e.to_str()) != Some("md") {
                continue;
            }
            let slug = path
                .file_stem()
                .and_then(|s| s.to_str())
                .unwrap_or("")
                .to_string();
            if let Ok(raw) = fs::read_to_string(&path) {
                let (title, date, tags) = parse_frontmatter(&raw);
                articles.push(ArticleMeta {
                    slug,
                    title: title.unwrap_or_else(|| "Untitled".to_string()),
                    date: date.unwrap_or_else(|| "1970-01-01".to_string()),
                    tags,
                });
            }
        }
    }
    articles.sort_by(|a, b| b.date.cmp(&a.date));
    articles
}

async fn list_articles(State(state): State<Arc<AppState>>) -> Json<Vec<ArticleMeta>> {
    Json(state.articles.clone())
}

async fn get_article(
    State(_state): State<Arc<AppState>>,
    Path(slug): Path<String>,
) -> Result<Json<ArticleDetail>, StatusCode> {
    let path = format!("../content/{}.md", slug);
    let raw = fs::read_to_string(&path).map_err(|_| StatusCode::NOT_FOUND)?;

    let (title, date, tags) = parse_frontmatter(&raw);
    let content = raw
        .strip_prefix("---")
        .and_then(|rest| rest.find("---").map(|pos| &rest[pos + 3..]))
        .unwrap_or(&raw)
        .to_string();

    Ok(Json(ArticleDetail {
        meta: ArticleMeta {
            slug,
            title: title.unwrap_or_else(|| "Untitled".to_string()),
            date: date.unwrap_or_else(|| "1970-01-01".to_string()),
            tags,
        },
        content,
    }))
}

#[tokio::main]
async fn main() {
    let articles = scan_articles();
    let state = Arc::new(AppState { articles });

    let app = Router::new()
        .route("/", get(|| async { "Blog API running" }))
        .route("/api/articles", get(list_articles))
        .route("/api/articles/:slug", get(get_article))
        .with_state(state);

    let listener = tokio::net::TcpListener::bind("127.0.0.1:8080").await.unwrap();
    println!("listening on http://{}", listener.local_addr().unwrap());
    axum::serve(listener, app).await.unwrap();
}
```

### 5. 运行它

```bash
cd server
cargo run
```

开另一个终端测试：

```bash
curl http://localhost:8080/api/articles
# 输出: [{"slug":"microservice","title":"C++ 微服务开发笔记","date":"2026-07-02","tags":["cpp","grpc"]},...]

curl http://localhost:8080/api/articles/hello-world
# 输出: {"meta":{...},"content":"# 这是我的第一篇博客..."}
```

**到这里,后端已经工作。你的博客已经能通过 API 提供文章了。**

---

## 6. 给 AI 的 Vue 前端提示词（同样粘贴给 AI 就行）

```
请生成 Vue 3 + Vite 前端,连接 http://localhost:8080

需要两个页面:

1. 首页(文章列表):
   GET /api/articles 返回 [{slug,title,date,tags}]
   卡片式展示,点击标题跳详情

2. 详情页 /article/:slug:
   GET /api/articles/:slug 返回 {meta:{slug,title,date,tags}, content:"markdown原文"}
   用 markdown-it 渲染 content 为 HTML

技术: Vue3 Composition API, vue-router, fetch, 简洁现代样式
生成文件: package.json, vite.config.js, index.html, src/main.js,
src/App.vue, src/router.js, src/api.js,
src/views/ArticleListView.vue, src/views/ArticleDetailView.vue
```

生成后:

```bash
cd web
npm install
npm run dev
```

浏览器打开 `http://localhost:5173`,你就有一个带文件系统的博客了。

---

## 第二阶段:接入你的 MySQL 微服务(明确路线图,让你的 C++ 微服务登场)

**这一步在博客界面跑通后进行,预计只需 1~2 天:**

| 步骤 | 做什么                                                                  |
| ---- | ----------------------------------------------------------------------- |
| 1    | 把 posts 表建好(`id, title, date, tags, content`)                       |
| 2    | 在 Rust 服务里加 tonic 依赖,生成一个 gRPC 客户端调用你的 C++ 微服务     |
| 3    | 改 `list_articles` 和 `get_article`:从 microservice 查数据,而不是读文件 |
| 4    | 你的 C++ 微服务成了唯一数据源,文件驱动退役!                             |

**那时你就有完整的架构:**
```
前端 → Rust 业务服务 →（gRPC）→ C++ MySQL 微服务 → MySQL
```

面试时,你可以说:

> "我有一个 C++ 实现的 MySQL 微服务(连接池、读写分离、gRPC 暴露),还有一个 Rust 实现的业务服务(Axum + tonic)。两者通过 gRPC 通讯,前端部署 Vue3。整体是一个微服务架构的最小实例,我完整参与设计并实现了这条链路。"

**这句话的重量,远超一万行 CRUD 代码。**

---

## 今天的任务

1. 创建 `content/`,写两篇 md
2. 复制上面的 `main.rs` 和 `Cargo.toml`
3. `cargo run`,curl 测试 API 返回正确
4. 让 AI 生成 Vue 前端,跑通界面
5. 截图记录——今天你就有一个能看的博客了

只要这一步走完,第二阶段的微服务接入就是水到渠成。你的 C++ 微服务不是白写,它是为这个博客准备的"核弹",第二阶段正式引爆。