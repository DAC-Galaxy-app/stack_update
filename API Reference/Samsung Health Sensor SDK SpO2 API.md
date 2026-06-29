# 04. Samsung Health Sensor SDK SpO2 API

## 1. 목적

이 문서는 Samsung Health Sensor SDK의 SpO2 API를 정리한다.

SpO2는 `continuous tracker`가 아니라 `on-demand tracker`이다.  
일반적인 장시간 연속 측정용이 아니라, 사용자가 Galaxy Watch를 올바르게 착용한 상태에서 짧은 시간 동안 산소포화도 측정을 수행하는 구조로 설계한다.

---

## 2. API 요약

| 항목 | 내용 |
|---|---|
| TrackerType | `HealthTrackerType.SPO2_ON_DEMAND` |
| ValueKey Set | `ValueKey.SpO2Set` |
| 측정 방식 | On-demand |
| 권장 파일 | `watch-app/sensor/Spo2TrackerManager.kt` |
| 주요 수집값 | SpO2, Heart Rate, Status |
| 일반 측정 시간 | 약 30초 |

---

## 3. Package / Import

```kotlin
import android.util.Log
import com.samsung.android.service.health.tracking.HealthTracker
import com.samsung.android.service.health.tracking.HealthTrackingService
import com.samsung.android.service.health.tracking.data.DataPoint
import com.samsung.android.service.health.tracking.data.HealthTrackerType
import com.samsung.android.service.health.tracking.data.ValueKey
```

---

## 4. TrackerType

```kotlin
HealthTrackerType.SPO2_ON_DEMAND
```

`SPO2_ON_DEMAND`는 Galaxy Watch에서 혈중 산소포화도 값을 측정하기 위한 `HealthTrackerType`이다.

```kotlin
val spo2Tracker = healthTrackingService.getHealthTracker(
    HealthTrackerType.SPO2_ON_DEMAND
)
```

---

## 5. ValueKey Set

```kotlin
ValueKey.SpO2Set
```

`ValueKey.SpO2Set`은 `SPO2_ON_DEMAND`에서 수신되는 `DataPoint` 값을 꺼내기 위한 key set이다.

공식 문서 기준 `ValueKey.SpO2Set`은 다음 `ValueKey`를 포함한다.

```kotlin
ValueKey.SpO2Set.SPO2
ValueKey.SpO2Set.HEART_RATE
ValueKey.SpO2Set.STATUS
```

---

## 6. ValueKey 목록

| ValueKey | Type | 의미 | 단위 / 범위 |
|---|---|---|---|
| `ValueKey.SpO2Set.SPO2` | `Integer` | Oxygen saturation value | `%` |
| `ValueKey.SpO2Set.HEART_RATE` | `Integer` | SpO2 측정 중 heart rate value | bpm |
| `ValueKey.SpO2Set.STATUS` | `Integer` | SpO2 measurement status | status code |

---

## 7. ValueKey 상세

### 7.1 `SPO2`

```kotlin
ValueKey.SpO2Set.SPO2
```

혈중 산소포화도 값이다.

```kotlin
val spo2 = dataPoint.getValue(
    ValueKey.SpO2Set.SPO2
)
```

| 항목 | 내용 |
|---|---|
| Type | `Integer` |
| 의미 | Oxygen saturation value |
| 단위 | `%` |
| 저장 변수명 | `spo2` |
| 저장 valueName | `spo2` |

---

### 7.2 `HEART_RATE`

```kotlin
ValueKey.SpO2Set.HEART_RATE
```

SpO2 측정 중 함께 제공되는 heart rate 값이다.

```kotlin
val heartRate = dataPoint.getValue(
    ValueKey.SpO2Set.HEART_RATE
)
```

| 항목 | 내용 |
|---|---|
| Type | `Integer` |
| 의미 | Heart rate value during SpO2 measurement |
| 단위 | bpm |
| 저장 변수명 | `spo2HeartRate` |
| 저장 valueName | `spo2_heart_rate` |

---

### 7.3 `STATUS`

```kotlin
ValueKey.SpO2Set.STATUS
```

SpO2 측정 상태를 나타내는 status flag이다.

```kotlin
val status = dataPoint.getValue(
    ValueKey.SpO2Set.STATUS
)
```

| 항목 | 내용 |
|---|---|
| Type | `Integer` |
| 의미 | SpO2 measurement status |
| 저장 변수명 | `spo2Status` |
| 저장 valueName | `spo2_status` |

---

## 8. Status Code

`ValueKey.SpO2Set.STATUS`는 SpO2 측정 상태를 나타낸다.

| Status | 의미 |
|---|---|
| `-6` | Time out |
| `-5` | Signal quality is low |
| `-4` | Device moved during measurement |
| `0` | Calculating SpO2 |
| `2` | SpO2 measurement was completed |

`STATUS == 2`이면 SpO2 측정이 완료된 상태이다.

---

## 9. 저장 변수명 권장

```kotlin
val timestamp: Long
val spo2: Int?
val spo2HeartRate: Int?
val spo2Status: Int?
```

---

## 10. 저장 valueName 권장

| valueName | 의미 |
|---|---|
| `spo2` | Oxygen saturation value |
| `spo2_heart_rate` | Heart rate during SpO2 measurement |
| `spo2_status` | SpO2 measurement status |

---

## 11. 권장 파일 위치

```text
watch-app/
└── sensor/
    └── Spo2TrackerManager.kt
```

---

## 12. SpO2 Tracker 생성

```kotlin
private var spo2Tracker: HealthTracker? = null

fun createTracker() {
    spo2Tracker = healthTrackingService.getHealthTracker(
        HealthTrackerType.SPO2_ON_DEMAND
    )
}
```

---

## 13. SpO2 측정 시작 / 종료

### 13.1 측정 시작

```kotlin
fun startMeasurement() {
    spo2Tracker?.setEventListener(spo2Listener)
}
```

### 13.2 측정 종료

```kotlin
fun stopMeasurement() {
    spo2Tracker?.unsetEventListener()
}
```

---

## 14. SpO2 Listener 예시

```kotlin
private val spo2Listener = object : HealthTracker.TrackerEventListener {

    override fun onDataReceived(dataPoints: List<DataPoint>) {
        dataPoints.forEach { dataPoint ->

            val timestamp = dataPoint.timestamp

            readSpo2(dataPoint, timestamp)
            readHeartRate(dataPoint, timestamp)
            readStatus(dataPoint, timestamp)
        }
    }

    override fun onFlushCompleted() {
        Log.d("Spo2Tracker", "Flush completed")
    }

    override fun onError(error: HealthTracker.TrackerError) {
        Log.e("Spo2Tracker", "Error: $error")
    }
}
```

---

## 15. Key별 읽기 함수 예시

SpO2 측정 중 `DataPoint`마다 포함된 key가 다를 수 있으므로, 안전하게 예외 처리하는 구조를 권장한다.

### 15.1 `SPO2`

```kotlin
private fun readSpo2(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val spo2 = dataPoint.getValue(
            ValueKey.SpO2Set.SPO2
        )

        onSampleReceived(
            "SPO2",
            timestamp,
            "spo2",
            spo2.toString(),
            "%"
        )
    } catch (e: Exception) {
        Log.d("Spo2Tracker", "SPO2 not available in this DataPoint")
    }
}
```

### 15.2 `HEART_RATE`

```kotlin
private fun readHeartRate(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val heartRate = dataPoint.getValue(
            ValueKey.SpO2Set.HEART_RATE
        )

        onSampleReceived(
            "SPO2",
            timestamp,
            "spo2_heart_rate",
            heartRate.toString(),
            "bpm"
        )
    } catch (e: Exception) {
        Log.d("Spo2Tracker", "HEART_RATE not available in this DataPoint")
    }
}
```

### 15.3 `STATUS`

```kotlin
private fun readStatus(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val status = dataPoint.getValue(
            ValueKey.SpO2Set.STATUS
        )

        onSampleReceived(
            "SPO2",
            timestamp,
            "spo2_status",
            status.toString(),
            null
        )

        if (status == 2) {
            stopMeasurement()
        }
    } catch (e: Exception) {
        Log.d("Spo2Tracker", "STATUS not available in this DataPoint")
    }
}
```

---

## 16. 전체 `Spo2TrackerManager.kt` 예시

```kotlin
class Spo2TrackerManager(
    private val healthTrackingService: HealthTrackingService,
    private val onSampleReceived: (
        sensorType: String,
        timestamp: Long,
        valueName: String,
        value: String,
        unit: String?
    ) -> Unit
) {

    private var spo2Tracker: HealthTracker? = null

    fun createTracker() {
        spo2Tracker = healthTrackingService.getHealthTracker(
            HealthTrackerType.SPO2_ON_DEMAND
        )
    }

    fun startMeasurement() {
        spo2Tracker?.setEventListener(spo2Listener)
    }

    fun stopMeasurement() {
        spo2Tracker?.unsetEventListener()
    }

    private val spo2Listener = object : HealthTracker.TrackerEventListener {

        override fun onDataReceived(dataPoints: List<DataPoint>) {
            dataPoints.forEach { dataPoint ->

                val timestamp = dataPoint.timestamp

                readSpo2(dataPoint, timestamp)
                readHeartRate(dataPoint, timestamp)
                readStatus(dataPoint, timestamp)
            }
        }

        override fun onFlushCompleted() {
            Log.d("Spo2Tracker", "Flush completed")
        }

        override fun onError(error: HealthTracker.TrackerError) {
            Log.e("Spo2Tracker", "Error: $error")
        }
    }

    private fun readSpo2(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val spo2 = dataPoint.getValue(
                ValueKey.SpO2Set.SPO2
            )

            onSampleReceived(
                "SPO2",
                timestamp,
                "spo2",
                spo2.toString(),
                "%"
            )
        } catch (e: Exception) {
            Log.d("Spo2Tracker", "SPO2 not available")
        }
    }

    private fun readHeartRate(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val heartRate = dataPoint.getValue(
                ValueKey.SpO2Set.HEART_RATE
            )

            onSampleReceived(
                "SPO2",
                timestamp,
                "spo2_heart_rate",
                heartRate.toString(),
                "bpm"
            )
        } catch (e: Exception) {
            Log.d("Spo2Tracker", "HEART_RATE not available")
        }
    }

    private fun readStatus(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val status = dataPoint.getValue(
                ValueKey.SpO2Set.STATUS
            )

            onSampleReceived(
                "SPO2",
                timestamp,
                "spo2_status",
                status.toString(),
                null
            )

            if (status == 2) {
                stopMeasurement()
            }
        } catch (e: Exception) {
            Log.d("Spo2Tracker", "STATUS not available")
        }
    }
}
```

---

## 17. 저장 스키마 예시

### 17.1 SpO2 전용 테이블

```kotlin
data class Spo2Sample(
    val id: Long = 0L,
    val subjectId: String,
    val sessionId: String,
    val timestamp: Long,
    val spo2: Int?,
    val heartRate: Int?,
    val status: Int?
)
```

### 17.2 전체 센서 공통 long-format

```kotlin
data class SensorSample(
    val id: Long = 0L,
    val subjectId: String,
    val sessionId: String,
    val deviceId: String,
    val sensorType: String,
    val trackerType: String,
    val timestamp: Long,
    val valueName: String,
    val value: String,
    val unit: String?,
    val status: Int?
)
```

---

## 18. Long-format 저장 예시

| sensorType | trackerType | valueName | value | unit | status |
|---|---|---|---|---|---|
| `SPO2` | `SPO2_ON_DEMAND` | `spo2` | `97` | `%` | `2` |
| `SPO2` | `SPO2_ON_DEMAND` | `spo2_heart_rate` | `78` | `bpm` | `2` |
| `SPO2` | `SPO2_ON_DEMAND` | `spo2_status` | `2` | `status_code` | `2` |

---

## 19. SpO2 품질 관리 기준

### 19.1 측정 완료 여부 확인

```kotlin
if (spo2Status == 2) {
    // SpO2 measurement completed
}
```

`STATUS == 2`일 때 SpO2 측정 완료로 판단한다.

---

### 19.2 Timeout 확인

```kotlin
if (spo2Status == -6) {
    // Time out
}
```

`STATUS == -6`이면 측정 시간이 초과된 상태이다.

---

### 19.3 Low Signal Quality 확인

```kotlin
if (spo2Status == -5) {
    // Signal quality is low
}
```

`STATUS == -5`이면 신호 품질이 낮은 상태이다.

---

### 19.4 Motion Artifact 확인

```kotlin
if (spo2Status == -4) {
    // Device moved during measurement
}
```

`STATUS == -4`이면 측정 중 사용자의 움직임 또는 워치 움직임이 감지된 상태이다.

---

## 20. SpO2 측정 권장 흐름

```text
1. HealthTrackingService 연결
2. Capability 확인
3. SPO2_ON_DEMAND 지원 여부 확인
4. SpO2 tracker 생성
5. 사용자에게 자세 고정 및 워치 착용 상태 안내
6. setEventListener() 호출
7. onDataReceived()에서 SpO2 DataPoint 수신
8. SPO2, HEART_RATE, STATUS 저장
9. STATUS == 2이면 측정 완료로 판단
10. 측정 종료 시 unsetEventListener()
11. 데이터 저장 또는 Phone App 전송
```

---

## 21. Capability 확인 예시

```kotlin
fun isSpo2Supported(
    supportedTypes: List<HealthTrackerType>
): Boolean {
    return supportedTypes.contains(
        HealthTrackerType.SPO2_ON_DEMAND
    )
}
```

사용 예:

```kotlin
if (isSpo2Supported(supportedTypes)) {
    spo2TrackerManager.createTracker()
}
```

---

## 22. On-demand 사용 주의사항

SpO2는 on-demand tracker이므로 다음 구조를 권장한다.

```text
continuous tracker 실행 중
↓
SpO2 측정 요청
↓
continuous tracker 일시 중지
↓
SPO2_ON_DEMAND 실행
↓
STATUS == 2 또는 error status 수신
↓
SpO2 listener 해제
↓
continuous tracker 재시작
```

주의사항:

- SpO2 측정은 일반적으로 약 30초 정도 걸린다.
- 사용자의 자세와 워치 착용 위치가 결과 품질에 영향을 준다.
- `STATUS` 값을 반드시 저장해야 한다.
- `STATUS == 2`인 경우에만 측정 완료로 판단한다.
- `STATUS == -4`, `-5`, `-6`이면 측정 실패 또는 품질 저하로 처리한다.
- 측정 실패 시 워치 착용 위치와 사용자 자세를 확인하고 재측정한다.

---

## 23. 공식 문서 링크

- `ValueKey.SpO2Set`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/ValueKey.SpO2Set.html

- `HealthTrackerType`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/HealthTrackerType.html

- `DataPoint`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/DataPoint.html

- `HealthTracker.TrackerEventListener`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/HealthTracker.TrackerEventListener.html
