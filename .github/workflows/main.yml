#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
从 Open-Meteo 拉取深圳天气，更新 README 中 <!-- WEATHER-START --> 与 <!-- WEATHER-END --> 之间的内容。
"""
import json
import re
from urllib.parse import urlencode
from urllib.request import urlopen, Request

# ===== 可按需修改 =====
CITY_NAME = "深圳"
LAT = 22.5431
LON = 114.0579
TIMEZONE = "Asia/Shanghai"  # 本地时间
# 温度单位：celsius / fahrenheit；风速单位：kmh / mph / ms / kn
TEMP_UNIT = "celsius"
WIND_UNIT = "kmh"
# =====================

API = "https://api.open-meteo.com/v1/forecast"

CURRENT_VARS = [
    "temperature_2m",
    "apparent_temperature",
    "relative_humidity_2m",
    "wind_speed_10m",
    "weather_code",
    "precipitation",
    "cloud_cover",
    "is_day",
]

DAILY_VARS = [
    "temperature_2m_max",
    "temperature_2m_min",
    "precipitation_sum",
    "uv_index_max",
    "weather_code",
]

WMO_MAP = {
    0: ("晴", "☀️"),
    1: ("多云转晴", "🌤️"),
    2: ("多云", "⛅️"),
    3: ("阴", "☁️"),
    45: ("有雾", "🌫️"),
    48: ("霜雾", "🌫️"),
    51: ("小毛毛雨", "🌦️"),
    53: ("中毛毛雨", "🌦️"),
    55: ("大毛毛雨", "🌧️"),
    56: ("小冻雨", "🧊🌧️"),
    57: ("大冻雨", "🧊🌧️"),
    61: ("小雨", "🌧️"),
    63: ("中雨", "🌧️"),
    65: ("大雨", "🌧️"),
    66: ("小冻雨", "🧊🌧️"),
    67: ("大冻雨", "🧊🌧️"),
    71: ("小雪", "🌨️"),
    73: ("中雪", "🌨️"),
    75: ("大雪", "❄️"),
    77: ("冰粒", "🌨️"),
    80: ("阵雨", "🌦️"),
    81: ("中阵雨", "🌧️"),
    82: ("暴阵雨", "⛈️"),
    85: ("小阵雪", "🌨️"),
    86: ("大阵雪", "❄️"),
    95: ("雷阵雨", "⛈️"),
    96: ("雷阵雨伴冰雹", "⛈️🧊"),
    99: ("强雷阵雨伴冰雹", "⛈️🧊"),
}

def code_to_text(code: int):
    return WMO_MAP.get(code, ("未知", "❔"))

def get(url: str):
    req = Request(url, headers={"User-Agent": "Mozilla/5.0 (GitHub Actions Weather Updater)"})
    with urlopen(req, timeout=30) as resp:
        return json.loads(resp.read().decode("utf-8"))

def build_api_url():
    params = {
        "latitude": LAT,
        "longitude": LON,
        "timezone": TIMEZONE,
        "current": ",".join(CURRENT_VARS),
        "daily": ",".join(DAILY_VARS),
        "temperature_unit": TEMP_UNIT,
        "wind_speed_unit": WIND_UNIT,
        "precipitation_unit": "mm",
        "forecast_days": 4,  # 今天 + 接下来 3 天
    }
    return f"{API}?{urlencode(params)}"

def render_md(data: dict) -> str:
    tz_abbr = data.get("timezone_abbreviation", "")
    cur = data["current"]
    daily = data["daily"]

    # 当前
    c_code = int(cur.get("weather_code", 0))
    c_text, c_emoji = code_to_text(c_code)
    c_temp = cur.get("temperature_2m")
    c_feel = cur.get("apparent_temperature")
    c_hum  = cur.get("relative_humidity_2m")
    c_wind = cur.get("wind_speed_10m")
    c_cloud= cur.get("cloud_cover")
    c_prec = cur.get("precipitation")
    c_time = cur.get("time")

    now_line = f"{c_emoji} **{CITY_NAME} 当前** · {c_text} ｜ {c_temp:.1f}°C（体感 {c_feel:.1f}°C）｜ 湿度 {c_hum}% ｜ 风速 {c_wind} {WIND_UNIT} ｜ 云量 {c_cloud}% ｜ 降水 {c_prec} mm"

    # 今日
    t_max = daily["temperature_2m_max"][0]
    t_min = daily["temperature_2m_min"][0]
    t_rain= daily.get("precipitation_sum", [0])[0]
    t_uv  = daily.get("uv_index_max", [None])[0]
    t_code= int(daily.get("weather_code", [c_code])[0])
    t_text, t_emoji = code_to_text(t_code)
    today_line = f"{t_emoji} **今日** · {t_text} ｜ {t_min:.1f}°C ~ {t_max:.1f}°C ｜ 降水量 {t_rain} mm" + (f" ｜ UV 指数 {t_uv}" if t_uv is not None else "")

    # 接下来的 3 天
    rows = []
    for i in range(1, min(4, len(daily["time"]))):
        d = daily["time"][i]
        code = int(daily.get("weather_code", [0]*len(daily["time"]))[i])
        tx, emoji = code_to_text(code)
        tmax = daily["temperature_2m_max"][i]
        tmin = daily["temperature_2m_min"][i]
        pr   = daily.get("precipitation_sum", [0]*len(daily["time"]))[i]
        rows.append(f"| {d} | {emoji} {tx} | {tmin:.1f}°C ~ {tmax:.1f}°C | {pr} mm |")

    table = "\n".join([
        "",
        "| 日期 | 天气 | 最低/最高 | 降水量 |",
        "|---|---|---|---|",
        *rows
    ])

    updated = f"数据更新：`{c_time} {tz_abbr}`"
    return "\n".join([updated, "", now_line, today_line, table])

def update_readme(md_block: str, path: str = "README.md"):
    with open(path, "r", encoding="utf-8") as f:
        content = f.read()

    start = "<!-- WEATHER-START -->"
    end = "<!-- WEATHER-END -->"
    pattern = re.compile(rf"{re.escape(start)}.*?{re.escape(end)}", re.S)

    new_block = f"{start}\n{md_block}\n{end}"
    new_content = pattern.sub(new_block, content)

    if new_content != content:
        with open(path, "w", encoding="utf-8", newline="\n") as f:
            f.write(new_content)
        print("README.md 已更新天气卡片。")
    else:
        print("内容未变化，无需更新。")

def main():
    url = build_api_url()
    data = get(url)
    md = render_md(data)
    update_readme(md)

if __name__ == "__main__":
    main()
