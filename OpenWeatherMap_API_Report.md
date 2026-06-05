# OpenWeatherMap API — Postman Practical Assignment
**Name:** Olumide D. Fakorede 
**Date:** June 2026  
**Collection Name:** OpenWeatherMap API 

---

## Part 2 — Reading and Interpreting the Response

### 1. HTTP Status Analysis

The request returned a status code of **200 OK**.

A 200 status means the request was received, understood, and processed successfully by the server. The API located the city, authenticated the API key, and returned the weather data as expected. No errors occurred on either the client or server side.

---

### 2. JSON Inspection — Three Key Fields

**Field 1: `main.temp`**  
Value returned: `298.91`  
This represents the current air temperature at the requested location. By default, OpenWeatherMap returns temperature in **Kelvin**, so 298.91K is approximately **25.76°C**. This is the primary data point for any weather-related decision or automation.

**Field 2: `weather[0].description`**  
Value returned: `"broken clouds"`  
This is a human-readable string describing the current sky conditions. It comes from the `weather` array — an array is used because in some cases multiple weather conditions can apply simultaneously. The `[0]` accesses the first (and most relevant) condition. This field is useful for display purposes or as a trigger condition in automations.

**Field 3: `wind.speed`**  
Value returned: `2.62`  
This represents the current wind speed in metres per second (m/s) at the location. Wind data is critical for logistics, outdoor operations, drone scheduling, and safety alerts. Combined with `wind.deg` (194° — south-southwest), it gives a full picture of wind conditions.

---

### 3. Temperature Units

By default, the API returns temperature in **Kelvin** — the SI unit for thermodynamic temperature. This is the API's baseline unit when no `units` parameter is specified.

To receive temperature in **Celsius**, add the `units=metric` query parameter to the request URL:

```
https://api.openweathermap.org/data/2.5/weather?q=Lagos&appid={{api_key}}&units=metric
```

To receive temperature in **Fahrenheit**, use `units=imperial` instead.

| Parameter      | Temperature Unit |
|----------------|-----------------|
| *(none)*       | Kelvin (K)      |
| `units=metric` | Celsius (°C)    |
| `units=imperial` | Fahrenheit (°F) |

---

## Part 3 — Exploring HTTP Methods & Error Handling

### 1. Misspelled City Test

**Request made:** `?q=Lagoss&appid={{api_key}}`

| Field | Value |
|-------|-------|
| Status Code | `200 OK` |
| Response body | Empty / no weather data returned |

Contrary to what might be expected, the API returned a **200 OK** status even for an unrecognised city name, with an empty response body rather than an error message. This is an example of **lenient API design** — the server technically fulfilled the request without crashing, but returned no useful data.

This is an important distinction for developers building automations: a 200 status alone cannot be trusted as confirmation that valid data was returned. Any code consuming this API must also check that the response body contains the expected fields before acting on it. A robust implementation would validate that `name`, `main.temp`, and `weather` fields exist in the response before proceeding.

---

### 2. Authentication Test — Removed API Key

**Request made:** `?q=Lagos` (no `appid` parameter)

| Field | Value |
|-------|-------|
| Status Code | `401 Unauthorized` |
| `message` in body | `"Invalid API key..."` |

A 401 status means the request was rejected because no valid credentials were provided. This confirms that the OpenWeatherMap API uses **key-based authentication** — every request must include a valid `appid` query parameter. Without it, the server refuses to return any data regardless of whether the city name is valid. This is a standard security control to prevent anonymous or abusive access to the API.

---

### 3. Forecast Comparison — `/data/2.5/forecast`

**Endpoint used:** `https://api.openweathermap.org/data/2.5/forecast?q=Lagos&appid={{api_key}}&units=metric`

**Structural differences between Current Weather and Forecast responses:**

| Aspect | Current Weather (`/weather`) | 5-Day Forecast (`/forecast`) |
|--------|------------------------------|------------------------------|
| Top-level structure | Single JSON object | Object containing a `list` array |
| Data points | 1 snapshot (right now) | Up to 40 entries (every 3 hours) |
| Time field | `dt` — single Unix timestamp | `dt` + `dt_txt` per list item |
| City info | Embedded directly in root | Nested inside a `city` object |
| `cnt` field | Not present | Number of forecast entries returned |

The most significant difference is that the forecast response wraps all data inside a `list` array, where each element represents a 3-hour forecast window. This means processing a forecast requires iterating over the array, whereas the current weather response can be read directly as a flat object.

---

## Part 4 — Automation Thinking Connection

### Reflection: Real-World Workflow for a Logistics Company

**Scenario: Automated Delivery Route Alerts**

A logistics company managing last-mile delivery drivers across Lagos could use the OpenWeatherMap API to build a weather-aware dispatch automation.

**Task being automated:**  
The system automatically checks weather conditions along active delivery routes at the start of each shift and flags routes that fall under unsafe weather thresholds — such as heavy rain, strong winds, or low visibility below 3,000m.

**Trigger condition:**  
The automation runs on a scheduled trigger every morning at 6:00 AM. It calls the `/forecast` endpoint for each active delivery zone. If any forecast entry within the next 6 hours returns `wind.speed` above 10 m/s, `visibility` below 3,000m, or a `weather.main` value of `"Thunderstorm"`, the trigger fires.

**Expected outcome:**  
Affected routes are automatically flagged in the dispatch dashboard, drivers assigned to those zones receive an SMS or push notification advising caution or a route change, and a supervisor is notified via email. This reduces accident risk, prevents weather-related delivery failures, and removes the manual task of a dispatcher checking weather forecasts for each zone individually every morning.

---

## Summary of Status Codes Encountered

| Code | Meaning | When Observed |
|------|---------|---------------|
| `200 OK` | Success | All valid requests — and misspelled city (returned empty body) |
| `401 Unauthorized` | Missing or invalid API key | Request sent without `appid` parameter |