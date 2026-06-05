# OpenWeatherMap API — Postman Assignment

Exploring the OpenWeatherMap REST API using Postman. Three requests were made, inspected, and documented for Lagos, Nigeria.

---

## Files

| File | Description |
|------|-------------|
| `OpenWeatherMap API.postman_collection.json` | Importable Postman collection with all 3 requests |
| `Basic Lagos weather test.png` | Screenshot — current weather request & response |
| `5 Days forecast for Lagos.png` | Screenshot — 5-day forecast request & response |
| `Geocoding.png` | Screenshot — geocoding request & response |

---

## Requests & Findings

### 1. Current Weather
**`GET /data/2.5/weather?q=Lagos&units=metric`**

- Status: `200 OK` — 155ms
- Temp: 25.76°C, feels like 26.64°C
- Conditions: Broken clouds (71%), humidity 86%, wind 2.62 m/s
- Observation: Without `units=metric`, temperature returns in Kelvin (298.91K) — not human-readable.

### 2. 5-Day Forecast
**`GET /data/2.5/forecast?q=Lagos&units=metric&cnt=5`**

- Status: `200 OK`
- Returns a `list` array of 3-hour forecast windows
- `cnt=5` limits results to 5 entries, reducing response payload
- Each entry includes `dt_txt`, `temp`, and `weather.description`

### 3. Geocoding
**`GET /geo/1.0/direct?q=Lagos&limit=3`**

- Status: `200 OK`
- Returns `lat: 6.4541`, `lon: 3.3947`, `country: NG`
- Multiple cities named "Lagos" can appear — `country` field is needed to pick the right one

---

## Status Codes Observed

| Code | Meaning | Cause |
|------|---------|-------|
| `200` | OK | All successful requests |
| `401` | Unauthorized | New API key used before 2-hour activation window |

---

## Setup

1. Import `OpenWeatherMap API.postman_collection.json` into Postman
2. Add a collection variable `apiKey` = your key from [openweathermap.org](https://openweathermap.org)
3. Send any request