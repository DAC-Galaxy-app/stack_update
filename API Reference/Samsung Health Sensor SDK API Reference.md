# Samsung Health Sensor SDK API Reference

> Samsung Health Sensor SDK + Samsung Health Research Stack API Reference
>
> This document summarizes every API used in this project for collecting physiological signals from Galaxy Watch and managing clinical research.

---

# Table of Contents

- Overview
- SDK Architecture
- Samsung Health Sensor SDK
    - HealthTrackingService
    - HealthTracker
    - HealthTrackerType
    - TrackerEventListener
    - DataPoint
    - ValueKey
    - TrackerUserProfile
- Continuous Tracker APIs
- On-demand Tracker APIs
- Samsung Health Research Stack APIs

---

# 1. Overview

본 프로젝트는 두 개의 SDK를 사용한다.

| SDK | 역할 |
|------|------|
| Samsung Health Sensor SDK | Galaxy Watch 센서 데이터 수집 |
| Samsung Health Research Stack | 연구 플랫폼(Participant / Consent / Survey / Backend / Portal) |

Sensor SDK는 실제 센서(raw signal)를 읽는다.

Research Stack은 연구 관리 시스템이다.

---

# 2. SDK Architecture

```
Galaxy Watch
│
├── Samsung Health Sensor SDK
│       │
│       ├── Heart Rate
│       ├── IBI
│       ├── PPG
│       ├── ECG
│       ├── SpO₂
│       ├── Accelerometer
│       ├── Skin Temperature
│       ├── BIA
│       ├── MF-BIA
│       ├── EDA
│       └── Sweat Loss
│
▼
Phone App
│
├── Samsung Health Research Stack
│       ├── Participant
│       ├── Consent
│       ├── Survey
│       ├── Task
│       ├── Upload
│       └── Session
│
▼
Backend
│
▼
Web Portal
```

---

# 3. Samsung Health Sensor SDK

Package

```kotlin
com.samsung.android.service.health.tracking

com.samsung.android.service.health.tracking.data
```

---

# 4. HealthTrackingService

## Description

Sensor SDK의 가장 상위 클래스이다.

Samsung Health Tracking Service와 연결하고
각 Sensor Tracker를 생성한다.

---

## Constructor

```kotlin
HealthTrackingService(
    ConnectionListener listener,
    Context context
)
```

---

## Member Functions

| Function | Description |
|-----------|-------------|
| connectService() | Samsung Health Service 연결 |
| disconnectService() | 연결 종료 |
| getTrackingCapability() | 지원 센서 확인 |
| getHealthTracker(type) | Tracker 생성 |
| getHealthTracker(type, ppgTypes) | PPG Tracker 생성 |
| getHealthTracker(type, userProfile) | BIA Tracker 생성 |
| getHealthTracker(type, userProfile, exerciseType) | Sweat Loss Tracker 생성 |

---

## Example

```kotlin
healthTrackingService =
    HealthTrackingService(connectionListener,this)

healthTrackingService.connectService()
```

---

# 5. ConnectionListener

Service 연결 상태를 알려준다.

```kotlin
private val connectionListener =
object : ConnectionListener {

    override fun onConnectionSuccess()

    override fun onConnectionEnded()

    override fun onConnectionFailed(
        e: HealthTrackerException
    )
}
```

---

## onConnectionSuccess()

호출 시점

```
HealthTrackingService.connectService()

↓

Service Connected

↓

onConnectionSuccess()
```

주요 작업

```kotlin
isServiceConnected = true

checkSupportedTracker()

createTrackers()
```

---

## onConnectionEnded()

Service 종료 시 호출

```kotlin
isServiceConnected = false
```

---

## onConnectionFailed()

Service 연결 실패

```kotlin
if(e.hasResolution()){

    e.resolve(this)

}
```

---

# 6. HealthTracker

각 Sensor 하나를 의미한다.

예를 들어

```
Heart Rate Tracker

PPG Tracker

ECG Tracker

SpO₂ Tracker

Accelerometer Tracker
```

모두 HealthTracker 객체이다.

---

## Functions

| Function | Description |
|-----------|-------------|
| setEventListener() | 측정 시작 |
| unsetEventListener() | 측정 종료 |
| flush() | 버퍼 비우기 |

---

## Example

```kotlin
val heartRateTracker =

healthTrackingService.getHealthTracker(

HealthTrackerType.HEART_RATE_CONTINUOUS

)

heartRateTracker.setEventListener(listener)
```

---

# 7. TrackerEventListener

센서 데이터가 들어오는 Callback

```kotlin
override fun onDataReceived(

dataPoints: List<DataPoint>

)

override fun onFlushCompleted()

override fun onError(
    error: TrackerError
)
```

---

## onDataReceived()

가장 중요한 Callback

모든 Sensor Data는 여기로 들어온다.

```
Sensor

↓

Tracker

↓

onDataReceived()

↓

DataPoint

↓

ValueKey

↓

Database
```

---

# 8. DataPoint

센서 데이터 한 개를 의미한다.

예

```
Timestamp

Heart Rate

PPG Green

PPG Red

PPG IR

Status
```

모두 DataPoint 안에 들어있다.

---

## Functions

```kotlin
dataPoint.timestamp

dataPoint.getTimestamp()

dataPoint.getValue(ValueKey)
```

---

## Example

```kotlin
val timestamp =

dataPoint.timestamp

val heartRate =

dataPoint.getValue(

ValueKey.HeartRateSet.HEART_RATE

)
```

---

# 9. ValueKey

ValueKey는

DataPoint 안에서

어떤 데이터를 가져올지 지정하는 Key이다.

예

```kotlin
ValueKey.HeartRateSet.HEART_RATE
```

↓

Heart Rate 반환

```kotlin
ValueKey.PpgSet.PPG_GREEN
```

↓

Green PPG 반환

```kotlin
ValueKey.EcgSet.ECG_MV
```

↓

ECG Signal 반환

---

# 10. TrackerUserProfile

일부 Sensor는 사용자 정보가 필요하다.

대표적으로

```
BIA

MF-BIA

Sweat Loss
```

---

## Builder

```kotlin
TrackerUserProfile.Builder()

.setHeight()

.setWeight()

.setAge()

.setGender()

.build()
```

---

## Example

```kotlin
val profile =

TrackerUserProfile.Builder()

.setHeight(175f)

.setWeight(72f)

.setAge(28)

.setGender(Gender.MALE)

.build()
```

---

# 11. HealthTrackerType

Tracker 종류를 정의하는 Enum

```
ACCELEROMETER_CONTINUOUS

HEART_RATE_CONTINUOUS

PPG_CONTINUOUS

PPG_ON_DEMAND

ECG_ON_DEMAND

SPO2_ON_DEMAND

SKIN_TEMPERATURE_CONTINUOUS

SKIN_TEMPERATURE_ON_DEMAND

EDA_CONTINUOUS

BIA_ON_DEMAND

MF_BIA_ON_DEMAND

SWEAT_LOSS
```

---

# Next

다음 문서에서는

- Heart Rate API
- PPG API
- ECG API
- SpO₂ API
- BIA API
- MF-BIA API
- Skin Temperature API
- EDA API
- Sweat Loss API

각 ValueKey 하나하나까지 전부 정리한다.
