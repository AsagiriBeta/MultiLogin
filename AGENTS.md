# AGENTS.md

## 项目概览

MultiLogin 是面向 **Minecraft Velocity 代理** 的 Java 插件（Gradle 多模块 monorepo）。可交付产物为 `velocity/build/libs/MultiLogin-Velocity-*.jar`。仓库内**没有**可启动的 Web 服务、Docker Compose 或集成测试用的 Velocity/后端服。

## 开发环境

- **JDK 21**（必须）。本机用 `java -version` 确认。
- **构建工具**：仓库自带 `./gradlew`（Gradle Wrapper），无需全局安装 Gradle。
- **Maven 仓库**：根目录 `repositories` 文件列出远程仓库；若依赖解析失败，可参考 `.github/workflows/build.yml` 中的 `Patch Gradle Repositories` 步骤（移除阿里云镜像、确保 Central 存在）。

## 常用命令

| 目的 | 命令 |
|------|------|
| CI 同等构建与测试 | `./gradlew --no-daemon test shadowJar` |
| 发布风格版本号 JAR | `./gradlew --no-daemon shadowJar -Denv=final` |
| 清理后全量构建 | `./gradlew --no-daemon clean test shadowJar` |
| 仅拉取/刷新依赖 | `./gradlew --no-daemon dependencies` |

产物路径：`velocity/build/libs/MultiLogin-Velocity-*.jar`（主插件）；其它模块 JAR 在各自 `*/build/libs/`。

## 测试与 Lint

- **单元测试**：各模块 `test` 任务当前为 `NO-SOURCE`；CI 仍执行 `test` 以保持流程一致。
- **Lint**：仓库未配置 Checkstyle/Spotless 等；以 `./gradlew test shadowJar` 作为质量门禁。

## 端到端（游戏内）验证

在 VM 内通常**无法**完整跑通登录流程（需要 Velocity、后端 Minecraft 服、外置 Yggdrasil 等）。本地/云代理开发以 **成功 shadowJar + 检查 `velocity-plugin.json`** 为准。插件元数据位于 JAR 根目录的 `velocity-plugin.json`（可用 `unzip -p velocity/build/libs/MultiLogin-Velocity-*.jar velocity-plugin.json` 查看）。

## Cursor Cloud specific instructions

- **无需**在 update 脚本中启动 Velocity、MySQL 或任何游戏服；update 脚本只负责依赖刷新。
- 若 `./gradlew` 首次下载依赖较慢，属正常现象；网络受限时可按 workflow 修补 `repositories` 后再构建。
- **不要**依赖 `velocity/build.gradle` 中的 `cloneVelocity` / `getLatestVelocity` 做日常构建；CI 与常规开发均通过 Maven（`repo.papermc.io` 等）解析 Velocity API。
- 构建成功后，用 `ls velocity/build/libs/MultiLogin-Velocity-*.jar` 与 `unzip -p … velocity-plugin.json` 快速确认环境可用。
- 项目 README 标明已停止维护；环境搭建目标仍是可编译、可打 JAR，而非部署生产代理。
