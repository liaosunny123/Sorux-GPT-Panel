# Sorux-GPT-Panel
SoruxGPT 面板管理，支持多号多节点 ChatGPT 共享管理。  

交流群：168626495  

特别注意：本 Panel 不允许任何未经许可的商业化授权，如果需要商业化授权请联系 epicmocn@gmail.com。  

## Docker 镜像部署
> 以下为社区版本的镜像资源。特别注意：社区版本比商业版本阉割了部分内容。  

> 如果需要搭建帮助，可以有偿联系 epicmocn@gmail.com。

```bash
docker pull epicmo/soruxgpt_community:latest
```

## 部署文档

1. 先建立你的文件夹

```’bash
mkdir SoruxGPT
cd SoruxGPT
mkdir data # 用于存储面板数据
```

2. 新建 Docker-Compose 文件，并部署

Docker-Compose

```yaml
version: "3.9"
services:
  redis:
    container_name: "SoruxGPT-Redis"
    image: redis
    restart: unless-stopped
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 10s
      timeout: 5s
      retries: 3
  jaeger:
    container_name: "SoruxGPT-Jaeger"
    image: jaegertracing/all-in-one
    environment:
      - COLLECTOR_OTLP_ENABLED=true
    # 如果你不需要使用链路追踪，请注释这些端口
    #ports:
    #  - "16686:16686"
    #  - "14268:14268"
    #  - "14250:14250"
    #  - "6831:6831"
    healthcheck:
      test: [ "CMD-SHELL", "wget --spider -q http://localhost:16686/search || exit 1" ]
      interval: 10s
      timeout: 5s
      retries: 3
  sorux-gpt:
    container_name: "SoruxGPT"
    image: "epicmo/soruxgpt_community:latest"
    ports:
      - "5700:8080"
    env_file:
      - .env.docker.compose
    volumes:
      - "./gorm.db:/sorux/gpt/gorm.db"
    depends_on:
      redis:
        condition: service_healthy
      jaeger:
        condition: service_healthy
volumes:
  share-volume:
```

3. 新建配置文件(.env.docker.compose)：

```yaml
# Configure Log settings
# Optional values：DEBUG, INFO, WARN, ERROR, FATAL, TRACE, default as INFO
LOG_LEVEL=INFO
# Configure log redirect to file, for conveniently collect log file
# Optional values: enable, disable, default as disable
LOG_REDIRECT_TO_FILE=disable
# Configure log output dir, default as /var/log/sorux/gpt, have effect, when LogRedirectToFile is equal to enable
LOG_PATH=/var/log/sorux/gpt
# Configure the trace with state
LOGGER_WITH_TRACE_STATE=enable
# Otel settings
LOGGER_WITH_TRACE_STATE=disable
OTEL_SAMPLER=0.01
TracingEndPoint=jaeger:4318
# Configure backend running port
BINDING_BACKEND_PORT=5700
REDIS_ADDR=redis:6379
```

默认账号密码为：admin admin

4. 对于蟑螂，需要在配置处修改为以下内容，然后在管理面板添加节点即可。注意，添加节点的时候需要保证域名的格式为：xxx.xxx.xxx，例如 a.ai.com，不需要添加https/http

```yaml
OAUTH_URL: "https://{SoruxGPT 域名}/api/oauth?client_tag={代理这个节点的域名}"
LOGIN_CALLBACK: "https://{SoruxGPT 域名}/login"
ADMIN_PASSWORD: "{管理员密码}"
AUDIT_LIMIT_URL: "https://{SoruxGPT 域名}/api/audit?client_tag={代理这个节点的域名}"

USERTOKENS:
  - "{管理员密码}"

cool:
  autoMigrate: true

# sqlite数据库配置
database:
  default:
    type: "sqlite" # 数据库类型
    name: "./config/cool.sqlite" # 数据库名称,对于sqlite来说就是数据库文件名
    extra: busy_timeout=5000 # 扩展参数 如 busy_timeout=5000&journal_mode=ALL
    createdAt: "create_time" # 创建时间字段名称
    updatedAt: "update_time" # 更新时间字段名称
    # debug : true # 开启调试模式,启用后将在控制台打印相关sql语句
```

## Feature

- 节点管理：支持多个节点的调度管理
- 用户管理：支持用户管理
- 聊天审计：支持聊天内容审计，关键词检查
- 速率限制：支持对用户单独限速，对节点限速
- 聊天汇总查看：支持后台查看全部消息记录
- 内置虚拟货币系统：支持内置虚拟货币系统
- 等等...

## 基于面板你可以做的事情

- 给每个用户单独设置消息速率
- 给每个节点设置是否公益，是否限速，是否是 Plus
- 提供违禁词列表进行审查
- 通过虚拟货币和 API 对接发卡网或者你自己的商店逻辑
- 给每个用户分配单独的节点（如果你是车队那就全部分配节点，如果你是独购那就分配一个单独的节点）
- 等等，可以完成市面上基本上全部的 GPT 面板的运营逻辑

## 截图

![节点列表](1.png)

![用户管理](2.png)

![聊天界面](4.png)

## 社区版限制

- 用户数不能超过8
- 节点数不能超过1
- 不支持设备数限制
- 不支持违禁提问检查
- 不支持按用户单独限制消息频率
- 只支持创建公益节点
- 不支持统计 API
- 不支持审计
- 等等
