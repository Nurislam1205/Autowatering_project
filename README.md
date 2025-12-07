Below is a clean, technical, **GitHub-style explanation in English**, without embellishment, just facts. Suitable for README or code comments.

---

# Auto Watering System — Code Explanation

This Arduino program implements an automatic soil-moisture-based watering system using:

* **Soil moisture sensor (analog)**
* **Relay module** controlling a water pump
* **I2C LCD (16×2)** for on-device status display

The system reads soil moisture, converts this value into a percentage, displays it on an LCD, and activates a pump when the soil becomes too dry.

---

## 1. Hardware Components

* **Arduino (any compatible model)**
* **Soil moisture sensor (analog output)**
* **Relay module** for pump control
* **I2C 16×2 LCD** (address `0x27`)
* **Water pump**

Pins:

| Component | Arduino Pin |
| --------- | ----------- |
| Sensor    | A0          |
| Relay     | 7           |

---

## 2. Calibration Values

Two calibration points are used:

* `dryValue = 320` → sensor reading in dry soil
* `wetValue = 620` → sensor reading in wet soil

The raw sensor output is mapped to a **0–100% moisture scale** with:

```cpp
int humidityPercent = map(sensorValue, dryValue, wetValue, 0, 100);
```

Then constrained:

```cpp
humidityPercent = constrain(humidityPercent, 0, 100);
```

---

## 3. Watering Logic

The system waters the soil only when:

```
humidityPercent < humidityThreshold
```

`humidityThreshold` is set to **30%**.

Once watering begins:

* The relay is switched **ON** (LOW signal).
* Watering continues for a fixed duration (`wateringDuration = 5000 ms`).
* After timeout, the relay switches **OFF**.

A state variable `isWatering` prevents repeated triggering during watering.

---

### Watering cycle summary

1. Soil moisture < threshold → pump starts.
2. Pump runs for **5 seconds**.
3. Pump stops automatically.
4. System continues monitoring every second.

---

## 4. LCD Output

The LCD shows:

**Line 1:**

```
Moisture: XX%
```

**Line 2:** Depending on state:

* `"WATERING..."` while pump is active
* `"TOO DRY! PUMP ON"` when starting watering
* `"Watering Done!"` after completing a cycle
* `"Soil OK :)"` when moisture is above threshold

---

## 5. Serial Output

For debugging:

* Raw sensor value
* Calculated humidity %
* Current status: OK, TOO DRY, WATERING, PUMP ON/OFF

Example:

```
Sensor: 410 | Humidity: 45% | Status: OK
```

---

## 6. Functions Overview

### `startWatering()`

Activates the relay (LOW level) and prints `PUMP: ON`.

### `stopWatering()`

Deactivates the relay (HIGH level) and prints `PUMP: OFF`.

### `loop()`

Main logic:

* Read sensor
* Compute moisture %
* Update LCD
* Decide whether to start/stop watering

---

