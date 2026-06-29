# 03. Samsung Health Sensor SDK ECG API

## 1. 목적

이 문서는 Samsung Health Sensor SDK의 ECG API를 정리한다.

ECG는 `continuous tracker`가 아니라 `on-demand tracker`이다.  
따라서 장시간 연속 측정용이 아니라, 사용자가 Galaxy Watch의 전극을 접촉한 상태에서 짧은 시간 동안 측정하는 방식으로 설계한다.

---

## 2. API 요약

| 항목 | 내용 |
|---|---|
| TrackerType | `HealthTrackerType.ECG_ON_DEMAND` |
| ValueKey Set | `ValueKey.EcgSet` |
| 측정 방식 | On-demand |
| 권장 파일 | `watch-app/sensor/EcgTrackerManager.kt` |
| 주요 수집값 | ECG waveform, PPG Green, Lead-off flag, Sequence, ECG threshold |

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
HealthTrackerType.ECG_ON_DEMAND
```

`ECG_ON_DEMAND`는 ECG 측정용 `HealthTrackerType`이다.

```kotlin
val ecgTracker = healthTrackingService.getHealthTracker(
    HealthTrackerType.ECG_ON_DEMAND
)
```

---

## 5. ValueKey Set

```kotlin
ValueKey.EcgSet
```

`ValueKey.EcgSet`은 `ECG_ON_DEMAND`에서 수신되는 `DataPoint` 값을 꺼내기 위한 key set이다.

공식 문서 기준 `ValueKey.EcgSet`은 다음 `ValueKey`를 포함한다.

```kotlin
ValueKey.EcgSet.ECG_MV
ValueKey.EcgSet.PPG_GREEN
ValueKey.EcgSet.LEAD_OFF
ValueKey.EcgSet.SEQUENCE
ValueKey.EcgSet.MAX_THRESHOLD_MV
ValueKey.EcgSet.MIN_THRESHOLD_MV
```

---

## 6. ValueKey 목록

| ValueKey | Type | 의미 | 단위 / 범위 |
|---|---|---|---|
| `ValueKey.EcgSet.ECG_MV` | `Float` | ECG waveform value | millivolts, `mV` |
| `ValueKey.EcgSet.PPG_GREEN` | `Integer` | ECG 측정 중 함께 제공되는 Green PPG value | raw |
| `ValueKey.EcgSet.LEAD_OFF` | `Integer` | 사용자가 Galaxy Watch 전극 키를 접촉하고 있는지 나타내는 flag | `0` 또는 기타 값 |
| `ValueKey.EcgSet.SEQUENCE` | `Integer` | ECG data sequence number | `0 ~ 255` |
| `ValueKey.EcgSet.MAX_THRESHOLD_MV` | `Float` | `ECG_MV` 최대 threshold | mV |
| `ValueKey.EcgSet.MIN_THRESHOLD_MV` | `Float` | `ECG_MV` 최소 threshold | mV |

---

## 7. ValueKey 상세

### 7.1 `ECG_MV`

```kotlin
ValueKey.EcgSet.ECG_MV
```

ECG waveform 값이다.

```kotlin
val ecgMv = dataPoint.getValue(
    ValueKey.EcgSet.ECG_MV
)
```

| 항목 | 내용 |
|---|---|
| Type | `Float` |
| 의미 | ECG signal value |
| 단위 | `mV` |
| 저장 변수명 | `ecgMv` |
| 저장 valueName | `ecg_mv` |

---

### 7.2 `PPG_GREEN`

```kotlin
ValueKey.EcgSet.PPG_GREEN
```

ECG 측정 중 함께 제공되는 Green PPG 값이다.

```kotlin
val ppgGreen = dataPoint.getValue(
    ValueKey.EcgSet.PPG_GREEN
)
```

| 항목 | 내용 |
|---|---|
| Type | `Integer` |
| 의미 | Green PPG value |
| 단위 | raw |
| 저장 변수명 | `ppgGreen` |
| 저장 valueName | `ecg_ppg_green` |

---

### 7.3 `LEAD_OFF`

```kotlin
ValueKey.EcgSet.LEAD_OFF
```

사용자가 Galaxy Watch의 전극 키를 제대로 접촉하고 있는지 나타내는 flag이다.

```kotlin
val leadOff = dataPoint.getValue(
    ValueKey.EcgSet.LEAD_OFF
)
```

| 값 | 의미 |
|---|---|
| `0` | Galaxy Watch 전극 키 접촉 정상 |
| 기타 값 | 손가락이 전극 키에 제대로 접촉하지 않음 |

저장 변수명:

```kotlin
val leadOff: Int
```

저장 valueName:

```text
lead_off
```

`LEAD_OFF != 0`이면 ECG 품질 문제가 있을 가능성이 있으므로 분석 단계에서 제외하거나 별도 flag로 처리한다.

---

### 7.4 `SEQUENCE`

```kotlin
ValueKey.EcgSet.SEQUENCE
```

ECG data의 sequence number이다.

```kotlin
val sequence = dataPoint.getValue(
    ValueKey.EcgSet.SEQUENCE
)
```

| 항목 | 내용 |
|---|---|
| Type | `Integer` |
| 의미 | ECG data sequence number |
| 범위 | `0 ~ 255` |
| 저장 변수명 | `sequence` |
| 저장 valueName | `sequence` |

`SEQUENCE`는 ECG waveform sample의 순서 복원과 누락 확인에 사용한다.

---

### 7.5 `MAX_THRESHOLD_MV`

```kotlin
ValueKey.EcgSet.MAX_THRESHOLD_MV
```

`ECG_MV` 값의 최대 threshold이다.

```kotlin
val maxThresholdMv = dataPoint.getValue(
    ValueKey.EcgSet.MAX_THRESHOLD_MV
)
```

| 항목 | 내용 |
|---|---|
| Type | `Float` |
| 의미 | ECG_MV maximum threshold |
| 단위 | `mV` |
| 저장 변수명 | `maxThresholdMv` |
| 저장 valueName | `max_threshold_mv` |

`ECG_MV > MAX_THRESHOLD_MV`이면 saturation 상태로 보고 재측정이 필요할 수 있다.

---

### 7.6 `MIN_THRESHOLD_MV`

```kotlin
ValueKey.EcgSet.MIN_THRESHOLD_MV
```

`ECG_MV` 값의 최소 threshold이다.

```kotlin
val minThresholdMv = dataPoint.getValue(
    ValueKey.EcgSet.MIN_THRESHOLD_MV
)
```

| 항목 | 내용 |
|---|---|
| Type | `Float` |
| 의미 | ECG_MV minimum threshold |
| 단위 | `mV` |
| 저장 변수명 | `minThresholdMv` |
| 저장 valueName | `min_threshold_mv` |

`ECG_MV < MIN_THRESHOLD_MV`이면 saturation 상태로 보고 재측정이 필요할 수 있다.

---

## 8. DataPoint 구조

ECG 측정 시 `onDataReceived()`로 들어오는 ECG data list는 센서 동작 상태에 따라 `5개` 또는 `10개`의 `DataPoint`를 포함할 수 있다.

### 8.1 DataPoint가 5개인 경우

| DataPoint | 포함 값 |
|---|---|
| 1st `DataPoint` | `ECG_MV`, `PPG_GREEN`, `LEAD_OFF`, `SEQUENCE`, `MAX_THRESHOLD_MV`, `MIN_THRESHOLD_MV` |
| 2nd `DataPoint` | `ECG_MV` |
| 3rd `DataPoint` | `ECG_MV` |
| 4th `DataPoint` | `ECG_MV` |
| 5th `DataPoint` | `ECG_MV` |

### 8.2 DataPoint가 10개인 경우

| DataPoint | 포함 값 |
|---|---|
| 1st `DataPoint` | `ECG_MV`, `PPG_GREEN`, `LEAD_OFF`, `SEQUENCE`, `MAX_THRESHOLD_MV`, `MIN_THRESHOLD_MV` |
| 2nd `DataPoint` | `ECG_MV` |
| 3rd `DataPoint` | `ECG_MV` |
| 4th `DataPoint` | `ECG_MV` |
| 5th `DataPoint` | `ECG_MV` |
| 6th `DataPoint` | `ECG_MV`, `PPG_GREEN` |
| 7th `DataPoint` | `ECG_MV` |
| 8th `DataPoint` | `ECG_MV` |
| 9th `DataPoint` | `ECG_MV` |
| 10th `DataPoint` | `ECG_MV` |

따라서 모든 `DataPoint`에서 모든 `ValueKey`를 무조건 읽으면 안 된다.  
`ECG_MV`는 여러 `DataPoint`에 반복적으로 포함되지만, `LEAD_OFF`, `SEQUENCE`, threshold 값은 특정 `DataPoint`에만 포함될 수 있다.

---

## 9. 저장 변수명 권장

```kotlin
val timestamp: Long
val ecgMv: Float?
val ppgGreen: Int?
val leadOff: Int?
val sequence: Int?
val maxThresholdMv: Float?
val minThresholdMv: Float?
```

---

## 10. 저장 valueName 권장

| valueName | 의미 |
|---|---|
| `ecg_mv` | ECG waveform value |
| `ecg_ppg_green` | ECG 측정 중 PPG Green value |
| `lead_off` | 전극 접촉 여부 |
| `sequence` | ECG sample sequence |
| `max_threshold_mv` | ECG_MV maximum threshold |
| `min_threshold_mv` | ECG_MV minimum threshold |

---

## 11. 권장 파일 위치

```text
watch-app/
└── sensor/
    └── EcgTrackerManager.kt
```

---

## 12. ECG Tracker 생성

```kotlin
private var ecgTracker: HealthTracker? = null

fun createTracker() {
    ecgTracker = healthTrackingService.getHealthTracker(
        HealthTrackerType.ECG_ON_DEMAND
    )
}
```

---

## 13. ECG 측정 시작 / 종료

### 13.1 측정 시작

```kotlin
fun startMeasurement() {
    ecgTracker?.setEventListener(ecgListener)
}
```

### 13.2 측정 종료

```kotlin
fun stopMeasurement() {
    ecgTracker?.unsetEventListener()
}
```

---

## 14. ECG Listener 예시

```kotlin
private val ecgListener = object : HealthTracker.TrackerEventListener {

    override fun onDataReceived(dataPoints: List<DataPoint>) {
        dataPoints.forEach { dataPoint ->

            val timestamp = dataPoint.timestamp

            readEcgMv(dataPoint, timestamp)
            readPpgGreen(dataPoint, timestamp)
            readLeadOff(dataPoint, timestamp)
            readSequence(dataPoint, timestamp)
            readMaxThresholdMv(dataPoint, timestamp)
            readMinThresholdMv(dataPoint, timestamp)
        }
    }

    override fun onFlushCompleted() {
        Log.d("EcgTracker", "Flush completed")
    }

    override fun onError(error: HealthTracker.TrackerError) {
        Log.e("EcgTracker", "Error: $error")
    }
}
```

---

## 15. Key별 읽기 함수 예시

ECG `DataPoint`는 위치에 따라 포함하는 key가 다르다.  
따라서 key가 없는 `DataPoint`에서 `getValue()`를 호출할 수 있으므로 안전하게 예외 처리한다.

### 15.1 `ECG_MV`

```kotlin
private fun readEcgMv(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val ecgMv = dataPoint.getValue(
            ValueKey.EcgSet.ECG_MV
        )

        onSampleReceived(
            "ECG",
            timestamp,
            "ecg_mv",
            ecgMv.toString(),
            null
        )
    } catch (e: Exception) {
        Log.d("EcgTracker", "ECG_MV not available in this DataPoint")
    }
}
```

### 15.2 `PPG_GREEN`

```kotlin
private fun readPpgGreen(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val ppgGreen = dataPoint.getValue(
            ValueKey.EcgSet.PPG_GREEN
        )

        onSampleReceived(
            "ECG",
            timestamp,
            "ecg_ppg_green",
            ppgGreen.toString(),
            null
        )
    } catch (e: Exception) {
        Log.d("EcgTracker", "PPG_GREEN not available in this DataPoint")
    }
}
```

### 15.3 `LEAD_OFF`

```kotlin
private fun readLeadOff(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val leadOff = dataPoint.getValue(
            ValueKey.EcgSet.LEAD_OFF
        )

        onSampleReceived(
            "ECG",
            timestamp,
            "lead_off",
            leadOff.toString(),
            leadOff
        )
    } catch (e: Exception) {
        Log.d("EcgTracker", "LEAD_OFF not available in this DataPoint")
    }
}
```

### 15.4 `SEQUENCE`

```kotlin
private fun readSequence(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val sequence = dataPoint.getValue(
            ValueKey.EcgSet.SEQUENCE
        )

        onSampleReceived(
            "ECG",
            timestamp,
            "sequence",
            sequence.toString(),
            null
        )
    } catch (e: Exception) {
        Log.d("EcgTracker", "SEQUENCE not available in this DataPoint")
    }
}
```

### 15.5 `MAX_THRESHOLD_MV`

```kotlin
private fun readMaxThresholdMv(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val maxThresholdMv = dataPoint.getValue(
            ValueKey.EcgSet.MAX_THRESHOLD_MV
        )

        onSampleReceived(
            "ECG",
            timestamp,
            "max_threshold_mv",
            maxThresholdMv.toString(),
            null
        )
    } catch (e: Exception) {
        Log.d("EcgTracker", "MAX_THRESHOLD_MV not available in this DataPoint")
    }
}
```

### 15.6 `MIN_THRESHOLD_MV`

```kotlin
private fun readMinThresholdMv(
    dataPoint: DataPoint,
    timestamp: Long
) {
    try {
        val minThresholdMv = dataPoint.getValue(
            ValueKey.EcgSet.MIN_THRESHOLD_MV
        )

        onSampleReceived(
            "ECG",
            timestamp,
            "min_threshold_mv",
            minThresholdMv.toString(),
            null
        )
    } catch (e: Exception) {
        Log.d("EcgTracker", "MIN_THRESHOLD_MV not available in this DataPoint")
    }
}
```

---

## 16. 전체 `EcgTrackerManager.kt` 예시

```kotlin
class EcgTrackerManager(
    private val healthTrackingService: HealthTrackingService,
    private val onSampleReceived: (
        sensorType: String,
        timestamp: Long,
        valueName: String,
        value: String,
        status: Int?
    ) -> Unit
) {

    private var ecgTracker: HealthTracker? = null

    fun createTracker() {
        ecgTracker = healthTrackingService.getHealthTracker(
            HealthTrackerType.ECG_ON_DEMAND
        )
    }

    fun startMeasurement() {
        ecgTracker?.setEventListener(ecgListener)
    }

    fun stopMeasurement() {
        ecgTracker?.unsetEventListener()
    }

    private val ecgListener = object : HealthTracker.TrackerEventListener {

        override fun onDataReceived(dataPoints: List<DataPoint>) {
            dataPoints.forEach { dataPoint ->

                val timestamp = dataPoint.timestamp

                readEcgMv(dataPoint, timestamp)
                readPpgGreen(dataPoint, timestamp)
                readLeadOff(dataPoint, timestamp)
                readSequence(dataPoint, timestamp)
                readMaxThresholdMv(dataPoint, timestamp)
                readMinThresholdMv(dataPoint, timestamp)
            }
        }

        override fun onFlushCompleted() {
            Log.d("EcgTracker", "Flush completed")
        }

        override fun onError(error: HealthTracker.TrackerError) {
            Log.e("EcgTracker", "Error: $error")
        }
    }

    private fun readEcgMv(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val ecgMv = dataPoint.getValue(
                ValueKey.EcgSet.ECG_MV
            )

            onSampleReceived(
                "ECG",
                timestamp,
                "ecg_mv",
                ecgMv.toString(),
                null
            )
        } catch (e: Exception) {
            Log.d("EcgTracker", "ECG_MV not available")
        }
    }

    private fun readPpgGreen(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val ppgGreen = dataPoint.getValue(
                ValueKey.EcgSet.PPG_GREEN
            )

            onSampleReceived(
                "ECG",
                timestamp,
                "ecg_ppg_green",
                ppgGreen.toString(),
                null
            )
        } catch (e: Exception) {
            Log.d("EcgTracker", "PPG_GREEN not available")
        }
    }

    private fun readLeadOff(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val leadOff = dataPoint.getValue(
                ValueKey.EcgSet.LEAD_OFF
            )

            onSampleReceived(
                "ECG",
                timestamp,
                "lead_off",
                leadOff.toString(),
                leadOff
            )
        } catch (e: Exception) {
            Log.d("EcgTracker", "LEAD_OFF not available")
        }
    }

    private fun readSequence(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val sequence = dataPoint.getValue(
                ValueKey.EcgSet.SEQUENCE
            )

            onSampleReceived(
                "ECG",
                timestamp,
                "sequence",
                sequence.toString(),
                null
            )
        } catch (e: Exception) {
            Log.d("EcgTracker", "SEQUENCE not available")
        }
    }

    private fun readMaxThresholdMv(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val maxThresholdMv = dataPoint.getValue(
                ValueKey.EcgSet.MAX_THRESHOLD_MV
            )

            onSampleReceived(
                "ECG",
                timestamp,
                "max_threshold_mv",
                maxThresholdMv.toString(),
                null
            )
        } catch (e: Exception) {
            Log.d("EcgTracker", "MAX_THRESHOLD_MV not available")
        }
    }

    private fun readMinThresholdMv(
        dataPoint: DataPoint,
        timestamp: Long
    ) {
        try {
            val minThresholdMv = dataPoint.getValue(
                ValueKey.EcgSet.MIN_THRESHOLD_MV
            )

            onSampleReceived(
                "ECG",
                timestamp,
                "min_threshold_mv",
                minThresholdMv.toString(),
                null
            )
        } catch (e: Exception) {
            Log.d("EcgTracker", "MIN_THRESHOLD_MV not available")
        }
    }
}
```

---

## 17. 저장 스키마 예시

### 17.1 ECG 전용 테이블

```kotlin
data class EcgSample(
    val id: Long = 0L,
    val subjectId: String,
    val sessionId: String,
    val timestamp: Long,
    val sequence: Int?,
    val ecgMv: Float?,
    val ppgGreen: Int?,
    val leadOff: Int?,
    val maxThresholdMv: Float?,
    val minThresholdMv: Float?
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
| `ECG` | `ECG_ON_DEMAND` | `ecg_mv` | `0.124` | `mV` | `null` |
| `ECG` | `ECG_ON_DEMAND` | `ecg_ppg_green` | `123456` | `raw` | `null` |
| `ECG` | `ECG_ON_DEMAND` | `lead_off` | `0` | `flag` | `0` |
| `ECG` | `ECG_ON_DEMAND` | `sequence` | `12` | `index` | `null` |
| `ECG` | `ECG_ON_DEMAND` | `max_threshold_mv` | `1.5` | `mV` | `null` |
| `ECG` | `ECG_ON_DEMAND` | `min_threshold_mv` | `-1.5` | `mV` | `null` |

---

## 19. ECG 품질 관리 기준

### 19.1 Lead-off 확인

```kotlin
if (leadOff != 0) {
    // Electrode contact problem
}
```

`LEAD_OFF` 값이 `0`이 아니면 사용자가 전극을 제대로 접촉하지 않은 상태로 본다.

---

### 19.2 Saturation 확인

```kotlin
if (ecgMv > maxThresholdMv || ecgMv < minThresholdMv) {
    // ECG saturation
}
```

`ECG_MV`가 threshold 범위를 벗어나면 saturation 상태로 보고 재측정이 필요할 수 있다.

---

### 19.3 Sequence 확인

```kotlin
// sequence discontinuity check
```

`SEQUENCE`는 `0 ~ 255` 범위에서 제공된다. ECG sample 순서 복원과 누락 확인에 활용한다.

---

## 20. ECG 측정 권장 흐름

```text
1. HealthTrackingService 연결
2. Capability 확인
3. ECG_ON_DEMAND 지원 여부 확인
4. ECG tracker 생성
5. 사용자에게 착용/전극 접촉 안내
6. setEventListener() 호출
7. onDataReceived()에서 ECG DataPoint 수신
8. ECG_MV, PPG_GREEN, LEAD_OFF, SEQUENCE, threshold 저장
9. 측정 종료 시 unsetEventListener()
10. 데이터 저장 또는 Phone App 전송
```

---

## 21. Capability 확인 예시

```kotlin
fun isEcgSupported(
    supportedTypes: List<HealthTrackerType>
): Boolean {
    return supportedTypes.contains(
        HealthTrackerType.ECG_ON_DEMAND
    )
}
```

사용 예:

```kotlin
if (isEcgSupported(supportedTypes)) {
    ecgTrackerManager.createTracker()
}
```

---

## 22. On-demand 사용 주의사항

ECG는 on-demand tracker이므로 다음 구조를 권장한다.

```text
continuous tracker 실행 중
↓
ECG 측정 요청
↓
continuous tracker 일시 중지
↓
ECG_ON_DEMAND 실행
↓
ECG 측정 종료
↓
ECG listener 해제
↓
continuous tracker 재시작
```

주의사항:

- ECG는 foreground 측정 중심으로 설계한다.
- 사용자의 전극 접촉이 필요하다.
- `LEAD_OFF` 값을 반드시 저장한다.
- `SEQUENCE` 값을 저장해 sample 순서와 누락 여부를 확인한다.
- `MAX_THRESHOLD_MV`, `MIN_THRESHOLD_MV`를 저장해 saturation 여부를 판단한다.
- ECG 측정 중 continuous tracker 값을 함께 사용할 경우 품질 문제가 생길 수 있으므로 세션을 분리하는 것이 안전하다.

---

## 23. 공식 문서 링크

- `ValueKey.EcgSet`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/ValueKey.EcgSet.html

- `HealthTrackerType`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/HealthTrackerType.html

- `DataPoint`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/data/DataPoint.html

- `HealthTracker.TrackerEventListener`  
  https://developer.samsung.com/health/sensor/api-reference/com/samsung/android/service/health/tracking/HealthTracker.TrackerEventListener.html
