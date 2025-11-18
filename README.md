# 深圳天气 · GitHub 自动更新

> 这个仓库会**每小时**自动拉取深圳天气并更新 README 中的天气卡片（使用 [Open‑Meteo](https://open-meteo.com/) 免费 API，无需密钥）。

![Workflow Status](https://img.shields.io/badge/Weather%20Updater-active-success?logo=github)

---

## 👀 快速查看

> 数据源：Open‑Meteo，时区已设为 **Asia/Shanghai**（中国标准时间）。

<!-- WEATHER-START -->
数据更新：`2025-11-18T09:00 GMT+8`

☁️ **深圳 当前** · 阴 ｜ 16.7°C（体感 13.5°C）｜ 湿度 60% ｜ 风速 20.2 kmh ｜ 云量 94% ｜ 降水 0.0 mm
☁️ **今日** · 阴 ｜ 13.2°C ~ 23.0°C ｜ 降水量 0.0 mm ｜ UV 指数 4.4

| 日期 | 天气 | 最低/最高 | 降水量 |
|---|---|---|---|
| 2025-11-19 | ☁️ 阴 | 12.2°C ~ 16.3°C | 0.0 mm |
| 2025-11-20 | ☁️ 阴 | 12.7°C ~ 18.1°C | 0.0 mm |
| 2025-11-21 | ☁️ 阴 | 14.0°C ~ 20.6°C | 0.0 mm |
<!-- WEATHER-END -->

---

## ⚙️ 工作原理
- GitHub Actions 定时运行 `scripts/fetch_weather.py`，从 Open‑Meteo 获取**当前实况 + 今日预报 + 未来 3 天**，然后回写 README 中标记区块。
- 使用内置的 `GITHUB_TOKEN` 自动提交变更（无需额外配置 Token）。

## 🚀 一键使用
1. **创建一个新仓库**（或 Fork 本仓库）。
2. 把本项目的所有文件上传到你的仓库根目录。
3. 进入 **Settings → Actions → General → Workflow permissions**，把 **Workflow permissions** 调成 **Read and write permissions**（个人仓库通常默认即可）。
4. 打开 **Actions** 页签，首次运行会提示启用，点 **Enable workflows**。
5. 等待第一次定时任务，或手动在 **Actions** 中点击 `Run workflow` 立即更新。

> **提示**：如果你想把徽章绑定到你自己的仓库，可把上面的徽章替换为：
`![Update Weather](https://github.com/<你的用户名>/<你的仓库名>/actions/workflows/update_weather.yml/badge.svg)`

## 🛠 定制
- **改城市**：编辑 `scripts/fetch_weather.py` 顶部的 `LAT`/`LON`（经纬度），或把 `CITY_NAME` 也改成你喜欢的名称。
- **改频率**：修改 `.github/workflows/update_weather.yml` 里的 `cron` 表达式，例如每 2 小时：`0 */2 * * *`。
- **显示单位**：默认显示摄氏度和 km/h，如需华氏度或 mph，可在脚本里把 `UNITS` 变量调整为 `"fahrenheit"` 或 `"mph"`。

## 📝 许可
MIT License
