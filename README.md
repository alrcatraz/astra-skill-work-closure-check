# astra-skill-work-closure-check

任务收尾时强制执行的闭环检查清单 Hermes Agent skill。提供六阶段系统化检查：凭证扫描、技能更新、决策记录、服务登记、记忆准确性、环境基线对比。

## 功能

- 六阶段闭环检查：凭证泄露 → 技能更新 → 决策记录 → 服务/设备登记 → 记忆准确性 → 环境基线对比
- 标准化收尾流程：通知 → 等待确认 → 清理 → 汇总 → 再问
- 防坑提醒：不跳过确认、不混淆 skill/记忆、凭证跨 session 残留检查

## 安装

将 `SKILL.md` 复制到 Hermes profile 的 `skills/` 目录下：

```bash
cp SKILL.md ~/.hermes/profiles/default/skills/work-closure-check.md
```

## License

MIT — 详见 [LICENSE](LICENSE)
