# Multi-Agent Travel Planner 多 Agent 智能行程规划系统

> 基于多 Agent 协作的智能旅行规划系统，支持用户偏好解析、目的地推荐、航班/酒店/活动并行搜索、预算校验与自动调整。

本项目围绕自由行规划中信息分散、组合决策复杂、预算约束难控制的问题，设计了一套多 Agent 协作流程。用户输入预算、出发城市、旅行日期、人数和旅行风格后，系统会自动生成目的地推荐、航班方案、酒店方案、每日活动安排和预算明细。

项目提供 Python、Java、Go 三种实现，其中 Python 版本用于快速验证 Agent 编排逻辑，Java 和 Go 版本用于对比不同后端技术栈下的并行编排实现方式。

## 项目背景

传统旅行规划通常需要用户在多个平台之间反复切换：

- 在目的地攻略中筛选适合季节和预算的城市；
- 在航班、酒店、活动平台之间手动比价；
- 需要不断计算总预算，发现超预算后再手动降级；
- 多人出行时还需要考虑旅行风格、活动偏好和行程节奏。

本项目将旅行规划拆解为多个相对独立的子任务，并交给不同 Agent 处理。编排层负责串联 Agent、并行执行搜索任务，并在预算超出时触发自动调整。

## 核心能力

- **偏好解析**：解析预算、出发城市、旅行时间、人数、旅行风格和兴趣偏好。
- **目的地推荐**：根据季节、预算、旅行风格和目的地特征推荐候选城市。
- **并行资源搜索**：航班、酒店和活动三个 Agent 并行执行，降低端到端规划延迟。
- **预算控制**：Budget Agent 汇总费用并判断是否超预算。
- **渐进式调整**：当总费用超出预算时，自动降低酒店、航班或活动档位，并最多执行多轮调整。
- **多语言实现**：提供 Python、Java、Go 三种实现，便于对比不同语言的并发模型和工程组织方式。
- **API 与前端**：Python 版本提供 FastAPI 接口和 Streamlit 页面，便于本地演示。

## Agent 设计

| Agent | 职责 | 输入 | 输出 |
| --- | --- | --- | --- |
| Preference Agent | 补全并标准化用户偏好 | 原始用户请求 | 结构化偏好 |
| Destination Agent | 推荐候选目的地 | 用户偏好 | 目的地列表与推荐理由 |
| Flight Agent | 搜索并选择航班 | 出发地、目的地、日期 | 航班方案 |
| Hotel Agent | 匹配住宿方案 | 目的地、日期、风格、人数 | 酒店方案 |
| Activity Agent | 生成每日活动安排 | 目的地、天数、兴趣偏好 | 日程安排 |
| Budget Agent | 预算校验与调整建议 | 航班、酒店、活动费用 | 预算明细与调整建议 |

## 系统架构

```text
用户请求
  │
  ▼
Preference Agent
  │
  ▼
Destination Agent
  │
  ├───────────────┬───────────────┐
  ▼               ▼               ▼
Flight Agent   Hotel Agent    Activity Agent
  │               │               │
  └───────────────┴───────────────┘
                  │
                  ▼
            Budget Agent
                  │
          ┌───────┴────────┐
          ▼                ▼
      预算通过          超出预算
          │                │
          ▼                ▼
      输出行程       降级策略并重新规划
```

编排策略：

- **Pipeline**：偏好解析、目的地推荐、预算校验等步骤按顺序执行。
- **Parallel**：航班、酒店、活动搜索互不依赖，使用并行方式执行。
- **Budget Loop**：预算超出时触发调整循环，避免一次性生成不可用方案。

## 技术栈

| 版本 | 技术栈 | 说明 |
| --- | --- | --- |
| Python | FastAPI、Streamlit、Pydantic、asyncio、pytest | 主力演示版本，适合快速验证 Agent 编排 |
| Java | Spring Boot、CompletableFuture、Maven | 后端服务版本，适合 Java 工程化表达 |
| Go | Gin、goroutine、WaitGroup | 并发模型清晰，适合轻量服务实现 |

## 项目结构

```text
multi-agent-travel-planner/
├── docs/                         # 架构、代码与面试整理文档
├── python/                       # Python 实现
│   ├── agents/                   # 6 个 Agent
│   ├── orchestrator/             # Pipeline、并行执行、预算循环
│   ├── tools/                    # 航班/酒店/活动等模拟搜索工具
│   ├── api/                      # FastAPI 接口
│   ├── ui/                       # Streamlit 前端
│   └── tests/                    # 单元测试
├── java/                         # Java Spring Boot 实现
├── golang/                       # Go Gin 实现
├── plan.md                       # 规划与实现记录
└── README.md
```

## 快速启动

### Python 版本

```bash
cd python
pip install -r requirements.txt

# CLI 演示
python main.py

# 自定义参数
python main.py --budget 15000 --departure 上海 --start 2026-06-01 --end 2026-06-07 --style luxury --travelers 2

# 启动 API 服务
python -m api.app

# 启动 Streamlit 前端
streamlit run ui/streamlit_app.py

# 运行测试
python -m pytest tests/ -v
```

### Java 版本

```bash
cd java
mvn spring-boot:run
```

测试接口：

```bash
curl -X POST http://localhost:8080/api/plan \
  -H "Content-Type: application/json" \
  -d '{"budget": 10000, "departureCity": "北京", "startDate": "2026-05-01", "endDate": "2026-05-05"}'
```

### Go 版本

```bash
cd golang
go mod tidy
go run ./cmd/server
```

测试接口：

```bash
curl -X POST http://localhost:8080/api/plan \
  -H "Content-Type: application/json" \
  -d '{"origin_city": "上海", "duration_days": 5, "budget_cny": 12000, "travel_style": "comfort"}'
```

## API 示例

### POST /api/plan

请求示例：

```json
{
  "budget": 10000,
  "departure_city": "北京",
  "start_date": "2026-05-01",
  "end_date": "2026-05-05",
  "travel_style": "comfort",
  "num_travelers": 1,
  "interests": ["美食", "历史"],
  "notes": ""
}
```

响应示例：

```json
{
  "destination": "首尔",
  "country": "韩国",
  "flight_cost": 3281,
  "hotel_cost": 1580,
  "activity_cost": 2270,
  "total_cost": 7131,
  "budget": 10000,
  "within_budget": true,
  "adjustment_rounds": 0,
  "hotel_name": "首尔精品设计酒店",
  "days": 4,
  "highlights": ["景福宫", "明洞", "北村韩屋村", "南山塔"]
}
```

## 运行说明

默认实现使用 Mock 数据，不依赖真实航班、酒店或地图平台 API，因此可以本地直接运行和测试。Mock 数据主要用于验证多 Agent 编排、预算控制和接口结构。

如果需要接入真实服务，可以在 `tools/` 层替换数据来源，例如：

- 航班搜索 API；
- 酒店搜索 API；
- 地图/地点推荐 API；
- 天气查询 API；
- 大模型服务 API。

## 设计亮点

### 1. 任务拆分清晰

将复杂旅行规划拆成偏好、目的地、航班、酒店、活动和预算六个子任务，每个 Agent 只关注单一职责，降低模块耦合。

### 2. 并行搜索降低延迟

航班、酒店和活动搜索相互独立，因此编排层使用并行执行方式聚合结果。Python 版本使用 `asyncio.gather`，Java 版本使用 `CompletableFuture`，Go 版本使用 `goroutine + WaitGroup`。

### 3. 预算循环提高方案可用性

行程规划不是一次性生成文本，而是要满足预算约束。系统在预算超出时触发降级策略，例如降低酒店档位、缩减高价活动或调整航班选择，直到预算满足或达到最大调整轮数。

### 4. 多语言实现便于工程对比

同一套业务流程分别用 Python、Java 和 Go 实现，可以对比不同语言在数据建模、并行编排、接口设计和测试组织上的差异。

## 面试说明

如果被问到项目数据是否真实，可以这样回答：

> 当前项目默认使用 Mock 数据，主要用于验证多 Agent 编排、并行搜索和预算控制逻辑。实际落地时，航班、酒店、活动和天气都可以通过工具层替换为真实 API，核心编排流程不需要大改。

如果被问到为什么拆成多个 Agent，可以这样回答：

> 旅行规划包含偏好理解、目的地推荐、航班搜索、酒店匹配、活动安排和预算控制等异构任务。拆成多个 Agent 后，每个模块职责清晰，也方便并行执行和单独替换工具实现。

如果被问到项目目前的重点，可以这样回答：

> 这个项目的重点不是调用某一个真实旅游 API，而是验证多 Agent 工作流如何把复杂任务拆解、并行执行，并通过预算循环保证最终方案满足约束。

## 注意事项

- 不要提交真实 API Key 或个人旅行数据。
- 当前默认使用 Mock 数据，适合本地演示和测试。
- `docs/` 目录包含架构、代码讲解和项目复盘材料，可作为理解项目的辅助资料。

## License

MIT
