# StoryCraft 代码规范与质量检查规范

本文档用于统一 StoryCraft 项目的代码风格和质量标准，确保代码可读、可维护、可协作。

> 本规范以 Python 为主要参考。若后续技术栈调整，可在此基础上进行相应替换。

## 1. 目标

- 提高代码可读性和一致性
- 降低代码维护成本
- 在代码进入 `develop` 或 `main` 前拦截明显问题
- 明确 review 时的质量门禁标准

## 2. 通用代码规范

### 2.1 命名规范

| 类型 | 规范 | 示例 |
|---|---|---|
| 模块/包 | 小写，用下划线分隔 | `story_outline.py`, `character_agent` |
| 类 | 大驼峰 | `StoryEngine`, `CharacterProfile` |
| 函数/方法 | 小写，用下划线分隔，动词开头 | `generate_outline()`, `validate_scene()` |
| 常量 | 全大写，下划线分隔 | `MAX_RETRY_COUNT`, `DEFAULT_TEMPERATURE` |
| 变量 | 小写，用下划线分隔 | `scene_count`, `prompt_template` |
| 私有成员 | 以下划线开头 | `_internal_helper()`, `_config_cache` |

- 命名应具有描述性，避免使用 `a`、`b`、`tmp`、`x1` 等无意义名称。
- 避免缩写，除非该缩写是领域内广泛认可的（如 `LLM`、`API`、`HTTP`）。

### 2.2 代码结构

- 每个模块只负责一个明确职责，避免“万能文件”。
- 函数/方法应单一职责，长度控制在 50 行以内；过长时应拆分。
- 类内部顺序建议：类变量 → `__init__` → 公共方法 → 私有方法 → 特殊方法。
- 导入顺序：标准库 → 第三方库 → 本项目模块，每组之间空一行。
- 避免循环导入；如无法避免，采用延迟导入或调整模块边界。

### 2.3 注释与文档

- 所有公共函数、类、方法必须包含文档字符串（docstring），说明：功能、参数、返回值、可能抛出的异常。
- 注释应解释“为什么”，而不是复述“做了什么”。
- 避免大段注释掉的代码；如需保留，说明原因或移除。
- 复杂算法、业务规则或临时方案应加注释说明背景。

示例：

```python
def generate_outline(theme: str, chapters: int = 10) -> str:
    """根据主题生成小说大纲。

    Args:
        theme: 小说主题或核心概念。
        chapters: 预计章节数，默认 10。

    Returns:
        结构化的大纲文本。

    Raises:
        ValueError: 当 theme 为空或 chapters 小于 1 时。
    """
```

### 2.4 错误处理

- 对可预期的错误使用异常或显式返回值，避免静默吞掉异常。
- 捕获异常时应尽量精确，不要直接用裸 `except:` 或 `except Exception:` 覆盖所有情况。
- 对外部服务（如 LLM API）调用应设置超时、重试和降级策略。

### 2.5 日志与输出

- 使用日志模块（如 Python 的 `logging`）而不是 `print` 输出调试信息。
- 日志级别使用规范：`DEBUG` 调试信息、`INFO` 关键流程、`WARNING` 可恢复问题、`ERROR` 需要处理的问题。
- 禁止在代码中硬编码 API Key、Token、密码等敏感信息。

### 2.6 依赖最小化

StoryCraft 遵循**最小依赖原则**：

- 运行时依赖应控制在最低限度，当前框架级程序仅依赖 `langgraph`。
- 优先使用标准库或项目内部模块实现功能，避免引入第三方依赖。
- 每新增一个运行时依赖，必须有明确理由，并记录在 `docs/development/TECH_STACK.md` 中。
- 导入顺序保持：标准库 → 第三方库 → 本项目模块，便于识别外部依赖范围。

## 3. Python 专用规范（推荐）

- 遵循 [PEP 8](https://peps.python.org/pep-0008/) 风格。
- 使用类型注解（Type Hints），提高可读性和静态检查效果。
- 行长度限制为 100 个字符（可在工具配置中调整）。
- 字符串优先使用双引号，保持一致。
- 使用 f-string 进行字符串格式化，避免 `%` 和 `format()` 的过度使用。

## 4. 代码格式化

- 使用自动化格式化工具统一风格，避免在 review 中争论格式问题。
- 推荐工具：
  - Python：`ruff format` 或 `black`
  - 其他语言：根据官方主流工具选择（如 `prettier` 等）
- 所有提交代码在合并前必须通过格式化检查。

## 5. 静态检查（Lint）

- 使用静态检查工具拦截语法错误、未使用变量、导入错误、命名不规范等问题。
- 推荐工具：
  - Python：`ruff check` 或 `flake8`
- 检查规则应保持严格但合理，避免误报过多导致团队忽视告警。
- 不得通过注释（如 `# noqa`）随意绕过规则，除非有明确理由并在注释中说明。

## 6. 类型检查

- 强烈推荐使用类型注解，并运行类型检查工具。
- 推荐工具：
  - Python：`mypy` 或 `pyright`
- 对于第三方库缺失类型信息的情况，可逐步添加 `typing.Any` 或 stub 文件，并在注释中说明。
- 类型检查应作为 PR 合并前的必要检查项。

## 7. 测试规范

- 所有功能代码都应配套测试，核心业务逻辑必须有单元测试。
- 测试文件命名：`test_<模块名>.py`。
- 测试函数命名：`test_<被测行为>`，描述清晰。
- 测试应独立、可重复、不依赖外部真实服务；需要外部服务时使用 mock。
- 测试覆盖率应逐步达到 70% 以上，核心模块优先。

示例：

```python
def test_generate_outline_with_empty_theme_raises():
    with pytest.raises(ValueError):
        generate_outline("")
```

## 8. 提交前检查

提交代码前，至少应运行以下检查：

1. 格式化：`ruff format . --check`（或对应工具）
2. 静态检查：`ruff check .`
3. 类型检查：`mypy src/`
4. 测试：`pytest`

后续项目接入 pre-commit 或 CI 时，上述检查将自动运行。当前阶段可在本地手动执行。

## 9. PR 质量门禁

Pull Request 合并前必须满足以下条件：

- 代码已通过格式化检查
- 静态检查无错误
- 类型检查通过（后续接入后适用）
- 测试全部通过
- 至少一名 reviewer 批准
- 无未解决的 review 评论
- 文档或注释已同步更新（如接口发生变更）

## 10. 配置文件约定

- 配置模板存放在 `configs/` 目录下，文件内使用占位符而非真实值。
- 敏感配置（API Key、密钥等）通过环境变量或本地配置文件加载，且不提交到仓库。
- 示例配置应包含注释说明每个字段的用途。
- 推荐在仓库中提供 `.env.example` 或 `config.example.yaml` 文件。

## 11. 工具配置示例（Python）

以下配置供参考，后续可根据实际技术栈落地为正式文件。

### 11.1 `pyproject.toml`（推荐）

以下配置与项目当前实际配置一致，运行时依赖保持最小化：

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "storycraft"
version = "0.1.0"
description = "StoryCraft 是一个用于小说生成的 AI Agent。"
requires-python = ">=3.10"
readme = "README.md"
license = { text = "MIT" }

# 最小依赖原则：运行时仅依赖 LangGraph
dependencies = [
    "langgraph",
]

[project.optional-dependencies]
dev = [
    "pytest",
    "ruff",
    "mypy",
]

[tool.ruff]
line-length = 100
select = ["E", "F", "I", "W", "UP"]

[tool.mypy]
python_version = "3.10"
strict = true
```

### 11.2 常用检查命令

```bash
# 格式化检查
ruff format . --check

# 自动修复部分问题
ruff check . --fix

# 类型检查
mypy src/

# 运行测试
pytest
```

## 12. 附录：Review 检查清单

Reviewer 在 review 时至少关注：

- [ ] 功能是否正确实现，边界情况是否处理
- [ ] 是否有重复代码或可提取的公共逻辑
- [ ] 命名是否清晰、一致
- [ ] 是否有合适的错误处理和日志
- [ ] 是否包含必要的测试
- [ ] 文档和注释是否同步更新
- [ ] 是否有敏感信息泄露风险

## 相关文档

- [协作说明](./COLLABORATION.md)
- [技术栈与依赖说明](./TECH_STACK.md)
- [项目说明](../../README.md)
