# Design Document: AI-Powered Mushroom Growth Monitoring System

## Overview

This design specifies a comprehensive AI-powered monitoring system for semi-commercial mushroom cultivation in India. The system architecture prioritizes offline-first operation, edge computing, and resilience to power and connectivity challenges while providing real-time intelligence across the complete mushroom growth lifecycle.

### Design Principles

1. **Offline-First**: All critical functionality operates without internet connectivity
2. **Edge Intelligence**: ML inference runs locally on affordable hardware
3. **Resilience**: Graceful degradation under power/connectivity constraints
4. **Simplicity**: Minimal configuration, automatic operation, clear interfaces
5. **Localization**: Optimized for Indian substrates, climate, and economic constraints
6. **Privacy**: Data remains local by default, cloud sync is optional and anonymized

### Technology Stack

- **Edge Platform**: Raspberry Pi 4 (4GB RAM) or equivalent ARM-based SBC
- **Operating System**: Linux-based (Raspberry Pi OS Lite or Ubuntu Server)
- **ML Framework**: TensorFlow Lite for edge inference
- **Computer Vision**: OpenCV for image processing
- **Sensor Interface**: Python with smbus2/spidev for I2C/SPI communication
- **Database**: SQLite for local time-series and metadata storage
- **Web Interface**: Lightweight web server (Flask/FastAPI) with responsive HTML/CSS/JS frontend
- **Language Support**: i18n framework for Hindi/English localization

## Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                     Grower Interface Layer                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Web Dashboard│  │  SMS Alerts  │  │  Mobile App  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                   Application Logic Layer                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Batch Manager│  │ Alert Engine │  │ Report Gen   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Stage Tracker│  │ Yield Predict│  │ Harvest Pred │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                    ML Inference Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Contamination│  │ Growth Stage │  │ Yield Model  │      │
│  │   Detector   │  │  Classifier  │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │ Harvest Time │  │ Anomaly Det  │                        │
│  │   Predictor  │  │              │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                    Data Collection Layer                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Sensor Reader│  │ Image Capture│  │ Data Storage │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                      Hardware Layer                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Temp/Humidity│  │  CO2 Sensor  │  │    Camera    │      │
│  │    Sensor    │  │              │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │ Light Sensor │  │ Battery/UPS  │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **Sensor Collection**: Hardware sensors sample environmental data every 5 minutes
2. **Image Acquisition**: Camera captures images with adaptive frequency:
   - Colonization phase (Day 1-15): 4x daily (every 6 hours)
   - Pinning/fruiting phase (Day 15+): 8-12x daily (every 2-3 hours)
3. **Preprocessing**: Raw sensor data normalized, images resized/enhanced
4. **ML Inference**: TensorFlow Lite models process data locally
5. **Decision Logic**: Application layer interprets ML outputs, generates alerts
6. **Storage**: All data persisted to local SQLite database
7. **Presentation**: Web interface queries database, displays real-time status
8. **Sync (when online)**: Background process uploads anonymized data, downloads model updates

**Note:** Yield forecasting is a Phase 2 capability (6-12 months post-deployment). Phase 1 focuses on contamination detection + growth stage tracking + harvest timing, which provide immediate value without requiring extensive ground truth datasets.

### Power Management Architecture

```
Mains Power ──┬──> Edge Device
              │
              └──> Battery Charger ──> Battery ──> Edge Device
                                                    (on power loss)

Power States:
- Normal: Full operation, 5-min sensor sampling, 4x daily imaging
- Low Battery (<20%): Alert generated, continue normal operation
- Critical Battery (<10%): Low-power mode, 15-min sampling, 2x daily imaging
- Battery Depleted: Graceful shutdown, data persisted
```

## Physical Setup and Batch-to-Camera Mapping

### Camera Placement Options

**Option A (Simple deployment): One camera per rack zone**
- Each camera covers 8-12 bags in frame
- Manual labeling during setup: "Camera 1 = Rack A, Bags 1-10"
- Image regions mapped to bag IDs during calibration wizard

**Option B (Advanced deployment): Single wide-angle camera + object detection**
- YOLOv5/v8 bag detection model identifies individual bags in frame
- Manual bag ID assignment using QR code labels or colored markers
- Automatic bag tracking across frames using bounding box persistence

### Data Structure for Batch Tracking

Each captured image tagged with:
- batch_id
- bag_id
- camera_id
- timestamp
- rack_location

SQLite table structure:
```sql
images(
    id, 
    batch_id, 
    bag_id, 
    camera_id, 
    timestamp, 
    filepath, 
    growth_stage, 
    contamination_detected
)
```

Contamination alerts include bag location:
- Example: "Bag #47, Rack A, Position 3 - Green mold detected"

### Setup Wizard Requirements

1. Farmer positions camera
2. System captures test image
3. Farmer draws bounding boxes around each bag or rack zone
4. System assigns bag IDs
5. Calibration stored for consistent tracking

### Physical Camera-to-Rack Placement Diagram

**Camera Specifications:**
- Field of View (FOV): 62.2° diagonal (Pi Camera Module v2)
- Recommended camera-to-bag distance: 0.5-1.5 meters
- Lighting: Natural light + supplemental LED (5000K daylight, 500-1000 lux)

**Placement Guidelines:**
```
                    Camera (mounted on ceiling/wall)
                           |
                           | 0.5-1.5m
                           |
                           v
    ┌─────────────────────────────────────────┐
    │  Rack A - Bags positioned in 3x4 grid   │
    │  ┌───┐ ┌───┐ ┌───┐ ┌───┐               │
    │  │ 1 │ │ 2 │ │ 3 │ │ 4 │  Row 1        │
    │  └───┘ └───┘ └───┘ └───┘               │
    │  ┌───┐ ┌───┐ ┌───┐ ┌───┐               │
    │  │ 5 │ │ 6 │ │ 7 │ │ 8 │  Row 2        │
    │  └───┘ └───┘ └───┘ └───┘               │
    │  ┌───┐ ┌───┐ ┌───┐ ┌───┐               │
    │  │ 9 │ │10 │ │11 │ │12 │  Row 3        │
    │  └───┘ └───┘ └───┘ └───┘               │
    └─────────────────────────────────────────┘
```

**Minimum/Maximum Distances:**
- Minimum: 0.5m (closer causes distortion, bags at edges out of focus)
- Maximum: 1.5m (farther reduces resolution, contamination details lost)
- Optimal: 0.8-1.0m (balance between coverage and detail)

## Components and Interfaces

### 1. Sensor Reader Component

**Responsibility**: Interface with hardware sensors, collect environmental data

**Interfaces**:
```python
class SensorReader:
    def read_temperature() -> float
    def read_humidity() -> float
    def read_co2() -> int
    def read_light() -> int
    def get_all_readings() -> EnvironmentalReading
    def calibrate_sensors() -> CalibrationResult
    def check_sensor_health() -> SensorHealthStatus
```

**Implementation Details**:
- Uses I2C protocol for DHT22/SHT31 (temp/humidity), MH-Z19 (CO2), BH1750 (light)
- Implements retry logic (3 attempts) for failed sensor reads
- Validates readings against physical limits (temp: -10 to 60°C, humidity: 0-100%, CO2: 0-5000ppm)
- Caches last valid reading for up to 15 minutes if sensor fails
- Logs all sensor errors with timestamps for diagnostics

### 2. Image Capture Component

**Responsibility**: Capture images from camera, handle exposure and storage

**Interfaces**:
```python
class ImageCapture:
    def capture_image() -> Image
    def adjust_exposure(ambient_light: int) -> ExposureSettings
    def save_image(image: Image, batch_id: str) -> ImageMetadata
    def get_storage_usage() -> StorageStats
    def cleanup_old_images(keep_days: int) -> int
    def verify_camera_health() -> CameraHealthStatus
```

**Implementation Details**:
- Uses OpenCV VideoCapture for USB camera interface
- Captures at 2592x1944 resolution (5MP), saves as JPEG with 85% quality
- Auto-exposure based on light sensor reading (low light: longer exposure, high light: shorter)
- Stores images in `/data/images/{batch_id}/{YYYY-MM-DD}/` directory structure
- Implements storage quota: keeps last 90 days, deletes oldest when >80% full
- Captures test image every hour to verify camera functionality

### 3. Contamination Detector Component

**Responsibility**: Analyze images for contamination using ML model

**Interfaces**:
```python
class ContaminationDetector:
    def analyze_image(image: Image) -> ContaminationResult
    def load_model(model_path: str) -> bool
    def preprocess_image(image: Image) -> Tensor
    def postprocess_output(model_output: Tensor) -> ContaminationResult
```

**Model Architecture**:
- Base: MobileNetV2 (lightweight, optimized for edge)
- Input: 224x224 RGB image
- Output: Multi-label classification (bacterial, fungal, pest, clean) + bounding boxes
- Training: Transfer learning on ImageNet, fine-tuned on 10,000+ labeled Indian substrate images
- Quantization: INT8 quantization for 4x speedup on edge hardware
- Inference time: <2 seconds on Raspberry Pi 4

**Contamination Classes**:
- Bacterial (wet spots, slime, discoloration)
- Fungal - Green mold (Trichoderma)
- Fungal - Black mold (Aspergillus)
- Fungal - Cobweb mold
- Pest - Mites
- Pest - Flies
- Clean (no contamination)

**Output Format**:
```python
ContaminationResult:
    - detected: bool
    - contamination_type: List[str]  # Can have multiple types
    - confidence: float (0.0-1.0)
    - severity: str  # "low", "medium", "high", "critical"
    - bounding_boxes: List[BoundingBox]  # Regions of contamination
    - timestamp: datetime
```

### 4. Growth Stage Classifier Component

**Responsibility**: Determine current growth stage from visual and temporal data

**Interfaces**:
```python
class GrowthStageClassifier:
    def classify_stage(image: Image, days_since_inoculation: int, 
                       substrate_type: str) -> GrowthStage
    def estimate_colonization_percentage(image: Image) -> float
    def count_pins(image: Image) -> int
    def measure_fruiting_body_size(image: Image) -> List[float]
```

**Stage Classification Logic**:
- **Colonization** (Days 0-14): Mycelium spreading through substrate
  - Visual: White mycelium coverage increasing
  - Metric: Colonization percentage (0-100%)
  - Transition: >90% colonization OR day 14 reached
  
- **Pinning** (Days 14-18): Pin formation (baby mushrooms)
  - Visual: Small pin-like structures appearing
  - Metric: Pin count
  - Transition: Pins >5mm in size
  
- **Fruiting** (Days 18-25): Mushroom growth
  - Visual: Mushrooms expanding in size
  - Metric: Average cap diameter
  - Transition: Caps reach variety-specific harvest size
  
- **Harvest** (Days 25-28): Ready for harvest
  - Visual: Caps fully opened, gills visible
  - Metric: Cap opening angle
  - Transition: Manual batch completion by grower

**Model Architecture**:
- Hybrid approach: CNN for visual features + decision tree for temporal logic
- CNN: EfficientNet-Lite0 (224x224 input)
- Features: Mycelium density, pin presence, mushroom size, cap opening
- Decision tree: Incorporates days since inoculation, substrate type, variety

### 5. Yield Forecasting Component

**Responsibility**: Predict final yield based on current growth metrics

**Interfaces**:
```python
class YieldForecaster:
    def predict_yield(batch_id: str) -> YieldPrediction
    def update_forecast(batch_id: str, current_metrics: GrowthMetrics) -> YieldPrediction
    def get_confidence_interval(prediction: YieldPrediction) -> Tuple[float, float]
```

**Prediction Model**:
- Algorithm: Gradient Boosting Regressor (XGBoost)
- Features:
  - Pin count at pinning stage
  - Average environmental conditions (temp, humidity, CO2) during colonization
  - Substrate type (one-hot encoded)
  - Mushroom variety (one-hot encoded)
  - Colonization duration (days)
  - Historical yield for same substrate/variety combination
- Target: Final yield in kilograms
- Training: 1000+ completed batches from Indian growers
- Accuracy: ±15% MAPE (Mean Absolute Percentage Error)

**Forecast Timeline**:
- Initial forecast: Generated at pinning stage (day 14-18)
- Daily updates: Forecast refined as fruiting progresses
- Confidence interval: 90% confidence bounds provided

### 6. Harvest Window Predictor Component

**Responsibility**: Predict optimal harvest timing

**Interfaces**:
```python
class HarvestPredictor:
    def predict_harvest_window(batch_id: str) -> HarvestWindow
    def check_harvest_readiness(batch_id: str) -> HarvestReadiness
```

**Prediction Logic**:
- Tracks mushroom cap diameter and opening angle from images
- Variety-specific thresholds:
  - Oyster: Cap 5-10cm, 70-80% open
  - Button: Cap 3-5cm, closed to slightly open
  - Shiitake: Cap 5-8cm, 60-70% open
- Environmental adjustment: High temp/low humidity accelerates maturity
- Harvest window: 12-24 hour optimal period before quality degradation
- Alert timing: 24 hours before window opens, urgent alert at window start

### 7. Alert Engine Component

**Responsibility**: Generate and deliver alerts based on system events

**Interfaces**:
```python
class AlertEngine:
    def generate_alert(alert_type: str, severity: str, message: str, 
                       batch_id: str) -> Alert
    def deliver_alert(alert: Alert, channels: List[str]) -> DeliveryStatus
    def check_duplicate(alert: Alert) -> bool
    def escalate_alert(alert_id: str) -> Alert
    def acknowledge_alert(alert_id: str, user_id: str) -> bool
```

**Alert Types**:
1. **Environmental**: Temp/humidity/CO2 out of range
2. **Contamination**: Contamination detected
3. **Harvest**: Harvest window approaching/open
4. **System Health**: Sensor failure, camera failure, low battery, overheating
5. **Forecast**: Yield forecast updated significantly

**Severity Levels**:
- **Low**: Informational, no immediate action needed
- **Medium**: Attention recommended within 24 hours
- **High**: Action recommended within 6 hours
- **Critical**: Immediate action required

**Delivery Channels**:
- **Dashboard**: Always displayed on web interface
- **SMS**: For high/critical alerts (requires GSM modem or cloud SMS service)
- **Push Notification**: For mobile app users
- **Email**: For non-urgent alerts (requires internet)

**Deduplication Logic**:
- Same alert type + batch + 6-hour window = duplicate (suppressed)
- Escalation: If alert unacknowledged for 24 hours, severity increases one level

### 8. Batch Manager Component

**Responsibility**: Manage cultivation batch lifecycle

**Interfaces**:
```python
class BatchManager:
    def create_batch(substrate: str, variety: str, inoculation_date: date) -> Batch
    def get_batch(batch_id: str) -> Batch
    def list_active_batches() -> List[Batch]
    def update_batch_stage(batch_id: str, new_stage: str) -> Batch
    def complete_batch(batch_id: str, actual_yield: float) -> Batch
    def get_batch_history(batch_id: str) -> BatchHistory
```

**Batch Lifecycle**:
1. **Created**: Grower inputs substrate, variety, inoculation date
2. **Active**: System monitors, generates predictions/alerts
3. **Harvest Ready**: Harvest window open
4. **Completed**: Grower marks as harvested, enters actual yield
5. **Archived**: Historical data retained for analytics

**Batch Data Model** (see Data Models section below)

### 9. Data Storage Component

**Responsibility**: Persist all system data locally

**Interfaces**:
```python
class DataStorage:
    def store_sensor_reading(reading: EnvironmentalReading) -> bool
    def store_image_metadata(metadata: ImageMetadata) -> bool
    def store_ml_result(result: MLResult) -> bool
    def store_alert(alert: Alert) -> bool
    def query_sensor_data(batch_id: str, start: datetime, end: datetime) -> List[EnvironmentalReading]
    def query_images(batch_id: str, start: datetime, end: datetime) -> List[ImageMetadata]
    def get_batch_summary(batch_id: str) -> BatchSummary
```

**Storage Implementation**:
- **Database**: SQLite (single file, no server, ACID compliant)
- **Location**: `/data/mushroom_monitor.db`
- **Backup**: Daily backup to `/data/backups/` (keep last 7 days)
- **Retention**: 90 days for sensor data, 90 days for images (configurable)
- **Encryption**: SQLite encryption extension (SQLCipher) for data at rest

### 10. Cloud Sync Component

**Responsibility**: Synchronize data with cloud when connectivity available

**Interfaces**:
```python
class CloudSync:
    def check_connectivity() -> bool
    def sync_data() -> SyncResult
    def upload_sensor_data(start: datetime, end: datetime) -> bool
    def upload_images(batch_id: str) -> bool
    def download_model_updates() -> List[ModelUpdate]
    def anonymize_data(data: Any) -> Any
```

**Sync Strategy**:
- **Trigger**: Every 6 hours if internet available
- **Priority**: Alerts > ML results > Sensor data > Images
- **Anonymization**: Remove grower PII, GPS coordinates, replace batch IDs with UUIDs
- **Bandwidth**: Compress data (gzip), throttle to 1 Mbps to avoid network saturation
- **Retry**: Exponential backoff (1min, 5min, 15min, 1hr) on failure
- **Model Updates**: Download new TFLite models, validate checksums, deploy after validation

### 11. Web Dashboard Component

**Responsibility**: Provide user interface for monitoring and control

**Interface**: HTTP REST API + Server-Sent Events (SSE) for real-time updates

**API Endpoints**:
```
GET  /api/batches                    # List all batches
POST /api/batches                    # Create new batch
GET  /api/batches/{id}               # Get batch details
PUT  /api/batches/{id}               # Update batch
POST /api/batches/{id}/complete      # Mark batch as completed

GET  /api/batches/{id}/sensors       # Get sensor data (time range)
GET  /api/batches/{id}/images        # Get image list
GET  /api/batches/{id}/alerts        # Get alerts
POST /api/alerts/{id}/acknowledge    # Acknowledge alert

GET  /api/system/health              # System health status
GET  /api/system/storage             # Storage usage
POST /api/system/calibrate           # Trigger sensor calibration

GET  /api/reports/batch/{id}         # Generate batch report
GET  /api/reports/summary            # Generate summary report

GET  /events                         # SSE stream for real-time updates
```

**Frontend**:
- Responsive design (mobile-first, works on phones/tablets)
- Real-time updates via Server-Sent Events
- Charts: Temperature/humidity/CO2 trends (Chart.js)
- Image gallery: Crop progression timeline
- Alert panel: Active alerts with priority indicators
- Batch cards: Current stage, yield forecast, harvest countdown
- Language toggle: Hindi/English

### 12. Report Generator Component

**Responsibility**: Generate analytics and reports

**Interfaces**:
```python
class ReportGenerator:
    def generate_batch_report(batch_id: str) -> Report
    def generate_summary_report(start_date: date, end_date: date) -> Report
    def export_to_pdf(report: Report) -> bytes
    def export_to_csv(data: Any) -> str
    def calculate_analytics(batches: List[Batch]) -> Analytics
```

**Report Types**:

1. **Batch Completion Report**:
   - Batch metadata (substrate, variety, dates)
   - Final yield vs. predicted yield
   - Environmental summary (avg temp, humidity, CO2)
   - Growth stage durations
   - Contamination events (if any)
   - Image timeline

2. **Summary Report** (multi-batch):
   - Total yield across batches
   - Average yield by substrate type
   - Average yield by variety
   - Contamination rate
   - Environmental patterns correlated with high yield
   - Month-over-month comparisons

**Analytics Calculations**:
- Yield per kg of substrate
- Days from inoculation to harvest
- Contamination rate (% of batches affected)
- Optimal environmental ranges (based on high-yield batches)

## Data Models

### EnvironmentalReading
```python
{
    "id": "uuid",
    "batch_id": "uuid",
    "timestamp": "datetime",
    "temperature": "float",      # Celsius
    "humidity": "float",         # Percentage (0-100)
    "co2": "int",                # PPM
    "light": "int",              # Lux
    "sensor_health": "string"    # "ok", "degraded", "failed"
}
```

### ImageMetadata
```python
{
    "id": "uuid",
    "batch_id": "uuid",
    "timestamp": "datetime",
    "file_path": "string",
    "resolution": "tuple",       # (width, height)
    "file_size": "int",          # Bytes
    "exposure_settings": "dict", # ISO, shutter speed, etc.
    "processed": "bool"          # Has ML analysis been run?
}
```

### ContaminationResult
```python
{
    "id": "uuid",
    "image_id": "uuid",
    "batch_id": "uuid",
    "timestamp": "datetime",
    "detected": "bool",
    "contamination_types": "list[string]",  # ["bacterial", "fungal_green", ...]
    "confidence": "float",                   # 0.0-1.0
    "severity": "string",                    # "low", "medium", "high", "critical"
    "bounding_boxes": "list[dict]",          # [{"x": int, "y": int, "w": int, "h": int, "type": str}]
    "model_version": "string"                # e.g., "contamination_v2.3"
}
```

### GrowthStage
```python
{
    "id": "uuid",
    "batch_id": "uuid",
    "timestamp": "datetime",
    "stage": "string",                    # "colonization", "pinning", "fruiting", "harvest"
    "colonization_percentage": "float",   # 0-100, null if not colonization stage
    "pin_count": "int",                   # null if not pinning stage
    "avg_fruiting_body_size": "float",    # mm, null if not fruiting stage
    "days_in_stage": "int",
    "estimated_days_to_next": "int",
    "model_version": "string"
}
```

### YieldPrediction
```python
{
    "id": "uuid",
    "batch_id": "uuid",
    "timestamp": "datetime",
    "predicted_yield_kg": "float",
    "confidence_interval_lower": "float",  # 90% CI lower bound
    "confidence_interval_upper": "float",  # 90% CI upper bound
    "confidence_score": "float",           # 0.0-1.0
    "features_used": "dict",               # Feature values used for prediction
    "model_version": "string"
}
```

### HarvestWindow
```python
{
    "id": "uuid",
    "batch_id": "uuid",
    "timestamp": "datetime",
    "window_start": "datetime",
    "window_end": "datetime",
    "readiness_score": "float",            # 0.0-1.0, current readiness
    "quality_degradation_rate": "float",   # % per hour after window_end
    "model_version": "string"
}
```

### Alert
```python
{
    "id": "uuid",
    "batch_id": "uuid",                    # null for system-wide alerts
    "timestamp": "datetime",
    "alert_type": "string",                # "environmental", "contamination", "harvest", "system_health", "forecast"
    "severity": "string",                  # "low", "medium", "high", "critical"
    "title": "string",                     # Short title
    "message": "string",                   # Detailed message
    "recommended_action": "string",        # What grower should do
    "acknowledged": "bool",
    "acknowledged_at": "datetime",
    "acknowledged_by": "string",           # user_id
    "escalated": "bool",
    "escalated_at": "datetime"
}
```

### Batch
```python
{
    "id": "uuid",
    "created_at": "datetime",
    "substrate_type": "string",            # "paddy_straw", "sugarcane_bagasse", "cotton_seed_hull"
    "mushroom_variety": "string",          # "oyster", "button", "shiitake", "milky"
    "inoculation_date": "date",
    "current_stage": "string",             # "colonization", "pinning", "fruiting", "harvest", "completed"
    "stage_updated_at": "datetime",
    "predicted_yield_kg": "float",
    "predicted_harvest_date": "date",
    "actual_yield_kg": "float",            # null until completed
    "actual_harvest_date": "date",         # null until completed
    "status": "string",                    # "active", "completed", "terminated"
    "notes": "string"                      # Grower notes
}
```

### SystemHealth
```python
{
    "timestamp": "datetime",
    "cpu_temperature": "float",            # Celsius
    "cpu_usage": "float",                  # Percentage
    "memory_usage": "float",               # Percentage
    "storage_usage": "float",              # Percentage
    "battery_level": "float",              # Percentage, null if on mains
    "power_source": "string",              # "mains", "battery"
    "sensor_status": "dict",               # {"temp": "ok", "humidity": "ok", "co2": "failed", ...}
    "camera_status": "string",             # "ok", "degraded", "failed"
    "connectivity_status": "string"        # "online", "offline"
}
```

### ModelUpdate
```python
{
    "id": "uuid",
    "model_name": "string",                # "contamination_detector", "growth_classifier", etc.
    "version": "string",                   # "v2.3"
    "release_date": "datetime",
    "file_url": "string",                  # Cloud storage URL
    "file_size": "int",                    # Bytes
    "checksum": "string",                  # SHA256
    "changelog": "string",                 # What's new
    "min_accuracy": "float",               # Minimum accuracy threshold for deployment
    "deployed": "bool",
    "deployed_at": "datetime"
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property Reflection

After analyzing all acceptance criteria, several patterns of redundancy emerged:

1. **Accuracy Properties (1.1-1.3, 2.2-2.4, 4.2, 5.3)**: Each sensor/model accuracy requirement can be tested with a single property pattern: "For any test input, error is within specified bounds"

2. **Data Presence Properties**: Many criteria just verify that data fields exist (timestamps, classifications, metrics). These can be combined into comprehensive data model validation properties.

3. **Substrate/Variety Adjustment Properties (8.5, 20.6-20.8)**: Multiple criteria verify that predictions differ based on substrate/variety. These can be consolidated into properties that verify prediction functions are sensitive to these parameters.

4. **Alert Generation Properties**: Many criteria verify alerts are generated for specific conditions. These follow a common pattern and can be grouped by alert type.

5. **Time-based Properties (1.6, 1.7, 2.5, 3.3, etc.)**: Multiple criteria verify operations complete within time bounds. These share a common testing pattern.

6. **Storage/Retrieval Properties (1.8, 3.8, 5.8, 12.8, 18.8)**: Multiple criteria verify data persistence and retrieval. These can be consolidated into round-trip properties.

After reflection, we'll focus on unique, high-value properties that provide comprehensive coverage without redundancy.

### Core Correctness Properties

**Property 1: Sensor Reading Accuracy Bounds**
*For any* sensor reading compared against a calibrated reference, the measurement error SHALL be within the specified accuracy bounds (temperature: ±0.5°C, humidity: ±3%, CO2: ±50 ppm)
**Validates: Requirements 1.1, 1.2, 1.3**

**Property 2: Sensor Reading Completeness**
*For any* environmental reading collected by the system, it SHALL include timestamp with millisecond precision, temperature, humidity, CO2, and light measurements
**Validates: Requirements 1.4, 1.5**

**Property 3: Sensor Sampling Frequency**
*For any* sequence of consecutive sensor readings, the time interval between readings SHALL be at most 5 minutes (300 seconds)
**Validates: Requirements 1.6**

**Property 4: Environmental Alert Timeliness**
*For any* sensor reading that exceeds configured safety thresholds, an alert SHALL be generated within 60 seconds
**Validates: Requirements 1.7**

**Property 5: Data Retention Period**
*For any* environmental reading from up to 90 days ago, it SHALL be retrievable from local storage
**Validates: Requirements 1.8**

**Property 6: Contamination Detection Accuracy**
*For any* labeled test image in the validation dataset, the contamination detector SHALL achieve minimum accuracy of 85% for bacterial contamination, 85% for mold contamination, and 80% for pest detection
**Validates: Requirements 2.2, 2.3, 2.4**

**Property 7: Contamination Analysis Completeness**
*For any* image analyzed for contamination, the result SHALL include detection status, contamination types (if detected), confidence score, severity level, and bounding boxes for contaminated regions
**Validates: Requirements 2.1, 2.6, 2.7, 2.8**

**Property 8: Contamination Alert Timeliness**
*For any* contamination event detected, an alert SHALL be generated within 2 minutes
**Validates: Requirements 2.5**

**Property 9: Growth Stage Classification Completeness**
*For any* batch with captured images, the system SHALL identify the current growth stage (colonization, pinning, fruiting, or harvest) and provide stage-appropriate metrics (colonization % for colonization stage, pin count for pinning stage, fruiting body size for fruiting stage)
**Validates: Requirements 3.1, 3.5, 3.6, 3.7**

**Property 10: Growth Stage Transition Logging**
*For any* batch, all growth stage transitions SHALL be logged with timestamps, and the complete transition history SHALL be retrievable
**Validates: Requirements 3.8**

**Property 11: Growth Stage Update Timeliness**
*For any* growth stage transition detected, the batch status SHALL be updated within 1 hour
**Validates: Requirements 3.3**

**Property 12: Harvest Window Prediction Presence**
*For any* batch entering the fruiting stage, a harvest window prediction SHALL be generated including window start time, window end time, and quality degradation rate
**Validates: Requirements 4.1, 4.7**

**Property 13: Harvest Window Prediction Accuracy**
*For any* completed batch with recorded optimal harvest time, the predicted harvest window SHALL include the actual optimal time within ±6 hours
**Validates: Requirements 4.2**

**Property 14: Harvest Alert Timeliness**
*For any* batch with a predicted harvest window, a readiness alert SHALL be generated 24 hours (±1 hour) before the window opens, and an urgent alert SHALL be generated when the window opens
**Validates: Requirements 4.3, 4.4**

**Property 15: Harvest Prediction Sensitivity**
*For any* two batches with identical conditions except mushroom variety, the predicted harvest windows SHALL differ according to variety-specific maturity timelines
**Validates: Requirements 4.5**

**Property 16: Yield Forecast Presence**
*For any* batch reaching the pinning stage, an initial yield forecast SHALL be generated including predicted yield in kg, confidence interval bounds, and confidence score
**Validates: Requirements 5.1, 5.4**

**Property 17: Yield Forecast Update Frequency**
*For any* batch in the fruiting stage, yield forecasts SHALL be updated at least once per 24-hour period
**Validates: Requirements 5.2**

**Property 18: Yield Forecast Accuracy**
*For any* completed batch with recorded actual yield, the final yield forecast SHALL be within ±15% of the actual yield
**Validates: Requirements 5.3**

**Property 19: Yield Forecast Substrate Sensitivity**
*For any* two batches with identical conditions except substrate type, the predicted yields SHALL differ according to substrate-specific yield characteristics
**Validates: Requirements 5.5, 8.5**

**Property 20: Yield Forecast Contamination Adjustment**
*For any* two otherwise identical batches where one has detected contamination, the contaminated batch SHALL have a lower yield forecast
**Validates: Requirements 5.7**

**Property 21: Yield Data Persistence**
*For any* completed batch, the actual yield data SHALL be stored and retrievable for historical analysis
**Validates: Requirements 5.8**

**Property 22: Offline ML Inference**
*For any* ML inference operation (contamination detection, growth stage classification, yield forecasting, harvest prediction), it SHALL complete successfully without network connectivity
**Validates: Requirements 6.1**

**Property 23: Offline Data Storage**
*For any* sensor reading or image captured while network is unavailable, it SHALL be stored locally and retrievable
**Validates: Requirements 6.2**

**Property 24: Offline Alert Generation**
*For any* alert condition occurring while network is unavailable, the alert SHALL be generated and displayed locally
**Validates: Requirements 6.3**

**Property 25: Offline UI Functionality**
*For any* user interface endpoint, it SHALL respond successfully when network is unavailable
**Validates: Requirements 6.4**

**Property 26: Automatic Sync Trigger**
*For any* transition from offline to online state, a data sync operation SHALL be automatically initiated within 5 minutes
**Validates: Requirements 6.5**

**Property 27: Sync Retry with Exponential Backoff**
*For any* failed sync operation, retry attempts SHALL follow exponential backoff pattern (delays: 1min, 2min, 4min, 8min, ...)
**Validates: Requirements 6.8**

**Property 28: Power Source Indication**
*For any* system state, the current power source (mains or battery) and battery level (if on battery) SHALL be available and accurate
**Validates: Requirements 7.5**

**Property 29: Data Persistence Frequency**
*For any* 5-minute time window during normal operation, at least one data persistence operation SHALL occur to save in-memory data
**Validates: Requirements 7.3**

**Property 30: Low Battery Alert**
*For any* system state where battery level drops below 20%, a low-power alert SHALL be generated
**Validates: Requirements 7.6**

**Property 31: Low Power Mode Activation**
*For any* system state where battery level drops below 10%, the system SHALL enter low-power mode and reduce sensor sampling frequency
**Validates: Requirements 7.7, 7.8**

**Property 32: Batch Unique Identifiers**
*For any* two distinct batches created by the system, their identifiers SHALL be different
**Validates: Requirements 12.3**

**Property 33: Batch Independent Tracking**
*For any* two concurrent batches, growth stage, alerts, and yield forecasts SHALL be tracked independently (changes to one batch SHALL NOT affect the other)
**Validates: Requirements 12.4, 12.5, 12.6**

**Property 34: Batch Data Persistence**
*For any* batch marked as completed, all historical data (sensor readings, images, alerts, predictions) SHALL remain retrievable
**Validates: Requirements 12.8**

**Property 35: Data Anonymization**
*For any* data uploaded during sync, personally identifiable information (grower name, location coordinates, contact info) SHALL be removed or pseudonymized
**Validates: Requirements 13.1, 14.5**

**Property 36: Model Validation Before Deployment**
*For any* new ML model downloaded during sync, it SHALL be validated against a test dataset before being deployed for inference
**Validates: Requirements 13.3**

**Property 37: Model Fallback Preservation**
*For any* new ML model deployed, the previous model version SHALL be retained and available for rollback
**Validates: Requirements 13.4**

**Property 38: Model Performance Regression Detection**
*For any* deployed ML model that performs below the minimum accuracy threshold on validation data, the system SHALL automatically revert to the previous model version
**Validates: Requirements 13.5**

**Property 39: Data Encryption at Rest**
*For any* data file stored on the edge device, it SHALL be encrypted using SQLCipher or equivalent encryption
**Validates: Requirements 14.1**

**Property 40: Data Encryption in Transit**
*For any* data transmitted during sync, it SHALL be encrypted using TLS 1.2 or higher
**Validates: Requirements 14.2**

**Property 41: Password Minimum Length**
*For any* password set by a grower, if it is less than 8 characters, it SHALL be rejected
**Validates: Requirements 14.4**

**Property 42: Image Capture Frequency**
*For any* 24-hour period during active batch monitoring, at least 4 images SHALL be captured
**Validates: Requirements 11.1**

**Property 43: Image Resolution Minimum**
*For any* image captured by the vision system, the resolution SHALL be at least 5 megapixels (2592x1944 or equivalent)
**Validates: Requirements 11.2**

**Property 44: Image Storage and Retrieval**
*For any* image captured, it SHALL be stored locally and retrievable via its metadata identifier
**Validates: Requirements 11.4**

**Property 45: Image Analysis Timeliness**
*For any* captured image, ML analysis SHALL complete within 5 minutes of capture
**Validates: Requirements 11.6**

**Property 46: Storage Cleanup Trigger**
*For any* system state where storage capacity reaches 80%, the oldest images (after sync if online) SHALL be deleted to free space
**Validates: Requirements 11.5**

**Property 47: Alert Deduplication**
*For any* two alerts of the same type for the same batch within a 6-hour window, only the first alert SHALL be delivered (subsequent duplicates SHALL be suppressed)
**Validates: Requirements 18.6**

**Property 48: Alert Escalation**
*For any* alert that remains unacknowledged for 24 hours, its severity level SHALL be escalated by one level
**Validates: Requirements 18.7**

**Property 49: Alert History Retention**
*For any* alert from up to 90 days ago, it SHALL be retrievable from the alert history log
**Validates: Requirements 18.8**

**Property 50: Sensor Health Check Frequency**
*For any* 15-minute time window during normal operation, at least one sensor connectivity check SHALL be performed
**Validates: Requirements 19.2**

**Property 51: Camera Health Check Frequency**
*For any* 60-minute time window during normal operation, at least one camera functionality check SHALL be performed
**Validates: Requirements 19.3**

**Property 52: Sensor Failure Alert**
*For any* sensor that fails a connectivity check, a system health alert SHALL be generated
**Validates: Requirements 19.4**

**Property 53: Camera Failure Alert**
*For any* camera that fails a functionality check, a system health alert SHALL be generated
**Validates: Requirements 19.5**

**Property 54: CPU Temperature Monitoring**
*For any* system state, the edge device CPU temperature SHALL be monitored and available in system health status
**Validates: Requirements 19.6**

**Property 55: Overheating Alert**
*For any* system state where CPU temperature exceeds 75°C, an overheating alert SHALL be generated
**Validates: Requirements 19.7**

**Property 56: System Error Logging**
*For any* system error or exception, it SHALL be logged with timestamp, error type, and stack trace
**Validates: Requirements 19.9**

**Property 57: Variety-Specific Growth Duration**
*For any* two batches with identical conditions except mushroom variety, the estimated growth stage durations SHALL differ according to variety-specific growth rates
**Validates: Requirements 20.6**

**Property 58: Variety-Specific Yield Characteristics**
*For any* two batches with identical conditions except mushroom variety, the yield forecasts SHALL differ according to variety-specific yield characteristics
**Validates: Requirements 20.8**

**Property 59: Variety-Specific Environmental Parameters**
*For any* mushroom variety, the system SHALL provide variety-specific optimal environmental parameters (temperature, humidity, CO2 ranges) that differ from other varieties
**Validates: Requirements 20.9**

**Property 60: Report Generation Completeness**
*For any* completed batch, the generated report SHALL include actual yield, predicted yield, growth duration, environmental summary, and contamination events (if any)
**Validates: Requirements 17.1**

**Property 61: Substrate Yield Analytics**
*For any* set of completed batches, the system SHALL calculate and provide average yield per substrate type
**Validates: Requirements 17.2**

**Property 62: Report Export Formats**
*For any* generated report, it SHALL be exportable in both PDF and CSV formats, and the exported files SHALL be valid and parseable
**Validates: Requirements 17.6, 17.7**

**Property 63: Offline Report Access**
*For any* report generation or viewing operation, it SHALL complete successfully without network connectivity
**Validates: Requirements 17.8**

## Error Handling

### Error Categories

1. **Hardware Errors**
   - Sensor communication failure
   - Camera malfunction
   - Storage device failure
   - Power supply issues

2. **Data Errors**
   - Invalid sensor readings (out of physical bounds)
   - Corrupted image files
   - Database corruption
   - Missing or incomplete data

3. **ML Errors**
   - Model file corruption
   - Inference timeout
   - Out-of-memory during inference
   - Invalid model output format

4. **Network Errors**
   - Sync failure (timeout, connection refused)
   - Model download failure
   - Authentication failure

5. **User Errors**
   - Invalid batch parameters
   - Invalid configuration values
   - Authentication failure

### Error Handling Strategies

**Sensor Communication Failure**:
- Retry 3 times with 1-second delay
- If all retries fail, use last valid reading (if <15 minutes old)
- Generate system health alert
- Log error with sensor ID and error details
- Continue operation with degraded data

**Camera Malfunction**:
- Retry capture 3 times
- If all retries fail, skip this capture cycle
- Generate system health alert
- Log error with camera details
- Continue operation without visual analysis for this cycle

**Invalid Sensor Readings**:
- Validate against physical bounds (temp: -10 to 60°C, humidity: 0-100%, CO2: 0-5000ppm)
- If invalid, discard reading and retry
- If persistent (>3 consecutive invalid readings), generate sensor failure alert
- Use last valid reading as fallback

**ML Inference Failure**:
- Log error with model name, input details, error message
- Retry once
- If retry fails, skip this analysis cycle
- Generate system health alert if failures persist (>3 consecutive)
- Continue operation without ML insights for this cycle

**Storage Full**:
- Trigger emergency cleanup (delete oldest images beyond 30 days)
- Generate critical alert to grower
- Reduce image capture frequency to 2x daily
- Attempt sync to offload data (if online)

**Database Corruption**:
- Attempt automatic repair using SQLite integrity check
- If repair fails, restore from most recent backup
- Generate critical alert
- Log corruption details for diagnostics

**Sync Failure**:
- Retry with exponential backoff (1min, 2min, 4min, 8min, 16min, 32min, 1hr)
- Queue data locally for next sync attempt
- Generate low-priority alert if offline >24 hours
- Continue normal operation (offline-first design)

**Model Download Failure**:
- Retry with exponential backoff
- Continue using current model version
- Log failure for diagnostics
- Retry on next sync cycle

**Power Loss**:
- Persist all in-memory data immediately on battery switchover
- Continue normal operation on battery
- Generate low-power alert when battery <20%
- Enter low-power mode when battery <10%
- Graceful shutdown when battery <5% (save all data, close database cleanly)

**Authentication Failure**:
- Return 401 Unauthorized
- Log failed attempt with timestamp and IP
- Rate limit: max 5 attempts per 15 minutes per IP
- Temporary lockout after 5 failed attempts

### Error Recovery

**Automatic Recovery**:
- Sensor failures: Automatic retry with fallback to cached values
- ML failures: Skip cycle, retry on next cycle
- Storage issues: Automatic cleanup and compression
- Power issues: Automatic battery switchover and low-power mode
- Model issues: Automatic rollback to previous version

**Manual Recovery**:
- Database corruption: Restore from backup (grower confirmation required)
- Complete system failure: Reboot and run diagnostics
- Persistent sensor failure: Grower must check physical connections
- Storage device failure: Grower must replace storage media

**Graceful Degradation**:
- If camera fails: Continue with sensor-only monitoring
- If CO2 sensor fails: Continue with temp/humidity monitoring
- If ML models fail: Continue with sensor monitoring and basic alerts
- If storage nearly full: Reduce capture frequency, prioritize critical data
- If battery low: Reduce sampling frequency, disable non-critical features

## Testing Strategy

### Dual Testing Approach

This system requires both unit testing and property-based testing for comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property tests**: Verify universal properties across all inputs
- Both approaches are complementary and necessary

### Unit Testing

Unit tests focus on:
- Specific examples demonstrating correct behavior
- Edge cases (empty inputs, boundary values, extreme conditions)
- Error conditions (sensor failures, invalid data, network errors)
- Integration points between components
- UI endpoints and API contracts

**Example Unit Tests**:
- Test sensor reader with specific temperature value (25.5°C)
- Test contamination detector with known contaminated image
- Test batch creation with valid parameters
- Test alert generation for specific threshold violation
- Test database query with specific date range
- Test authentication with valid/invalid credentials
- Test report generation for completed batch
- Test storage cleanup at exactly 80% capacity

### Property-Based Testing

Property tests focus on:
- Universal properties that hold for all inputs
- Comprehensive input coverage through randomization
- Invariants that must be preserved
- Round-trip properties (serialize/deserialize, encode/decode)
- Metamorphic properties (relationships between inputs/outputs)

**Property Testing Configuration**:
- Framework: Hypothesis (Python) or fast-check (JavaScript/TypeScript)
- Minimum 100 iterations per property test
- Each test tagged with: **Feature: mushroom-growth-monitoring, Property {N}: {property_text}**
- Shrinking enabled to find minimal failing examples
- Seed-based reproducibility for debugging

**Example Property Tests**:
- Property 1: For any sensor reading, accuracy is within bounds
- Property 6: For any test image, contamination detection accuracy ≥85%
- Property 18: For any completed batch, yield forecast error ≤15%
- Property 22: For any ML operation with network disabled, inference succeeds
- Property 32: For any two batches, IDs are unique
- Property 39: For any stored data file, it is encrypted
- Property 47: For any duplicate alerts within 6 hours, only first is delivered

**Generators for Property Tests**:
- Environmental readings: Random temp (15-45°C), humidity (30-95%), CO2 (400-2000ppm)
- Images: Synthetic images with known contamination patterns
- Batches: Random substrate types, varieties, dates
- Timestamps: Random dates within valid ranges
- Sensor failures: Random failure patterns
- Network states: Random online/offline transitions

### Integration Testing

Integration tests verify:
- End-to-end workflows (batch creation → monitoring → harvest → completion)
- Component interactions (sensor → storage → ML → alert → UI)
- Data flow through the system
- Sync operations (offline → online transition)
- Model update workflow (download → validate → deploy)

### Performance Testing

Performance tests verify:
- ML inference time <2 seconds for contamination detection
- ML inference time <5 seconds for growth stage classification
- Image analysis completes within 5 minutes
- Sensor sampling maintains 5-minute intervals under load
- UI response time <500ms for dashboard queries
- Database queries complete within 1 second
- System operates with 10 concurrent batches without degradation

### Hardware Testing

Hardware tests verify (requires physical setup):
- Sensor accuracy against calibrated references
- Camera image quality and resolution
- Battery runtime (8+ hours)
- Power switchover time (<100ms)
- Operation in temperature range (15-45°C)
- Operation in humidity range (30-95%)
- Storage device reliability

### Security Testing

Security tests verify:
- Authentication prevents unauthorized access
- Password requirements enforced (8+ characters)
- Data encryption at rest (SQLCipher)
- Data encryption in transit (TLS 1.2+)
- Data anonymization before upload
- SQL injection prevention
- XSS prevention in web interface
- Rate limiting on authentication attempts

### Acceptance Testing

Acceptance tests verify:
- Setup wizard completes successfully
- Grower can create and monitor batches
- Alerts are delivered and actionable
- Reports are generated and useful
- System operates offline for extended periods
- System recovers from power loss
- UI is usable in Hindi and English
- System handles 10 concurrent batches

### Test Environment

**Development Environment**:
- Raspberry Pi 4 (4GB) or equivalent
- Test sensors (DHT22, MH-Z19, BH1750)
- USB camera (5MP minimum)
- Battery/UPS for power testing
- Local network for connectivity testing

**CI/CD Pipeline**:
- Automated unit tests on every commit
- Automated property tests on every commit
- Integration tests on pull requests
- Performance tests on release candidates
- Security scans on every build

**Test Data**:
- Synthetic sensor data covering full range
- Labeled image dataset (10,000+ images) for contamination detection
- Historical batch data (1,000+ batches) for yield forecasting
- Edge cases and anomalies for robustness testing

### Test Coverage Goals

- Unit test coverage: >80% of code lines
- Property test coverage: All 63 correctness properties
- Integration test coverage: All major workflows
- Edge case coverage: All error handling paths
- Performance test coverage: All time-critical operations
- Security test coverage: All authentication and data protection mechanisms

## Deployment and Operations

### Initial Deployment

1. **Hardware Assembly**:
   - Install Raspberry Pi 4 in protective enclosure
   - Connect sensors via I2C/SPI
   - Connect USB camera
   - Connect battery/UPS
   - Mount in cultivation area

2. **Software Installation**:
   - Flash SD card with OS image
   - Install system software package
   - Run setup wizard
   - Calibrate sensors
   - Configure network (WiFi/Ethernet)
   - Set language preference
   - Create grower account

3. **Verification**:
   - Verify sensor readings
   - Verify camera capture
   - Verify ML models loaded
   - Verify database initialized
   - Verify web interface accessible
   - Create test batch

### Ongoing Operations

**Daily**:
- System automatically captures images 4x daily
- System automatically samples sensors every 5 minutes
- System automatically runs ML analysis
- System automatically generates alerts as needed

**Weekly**:
- Grower reviews batch progress
- Grower acknowledges alerts
- System automatically syncs data (if online)

**Monthly**:
- Grower reviews reports and analytics
- System automatically downloads model updates (if available)
- System automatically performs database maintenance

**As Needed**:
- Grower creates new batches
- Grower completes harvested batches
- Grower adjusts alert thresholds
- Grower exports reports

### Maintenance

**Automated Maintenance**:
- Daily database backup
- Weekly database vacuum (optimize storage)
- Monthly log rotation
- Automatic storage cleanup at 80% capacity
- Automatic model updates during sync

**Manual Maintenance**:
- Quarterly sensor calibration check
- Quarterly camera lens cleaning
- Annual battery replacement (if degraded)
- As-needed SD card replacement (if failing)

### Monitoring

**System Self-Monitoring**:
- CPU temperature
- Memory usage
- Storage usage
- Battery level
- Sensor health
- Camera health
- Network connectivity

**Alerts for Operators**:
- Sensor failures
- Camera failures
- Storage nearly full
- Battery low
- System overheating
- Database errors

### Backup and Recovery

**Automated Backups**:
- Daily database backup to `/data/backups/`
- Keep last 7 daily backups
- Sync backups to cloud (if online)

**Manual Backup**:
- Grower can export all data to USB drive
- Includes database, images, configuration

**Recovery Procedures**:
- Database corruption: Restore from most recent backup
- SD card failure: Flash new card, restore from cloud backup
- Complete system failure: Replace hardware, restore from backup

### Updates

**Software Updates**:
- System software updates via package manager
- Requires manual trigger by grower
- Automatic backup before update
- Rollback capability if update fails

**ML Model Updates**:
- Automatic download during sync
- Automatic validation before deployment
- Automatic rollback if performance degrades
- Grower notified of new models

### Scaling

**Single Grower, Multiple Locations**:
- Deploy one edge device per location
- Each device operates independently
- Cloud aggregates data across locations (if online)
- Grower can view all locations from mobile app

**Multiple Growers (Future)**:
- Federated learning: Aggregate model improvements across growers
- Privacy-preserving: Only anonymized data shared
- Opt-in: Growers control data sharing preferences

## Future Enhancements

### Phase 2 Features

1. **Advanced Analytics**:
   - Predictive maintenance for equipment
   - Optimal substrate mixing recommendations
   - Market price integration for harvest timing
   - Carbon footprint tracking

2. **Automation Integration**:
   - Automatic humidity control (humidifier/dehumidifier)
   - Automatic temperature control (heater/cooler)
   - Automatic ventilation control (CO2 management)
   - Automatic watering systems

3. **Mobile App**:
   - Native iOS/Android apps
   - Push notifications
   - Remote monitoring
   - Multi-location management

4. **Community Features**:
   - Grower forums and knowledge sharing
   - Best practices database
   - Peer-to-peer support
   - Expert consultation marketplace

### Phase 3 Features

1. **Market Integration**:
   - Direct buyer connections
   - Price discovery
   - Logistics coordination
   - Quality certification

2. **Financial Services**:
   - Crop insurance integration
   - Lending based on yield predictions
   - Payment processing
   - Expense tracking

3. **Supply Chain**:
   - Substrate supplier marketplace
   - Equipment rental
   - Cold storage coordination
   - Transportation booking

4. **Advanced ML**:
   - Disease prediction (before visible symptoms)
   - Genetic strain optimization
   - Climate change adaptation
   - Pest outbreak prediction
   - DNA Barcoding
   - Genotyphe and phenotype prediction
   - Genome editing

## Conclusion
