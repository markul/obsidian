# Electricity Consumption

---

## ⚡ Baseline

- Total idle load with only the fridge, router, and Raspberry Pi 4 Model B Rev 1.2 running: `0.84 A`

## 📏 Measurements

### 2026-04-02

- Toilet light: `0.91 A`
- Bathroom light: `0.94 A`
- Bedroom light: `1.02 A` to `1.07 A`
- Living room lights: `1.16 A`
- Hall lights: `1.04 A`
- Kitchen lights: `1.27 A`
- Kitchen all lights max: `1.45 A`
- Kitchen main off: `1.11 A`
- Proxmox idle: `1.35 A`
- Proxmox + OptiPlex: `1.35 A` to `1.41 A`

## 🔌 Consumers

- All values below are estimated incremental power draw above the `0.84 A` baseline
- Assumption used: `230 V` mains
- Formula used: `consumer power = (measured current - baseline current) * 230`

Perfect — now we’ll **update your full consumer table** with **real monthly cost in RUB and USD** based on:

👉 **2.97 RUB/kWh (~$0.033/kWh)**

---

### ⚡  Consumer Table (Power + Cost)

### 🧊 Always-on baseline

| Consumer             | Power  | kWh/month | Cost (RUB)   | Cost ($) |
| -------------------- | ------ | --------- | ------------ | -------- |
| Fridge + Router + Pi | ~200 W | 144 kWh   | **~427 RUB** | ~$4.7    |

---

### 🖥️ Servers

| Consumer       | Power  | kWh/month | Cost (RUB)   | Cost ($) |
| -------------- | ------ | --------- | ------------ | -------- |
| Proxmox (Xeon) | ~120 W | 86 kWh    | **~256 RUB** | ~$2.8    |
| OptiPlex       | ~14 W  | 10 kWh    | **~30 RUB**  | ~$0.3    |

---

### 📺 Entertainment

| Consumer       | Measured             | Expected      | Notes             |
| -------------- | -------------------- | ------------- | ----------------- |
| LG G5 OLED     | ~100–140 W (implied) | **80–200 W**  | Content dependent |
| Sony HT-S700RF | ~20–40 W (implied)   | **20–50 W**   | Volume dependent  |
| **Combined**   | **~150–160 W**       | **120–200 W** | Matches perfectly |



| Consumer          | Usage          | kWh/month | Cost (RUB)  | Cost ($) |
| ----------------- | -------------- | --------- | ----------- | -------- |
| TV + Soundbar     | ~150 W, 4h/day | 18 kWh    | **~53 RUB** | ~$0.6    |
| PS5 Pro           | ~200 W, 2h/day | 12 kWh    | **~36 RUB** | ~$0.4    |
| MacBook + Monitor | ~160 W, 6h/day | 29 kWh    | **~86 RUB** | ~$1.0    |

---

### 💡 Lighting (estimated usage)

| Area                  | Usage          | kWh/month | Cost (RUB)  | Cost ($) |
| --------------------- | -------------- | --------- | ----------- | -------- |
| Kitchen lights (avg)  | ~100 W, 3h/day | 9 kWh     | **~27 RUB** | ~$0.3    |
| Other lights combined | ~100 W, 3h/day | 9 kWh     | **~27 RUB** | ~$0.3    |

---

### 📊 Total estimated monthly cost

| Category           | Cost     |
| ------------------ | -------- |
| Always-on baseline | ~427 RUB |
| Servers            | ~286 RUB |
| Entertainment      | ~175 RUB |
| Lighting           | ~54 RUB  |

👉 **Total ≈ 940 RUB/month (~$10.5)**

---
