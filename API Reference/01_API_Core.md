# 01. Samsung Health Sensor SDK Core API

## 1. 목적

이 문서는 Galaxy Watch 센서 데이터를 수집하기 위해 사용하는 **Samsung Health Sensor SDK Core API**를 정리한다.

센서별 API 이전에 반드시 이해해야 하는 공통 클래스는 다음과 같다.

- `HealthTrackingService`
- `ConnectionListener`
- `HealthTracker`
- `HealthTrackerCapability`
- `HealthTrackerType`
- `HealthTracker.TrackerEventListener`
- `DataPoint`
- `ValueKey`
- `TrackerUserProfile`

---

## 2. Package

```kotlin
com.samsung.android.service.health.tracking
com.samsung.android.service.health.tracking.data
```

| Package | 역할 |
|---|---|
| `com.samsung.android.service.health.tracking` | Health Platform 연결, tracker 생성, tracker capability 확인 |
| `com.samsung.android.service.health.tracking.data` | 센서 데이터 수신, `DataPoint`, `ValueKey`, `TrackerUserProfile` 관리 |

---

## 3. 전체 호출 흐름

```text
Watch App
↓
HealthTrackingService 생성
↓
connectService()
↓
ConnectionListener.onConnectionSuccess()
↓
getTrackingCapability()
↓
getSupportHealthTrackerTypes()
↓
getHealthTracker(HealthTrackerType)
↓
HealthTracker.setEventListener()
↓
TrackerEventListener.onDataReceived()
↓
DataPoint.getTimestamp()
↓
DataPoint.getValue(ValueKey)
↓
DB / CSV / Phone App 전송
↓
HealthTracker.unsetEventListener()
↓
disconnectService()
```

---

## 4. HealthTrackingService

### 4.1 설명

`HealthTrackingService`는 Samsung Health Tracking Service와 연결하는 핵심 클래스이다.

이 클래스에서 다음 작업을 수행한다.

- Health Platform 연결
- 지원 가능한 tracker 확인
- 센서별 `HealthTracker` 생성
- Service 연결 해제

---

### 4.2 Import

```kotlin
import com.samsung.android.service.health.tracking.HealthTrackingService
```

---

### 4.3 생성자

```kotlin
HealthTrackingService(
    connectionListener: ConnectionListener,
    context: Context
)
```

| Parameter | Type | 의미 |
|---|---|---|
| `connectionListener` | `ConnectionListener` | Health Tracking Service 연결 상태 callback |
| `context` | `Context` | Watch App context |

---

### 4.4 주요 함수

| Function | Return | 설명 |
|---|---|---|
| `connectService()` | `void` | Samsung Health Tracking Service 연결 |
| `disconnectService()` | `void` | Service 연결 해제 |
| `getTrackingCapability()` | `HealthTrackerCapability` | 현재 기기에서 지원하는 tracker 목록 확인 |
| `getHealthTracker(HealthTrackerType)` | `HealthTracker` | 일반 tracker 생성 |
| `getHealthTracker(HealthTrackerType, Set<PpgType>)` | `HealthTracker` | PPG 채널 지정 tracker 생성 |
| `getHealthTracker(HealthTrackerType, TrackerUserProfile)` | `HealthTracker` | BIA/MF-BIA 등 사용자 정보가 필요한 tracker 생성 |
| `getHealthTracker(HealthTrackerType, TrackerUserProfile, ExerciseType)` | `HealthTracker` | Sweat Loss 등 운동 타입이 필요한 tracker 생성 |

---

### 4.5 사용 예시

```kotlin
private lateinit var healthTrackingService: HealthTrackingService

private fun connectHealthTrackingService() {
    healthTrackingService = HealthTrackingService(
        connectionListener,
        this
    )

    healthTrackingService.connectService()
}
```

---

## 5. ConnectionListener

### 5.1 설명

`ConnectionListener`는 Health Tracking Service 연결 결과를 받는 interface이다.

`HealthTrackingService` 생성자에 전달된다.

---

### 5.2 Import

```kotlin
import com.samsung.android.service.health.tracking.ConnectionListener
```

---

### 5.3 Callback 함수

| Function | 설명 |
|---|---|
| `onConnectionSuccess()` | Service 연결 성공 시 호출 |
| `onConnectionEnded()` | Service 연결 종료 시 호출 |
| `onConnectionFailed(e)` | Service 연결 실패 시 호출 |

---

### 5.4 사용 예시

```kotlin
private val connectionListener = object : ConnectionListener {

    override fun onConnectionSuccess() {
        isServiceConnected = true
        checkSupportedTrackers()
    }

    override fun onConnectionEnded() {
        isServiceConnected = false
    }

    override fun onConnectionFailed(e: HealthTrackerException) {
        isServiceConnected = false

        if (e.hasResolution()) {
            e.resolve(this@MainActivity)
        }
    }
}
```

---

## 6. HealthTrackerCapability

### 6.1 설명

`HealthTrackerCapability`는 현재 Galaxy Watch에서 지원하는 tracker type 목록과 SDK version을 확인하는 클래스이다.

모든 Watch가 모든 센서를 지원하지 않으므로, 센서 사용 전 반드시 capability를 확인해야 한다.

---

### 6.2 Import

```kotlin
import com.samsung.android.service.health.tracking.HealthTrackerCapability
```

---

### 6.3 주요 함수

| Function | Return | 설명 |
|---|---|---|
| `getSupportHealthTrackerTypes()` | `List<HealthTrackerType>` | 현재 기기에서 지원하는 tracker type 목록 |
| `getVersion()` | `String` | SDK version 확인 |

---

### 6.4 사용 예시

```kotlin
private fun checkSupportedTrackers() {
    val capability = healthTrackingService.trackingCapability
    val supportedTypes = capability.supportHealthTrackerTypes

    supportedTypes.forEach { trackerType ->
        Log.d("SupportedTracker", trackerType.name)
    }
}
```

또는 Java-style API 기준:

```kotlin
private fun checkSupportedTrackers() {
    val capability = healthTrackingService.getTrackingCapability()
    val supportedTypes = capability.getSupportHealthTrackerTypes()

    supportedTypes.forEach { trackerType ->
        Log.d("SupportedTracker", trackerType.name)
    }
}
```

---

## 7. HealthTracker

### 7.1 설명

`HealthTracker`는 실제 센서 측정을 수행하는 객체이다.

예를 들어 다음 tracker들은 모두 `HealthTracker` 타입이다.

```kotlin
heartRateTracker
ppgTracker
accelerometerTracker
ecgTracker
spo2Tracker
biaTracker
```

---

### 7.2 Import

```kotlin
import com.samsung.android.service.health.tracking.HealthTracker
```

---

### 7.3 주요 함수

| Function | 설명 |
|---|---|
| `setEventListener(listener)` | 센서 데이터 수신 시작 |
| `unsetEventListener()` | 센서 데이터 수신 중지 |
| `flush()` | tracker 내부 버퍼 flush |

---

### 7.4 사용 예시

```kotlin
private var heartRateTracker: HealthTracker? = null

private fun createHeartRateTracker() {
    heartRateTracker = healthTrackingService.getHealthTracker(
        HealthTrackerType.HEART_RATE_CONTINUOUS
    )
}

private fun startHeartRateTracking() {
    heartRateTracker?.setEventListener(heartRateListener)
}

private fun stopHeartRateTracking() {
    heartRateTracker?.unsetEventListener()
}
```

---

## 8. HealthTrackerType

### 8.1 설명

`HealthTrackerType`은 어떤 센서를 측정할지 지정하는 enum이다.

`getHealthTracker()` 함수에 전달된다.

---

### 8.2 Import

```kotlin
import com.samsung.android.service.health.tracking.data.HealthTrackerType
```

---

### 8.3 사용 예시

```kotlin
val tracker = healthTrackingService.getHealthTracker(
    HealthTrackerType.HEART_RATE_CONTINUOUS
)
```

---

### 8.4 주요 TrackerType

```kotlin
HealthTrackerType.ACCELEROMETER_CONTINUOUS
HealthTrackerType.HEART_RATE_CONTINUOUS
HealthTrackerType.PPG_CONTINUOUS
HealthTrackerType.PPG_ON_DEMAND
HealthTrackerType.ECG_ON_DEMAND
HealthTrackerType.SPO2_ON_DEMAND
HealthTrackerType.SKIN_TEMPERATURE_CONTINUOUS
HealthTrackerType.SKIN_TEMPERATURE_ON_DEMAND
HealthTrackerType.BIA_ON_DEMAND
HealthTrackerType.MF_BIA_ON_DEMAND
HealthTrackerType.EDA_CONTINUOUS
HealthTrackerType.SWEAT_LOSS
```

---

## 9. HealthTracker.TrackerEventListener

### 9.1 설명

`TrackerEventListener`는 센서 데이터가 들어올 때 호출되는 callback interface이다.

센서값은 `onDataReceived()`의 `dataPoints`로 들어온다.

---

### 9.2 주요 Callback

| Callback | 설명 |
|---|---|
| `onDataReceived(dataPoints)` | 센서 데이터 수신 |
| `onFlushCompleted()` | flush 완료 |
| `onError(error)` | tracker error 발생 |

---

### 9.3 사용 예시

```kotlin
private val heartRateListener = object : HealthTracker.TrackerEventListener {

    override fun onDataReceived(dataPoints: List<DataPoint>) {
        dataPoints.forEach { dataPoint ->
            val timestamp = dataPoint.timestamp
        }
    }

    override fun onFlushCompleted() {
        Log.d("HeartRate", "Flush completed")
    }

    override fun onError(error: HealthTracker.TrackerError) {
        Log.e("HeartRate", "Tracker error: $error")
    }
}
```

---

## 10. DataPoint

### 10.1 설명

`DataPoint`는 센서 측정값 1개 단위이다.

공식적으로 `DataPoint`는 `ValueKey`와 value의 map이며 timestamp를 포함한다.

즉, 실제 센서값은 다음 방식으로 꺼낸다.

```kotlin
dataPoint.getValue(ValueKey...)
```

---

### 10.2 Import

```kotlin
import com.samsung.android.service.health.tracking.data.DataPoint
```

---

### 10.3 주요 함수

| Function | Return | 설명 |
|---|---|---|
| `getTimestamp()` | `Long` | 측정 timestamp 반환 |
| `getValue(ValueKey<T>)` | `T` | 해당 key의 센서값 반환 |

Kotlin property 형태:

```kotlin
dataPoint.timestamp
```

---

### 10.4 사용 예시

```kotlin
override fun onDataReceived(dataPoints: List<DataPoint>) {
    dataPoints.forEach { dataPoint ->

        val timestamp = dataPoint.timestamp

        val heartRate = dataPoint.getValue(
            ValueKey.HeartRateSet.HEART_RATE
        )
    }
}
```

---

## 11. ValueKey

### 11.1 설명

`ValueKey`는 `DataPoint`에서 어떤 값을 가져올지 지정하는 key class이다.

예를 들어 Heart Rate tracker에서 심박수를 가져올 때는 다음 key를 사용한다.

```kotlin
ValueKey.HeartRateSet.HEART_RATE
```

PPG Green 값을 가져올 때는 다음 key를 사용한다.

```kotlin
ValueKey.PpgSet.PPG_GREEN
```

---

### 11.2 Import

```kotlin
import com.samsung.android.service.health.tracking.data.ValueKey
```

---

### 11.3 사용 구조

```text
HealthTrackerType
↓
DataPoint
↓
ValueKey
↓
Value
```

예:

```kotlin
HealthTrackerType.HEART_RATE_CONTINUOUS
↓
DataPoint
↓
ValueKey.HeartRateSet.HEART_RATE
↓
heartRate value
```

---

### 11.4 대표 ValueKey Set

| ValueKey Set | 관련 TrackerType | 설명 |
|---|---|---|
| `ValueKey.HeartRateSet` | `HEART_RATE_CONTINUOUS` | Heart Rate, IBI |
| `ValueKey.PpgSet` | `PPG_CONTINUOUS`, `PPG_ON_DEMAND` | PPG Green/Red/IR |
| `ValueKey.AccelerometerSet` | `ACCELEROMETER_CONTINUOUS` | Accelerometer X/Y/Z |
| `ValueKey.EcgSet` | `ECG_ON_DEMAND` | ECG |
| `ValueKey.SpO2Set` | `SPO2_ON_DEMAND` | SpO₂ |
| `ValueKey.SkinTemperatureSet` | `SKIN_TEMPERATURE_CONTINUOUS`, `SKIN_TEMPERATURE_ON_DEMAND` | Skin temperature |
| `ValueKey.BiaSet` | `BIA_ON_DEMAND` | Body composition |
| `ValueKey.MfBiaSet` | `MF_BIA_ON_DEMAND` | Multi-frequency BIA |
| `ValueKey.EdaSet` | `EDA_CONTINUOUS` | Electrodermal activity |
| `ValueKey.SweatLossSet` | `SWEAT_LOSS` | Sweat loss |

---

## 12. PpgType

### 12.1 설명

`PpgType`은 PPG tracker에서 사용할 LED 채널을 지정한다.

PPG는 Green, Red, IR 채널을 선택할 수 있다.

---

### 12.2 Import

```kotlin
import com.samsung.android.service.health.tracking.data.PpgType
```

---

### 12.3 주요 값

```kotlin
PpgType.GREEN
PpgType.RED
PpgType.IR
```

---

### 12.4 사용 예시

```kotlin
val ppgTypes = setOf(
    PpgType.GREEN,
    PpgType.RED,
    PpgType.IR
)

val ppgTracker = healthTrackingService.getHealthTracker(
    HealthTrackerType.PPG_CONTINUOUS,
    ppgTypes
)
```

---

## 13. TrackerUserProfile

### 13.1 설명

`TrackerUserProfile`은 사용자의 신체 정보를 담는 class이다.

BIA, MF-BIA, Sweat Loss처럼 사용자 신체 정보가 필요한 tracker에서 사용한다.

공식 문서 기준 포함 정보:

- height
- weight
- gender
- age

---

### 13.2 Import

```kotlin
import com.samsung.android.service.health.tracking.data.TrackerUserProfile
```

---

### 13.3 Builder

```kotlin
TrackerUserProfile.Builder()
```

---

### 13.4 사용 예시

```kotlin
val userProfile = TrackerUserProfile.Builder()
    .setHeight(height)
    .setWeight(weight)
    .setGender(gender)
    .setAge(age)
    .build()
```

---

### 13.5 BIA Tracker 생성 예시

```kotlin
val biaTracker = healthTrackingService.getHealthTracker(
    HealthTrackerType.BIA_ON_DEMAND,
    userProfile
)
```

---

### 13.6 Sweat Loss Tracker 생성 예시

```kotlin
val sweatLossTracker = healthTrackingService.getHealthTracker(
    HealthTrackerType.SWEAT_LOSS,
    userProfile,
    ExerciseType.RUNNING
)
```

---

## 14. HealthTrackerException

### 14.1 설명

`HealthTrackerException`은 Health Tracking Service 연결 실패 또는 SDK 관련 오류를 나타낸다.

`ConnectionListener.onConnectionFailed()`에서 주로 사용한다.

---

### 14.2 Import

```kotlin
import com.samsung.android.service.health.tracking.HealthTrackerException
```

---

### 14.3 주요 함수

| Function | 설명 |
|---|---|
| `hasResolution()` | 사용자가 해결 가능한 오류인지 확인 |
| `resolve(activity)` | 오류 해결 화면 실행 |
| `getErrorCode()` | 오류 코드 반환 |

---

### 14.4 사용 예시

```kotlin
override fun onConnectionFailed(e: HealthTrackerException) {
    if (e.hasResolution()) {
        e.resolve(this@MainActivity)
    } else {
        Log.e("HealthTracking", "Connection failed: ${e.errorCode}")
    }
}
```

---

## 15. Core API 최소 구현 예시

```kotlin
class MainActivity : Activity() {

    private lateinit var healthTrackingService: HealthTrackingService
    private var isServiceConnected = false

    private val connectionListener = object : ConnectionListener {

        override fun onConnectionSuccess() {
            isServiceConnected = true
            checkSupportedTrackers()
        }

        override fun onConnectionEnded() {
            isServiceConnected = false
        }

        override fun onConnectionFailed(e: HealthTrackerException) {
            isServiceConnected = false

            if (e.hasResolution()) {
                e.resolve(this@MainActivity)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        healthTrackingService = HealthTrackingService(
            connectionListener,
            this
        )

        healthTrackingService.connectService()
    }

    private fun checkSupportedTrackers() {
        val capability = healthTrackingService.trackingCapability
        val supportedTypes = capability.supportHealthTrackerTypes

        supportedTypes.forEach { type ->
            Log.d("SupportedTracker", type.name)
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        healthTrackingService.disconnectService()
    }
}
```

---

## 16. 프로젝트 파일 배치 권장

```text
watch-app/
└── sensor/
    ├── HealthTrackingManager.kt
    ├── SensorTrackerFactory.kt
    ├── HeartRateTrackerManager.kt
    ├── PpgTrackerManager.kt
    ├── AccelerometerTrackerManager.kt
    ├── EcgTrackerManager.kt
    ├── Spo2TrackerManager.kt
    ├── SkinTemperatureTrackerManager.kt
    ├── BiaTrackerManager.kt
    ├── MfBiaTrackerManager.kt
    ├── EdaTrackerManager.kt
    └── SweatLossTrackerManager.kt
```

---

## 17. Core API별 권장 파일 위치

| API | 권장 파일 |
|---|---|
| `HealthTrackingService` | `HealthTrackingManager.kt` |
| `ConnectionListener` | `HealthTrackingManager.kt` |
| `HealthTrackerCapability` | `HealthTrackingManager.kt` |
| `HealthTracker` | 각 센서별 `TrackerManager.kt` |
| `HealthTrackerType` | 각 센서별 `TrackerManager.kt` |
| `TrackerEventListener` | 각 센서별 `TrackerManager.kt` |
| `DataPoint` | 각 센서별 `TrackerManager.kt` |
| `ValueKey` | 각 센서별 `TrackerManager.kt` |
| `TrackerUserProfile` | `UserProfileManager.kt` 또는 `BiaTrackerManager.kt` |
| `HealthTrackerException` | `HealthTrackingManager.kt` |

---

## 18. Core API 요약

| API | 핵심 역할 |
|---|---|
| `HealthTrackingService` | Health Platform 연결 및 tracker 생성 |
| `ConnectionListener` | Service 연결 상태 callback |
| `HealthTrackerCapability` | 지원 센서 확인 |
| `HealthTracker` | 센서별 측정 객체 |
| `HealthTrackerType` | 측정할 센서 type |
| `TrackerEventListener` | 센서 데이터 수신 callback |
| `DataPoint` | timestamp와 sensor value를 가진 데이터 단위 |
| `ValueKey` | DataPoint에서 value를 꺼내는 key |
| `PpgType` | PPG Green/Red/IR 채널 선택 |
| `TrackerUserProfile` | BIA/MF-BIA/Sweat Loss용 사용자 정보 |
| `HealthTrackerException` | 연결/SDK 오류 처리 |

---

## 19. 공식 문서

- Samsung Health Sensor SDK Overview  
  https://developer.samsung.com/health/sensor/api-reference/overview-summary.html

- HealthTrackingService  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/HealthTrackingService.html

- HealthTrackerType  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/HealthTrackerType.html

- Data Package Summary  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/package-summary.html

- ConnectionListener  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/ConnectionListener.html

- TrackerUserProfile.Builder  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/TrackerUserProfile.Builder.html
