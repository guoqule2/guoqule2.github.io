# ai-agent-base · Maven Archetype

> 一条命令生成六模块的 AI Agent 工程：**Google ADK + Spring AI + DeepSeek**。
> 生成的工程自带装配链、SSE 流式对话、多模态（图片）支持、MCP 工具接入。

```
api / app / domain / trigger / infrastructure / types
```

---

## 这个仓库是什么

这里同时提供**两个东西**，缺一不可：

| 角色 | 文件 | 作用 |
|---|---|---|
| **只读 Maven 仓库** | `org/example/ai-agent-base-archetype/` | 存 archetype 的 jar，Maven 从这里下载模板 |
| **archetype catalog** | `archetype-catalog.xml` | 模板清单，让 IDEA 的下拉框能列出这个脚手架 |

> GitHub Pages 本质是免费静态托管，而 Maven 仓库本身就是「按坐标摆好的静态文件」，
> 所以它能直接当只读仓库用 —— **公开可读，不需要账号密码**。

---

## 快速开始（3 步）

### 第 1 步：配置 `settings.xml`（必须，否则第二步一定失败）

Maven 默认只认 Maven Central，不知道这个仓库的存在。不加配置会报：

```
Archetype not found in any of the configured repositories
```

编辑 `~/.m2/settings.xml`（Windows：`C:\Users\你的用户名\.m2\settings.xml`），在 `<settings>` 内加入：

```xml
<profiles>
    <profile>
        <id>guoqule2-pages</id>
        <repositories>
            <repository>
                <id>guoqule2-pages</id>
                <url>https://guoqule2.github.io/</url>
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>false</enabled></snapshots>
            </repository>
        </repositories>
        <pluginRepositories>
            <pluginRepository>
                <id>guoqule2-pages</id>
                <url>https://guoqule2.github.io/</url>
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>false</enabled></snapshots>
            </pluginRepository>
        </pluginRepositories>
    </profile>
</profiles>

<activeProfiles>
    <activeProfile>guoqule2-pages</activeProfile>
</activeProfiles>
```

> **已有 profiles 的话不要整份覆盖**，把上面两段**追加/合并**进去即可。
> 覆盖会丢掉你自己的 `<localRepository>` 和其他仓库配置。

---

### 第 2 步：生成项目

#### 方式 A：命令行（推荐，稳定）

```bash
mvn archetype:generate \
  -DarchetypeGroupId=org.example \
  -DarchetypeArtifactId=ai-agent-base-archetype \
  -DarchetypeVersion=1.0.3 \
  -DgroupId=com.yourcompany.ai \
  -DartifactId=my-agent \
  -Dversion=1.0.0 \
  -Dpackage=com.yourcompany.ai \
  -DinteractiveMode=false
```

> **离线环境**要显式指定插件版本，否则报 `Plugin not found`：
> `mvn -o org.apache.maven.plugins:maven-archetype-plugin:3.2.0:generate ...`

#### 方式 B：IDEA 界面

1. `File → New → Project → Maven Archetype`
2. **Manage catalogs... → `+`**，填：

   | 字段 | 值 |
   |---|---|
   | Name | `guoqule2-pages` |
   | Type | **Remote** |
   | URL | `https://guoqule2.github.io/archetype-catalog.xml` |

3. 回到 Archetype 下拉框，选 `guoqule2-pages`
4. 选中 `ai-agent-base-archetype`，填 GroupId / ArtifactId / Version / Package

> 如果 IDEA 里点 Create 后卡住或 502：去
> `Settings → Appearance & Behavior → System Settings → HTTP Proxy`
> 的 **Proxy exclusion** 里加上 `localhost,127.0.0.1`。

---

### 第 3 步：配置环境变量并启动

生成的工程里，模型和 MCP 的密钥都走环境变量，**不写死在 yml 里**。

| 变量 | 必需 | 说明 |
|---|---|---|
| `DEEPSEEK_API_KEY` | ✅ | DeepSeek 的 key，模型调用靠它 |
| `BAIDU_MCP_API_KEY` | ⭕ 可选 | 百度 AI 搜索 MCP，要用联网搜索才需要 |
| `BAIDU_MAP_AK` | ⭕ 可选 | 百度地图 MCP |
| `MYSQL_PASSWORD` | ⭕ 可选 | 默认 `123456`，连库时用 |

**Windows 设置（用户级，持久）**

```powershell
[Environment]::SetEnvironmentVariable('DEEPSEEK_API_KEY','sk-你的key','User')
```

**Linux / macOS**

```bash
export DEEPSEEK_API_KEY="sk-你的key"
# 永久生效写进 ~/.bashrc 或 ~/.zshrc
```

设完**重启 IDEA**（或新开终端），否则读不到。

启动：

```bash
cd my-agent
mvn -pl my-agent-app spring-boot:run
```

> **百度 MCP 默认是注释掉的**。要用联网搜索，把
> `my-agent-app/src/main/resources/Agent/onlyone-agent.yml` 里 `tool-mcp-list` 下的
> `# - sse:` 那几行取消注释，并配好 `BAIDU_MCP_API_KEY`。

---

## 生成的工程结构

```
my-agent/
├── my-agent-api/              对外契约：DTO + IAgentService 接口
├── my-agent-app/              启动模块：Application + yml 配置
├── my-agent-domain/           领域层：装配链、ChatService、MCP 工具
├── my-agent-trigger/          HTTP 层：Controller
├── my-agent-infrastructure/   基础设施层
└── my-agent-types/            通用类型：Response / 异常 / 枚举
```

分层依赖方向：`trigger → api`、`app → domain/trigger/infrastructure`、`domain → types`。

**对外接口**（`trigger` 模块，启动后可用）：

| 方法 | 路径 | 作用 |
|---|---|---|
| GET | `/api/v1/query_ai_agent_config_list` | 查询可用智能体 |
| POST | `/api/v1/create_session` | 创建会话 |
| POST | `/api/v1/chat` | 同步对话 |
| POST | `/api/v1/chat_stream` | SSE 流式对话 |

---

## 版本

| 版本 | 说明 |
|---|---|
| `1.0.3` | **当前版本**。修复模块依赖 groupId / version 参数化，清除硬编码密钥 |
| `1.0.2` | 修复 groupId 与 dependencyManagement 版本 |
| `1.0.1` | 仅修复 groupId（依赖版本仍错，勿用）|
| `1.0.0` | 初版，依赖坐标不可用 |

> release 仓库**不允许覆盖同名版本**，升级请递增版本号。

---

## 常见问题

**Q：报 `Archetype not found in any of the configured repositories`**

第 1 步没做，或做完没重启 IDEA。settings 改完要点 `Reload All Maven Projects`。

**Q：报 `Unresolved dependency: 'org.example:xxx-api'`**

用了 `1.0.0` 或 `1.0.1`，这两个版本的 pom 有硬编码缺陷，换 `1.0.3`。

**Q：报 `Could not resolve placeholder 'DEEPSEEK_API_KEY'`**

环境变量没设，或设完没重启进程。

**Q：启动时卡在 MCP initialize / 超时**

说明你启用了百度 MCP 但 key 无效。要么把 `# - sse:` 那几行保持注释，要么换上你自己的 key。

**Q：`401 Unauthorized`**

这个仓库是公开只读的，不需要认证。出现 401 一般是 `settings.xml` 里 `<server>` 的 id 与 repository 的 id 冲突，检查是否有同名 server 配置了错误密码。

---

## 相关链接

- Catalog：https://guoqule2.github.io/archetype-catalog.xml
- 仓库根：https://guoqule2.github.io/
- 模板内使用的框架：[Google ADK](https://google.github.io/adk-docs/) · [Spring AI](https://docs.spring.io/spring-ai/reference/)

## License

Apache-2.0
