# Requirements Document: AI-Powered Mushroom Growth Monitoring System

## Introduction

This document specifies requirements for an AI-powered growth monitoring system designed for semi-commercial mushroom cultivation in India. The system combines IoT environmental sensing, computer vision, and machine learning to provide real-time intelligence across the full mushroom growth lifecycle—from mycelium colonization through pinning, fruiting, and harvest. The system is optimized for Indian substrates, ambient conditions, and operates offline-first on edge hardware to serve growers without reliable internet or power infrastructure.

## Glossary

- **System**: The complete AI-powered mushroom growth monitoring platform including sensors, edge compute, ML models, and user interface
- **Edge_Device**: Local computing hardware that processes sensor data and runs ML models without requiring cloud connectivity
- **Growth_Stage**: Distinct phases in mushroom cultivation (colonization, pinning, fruiting, harvest)
- **Contamination_Event**: Presence of competing organisms (bacteria, mold, pests) that threaten crop viability
- **Environmental_Sensor**: IoT device measuring temperature, humidity, CO2, light, or other environmental parameters
- **Vision_System**: Camera-based component that captures images for ML analysis
- **ML_Model**: Machine learning model trained on Indian substrate data for detection, prediction, or classification tasks
- **Harvest_Window**: Optimal time period for harvesting mushrooms to maximize quality and shelf life
- **Substrate**: Growing medium for mushrooms (paddy straw, sugarcane bagasse, cotton seed hull)
- **Grower**: Semi-commercial mushroom farmer operating the system
- **Alert**: Notification to grower about detected issues or recommended actions
- **Yield_Forecast**: Predicted quantity of harvestable mushrooms
- **Data_Sync**: Process of uploading local data to cloud when connectivity is available

## Requirements

### Requirement 1: Environmental Monitoring

**User Story:** As a grower, I want continuous monitoring of environmental conditions, so that I can maintain optimal growing parameters throughout the cultivation cycle.

#### Acceptance Criteria

1. THE Environmental_Sensor SHALL measure temperature with ±0.5°C accuracy
2. THE Environmental_Sensor SHALL measure relative humidity with ±3% accuracy
3. THE Environmental_Sensor SHALL measure CO2 concentration with ±50 ppm accuracy
4. THE Environmental_Sensor SHALL measure light intensity in lux
5. WHEN sensor readings are collected, THE System SHALL timestamp each reading with millisecond precision
6. THE System SHALL record environmental data at minimum 5-minute intervals
7. WHEN environmental parameters exceed safe thresholds, THE System SHALL generate an Alert within 60 seconds
8. THE System SHALL store at least 90 days of environmental data locally on the Edge_Device

### Requirement 2: Contamination Detection

**User Story:** As a grower, I want early detection of contamination, so that I can take corrective action before significant crop loss occurs.

#### Acceptance Criteria

1. WHEN the Vision_System captures substrate images, THE ML_Model SHALL analyze them for contamination indicators
2. THE System SHALL detect bacterial contamination with minimum 85% accuracy
3. THE System SHALL detect mold contamination with minimum 85% accuracy
4. THE System SHALL detect pest presence with minimum 80% accuracy
5. WHEN a Contamination_Event is detected, THE System SHALL generate an Alert within 2 minutes
6. THE System SHALL classify contamination type (bacterial, fungal, pest)
7. THE System SHALL indicate contamination severity (low, medium, high, critical)
8. THE System SHALL provide visual highlighting of contaminated regions in captured images
9. THE ML_Model SHALL be trained on Indian substrate types (paddy straw, sugarcane bagasse, cotton seed hull)

### Requirement 3: Growth Stage Tracking

**User Story:** As a grower, I want automatic tracking of growth stages, so that I can understand crop progression and plan operations accordingly.

#### Acceptance Criteria

1. THE System SHALL identify the current Growth_Stage from visual analysis
2. THE System SHALL distinguish between colonization, pinning, fruiting, and harvest stages
3. WHEN Growth_Stage transitions occur, THE System SHALL update stage status within 1 hour
4. THE System SHALL estimate time remaining until next Growth_Stage transition
5. THE System SHALL track colonization percentage during the colonization stage
6. THE System SHALL count pin formation during the pinning stage
7. THE System SHALL measure fruiting body size during the fruiting stage
8. THE System SHALL maintain a historical log of Growth_Stage transitions for each cultivation batch

### Requirement 4: Harvest Window Prediction

**User Story:** As a grower, I want prediction of optimal harvest timing, so that I can maximize mushroom quality and minimize post-harvest losses.

#### Acceptance Criteria

1. WHEN mushrooms enter the fruiting stage, THE ML_Model SHALL predict the optimal Harvest_Window
2. THE System SHALL provide Harvest_Window predictions with ±6 hour accuracy
3. THE System SHALL generate harvest readiness Alerts 24 hours before the Harvest_Window opens
4. THE System SHALL generate urgent harvest Alerts when mushrooms reach peak maturity
5. THE System SHALL account for mushroom variety in Harvest_Window calculations
6. THE System SHALL adjust predictions based on current environmental conditions
7. THE System SHALL indicate expected quality degradation if harvest is delayed beyond the optimal window

### Requirement 5: Yield Forecasting

**User Story:** As a grower, I want accurate yield predictions, so that I can plan harvesting resources, storage, and market sales in advance.

**Note:** This is a PHASE 2 FEATURE requiring 6-12 months of ground truth data collection before deployment.

#### Acceptance Criteria

**Phase 1 (MVP - Months 1-3):**
1. Yield forecasting SHALL NOT be included in initial deployment
2. THE System SHALL operate in data collection mode:
   - System logs environmental data + growth stages
   - Farmer manually enters actual harvest weights
   - Minimum 50-100 bags tracked from pinning → harvest with weighed yields required for model training.
   - Also collect environmental data alongside monitoring.

**Phase 2 (Months 6-12):**
3. WHEN sufficient training data is collected, THE System SHALL deploy yield prediction model
4. THE ML_Model SHALL use LSTM/Transformer trained on time-series: environmental data + growth stage progression + historical yields
5. THE System SHALL predict yield in kilograms with ±15-20% accuracy target from pinning stage
6. THE System SHALL update Yield_Forecast daily as fruiting progresses
7. THE System SHALL provide confidence intervals for Yield_Forecast predictions
8. THE System SHALL account for substrate type in yield calculations
9. THE System SHALL account for environmental history in yield calculations
10. THE System SHALL adjust forecasts based on detected Contamination_Events

**Ground Truth Data Requirements:**
11. Harvest weight (kg) per bag SHALL be recorded within 1 hour of harvest
12. Substrate type, bag size, environmental history SHALL be linked to harvest data
13. Minimum dataset: 50 bags per species per substrate type

### Requirement 6: Offline-First Operation

**User Story:** As a grower in an area with unreliable internet, I want the system to function fully offline, so that I can monitor my crops without depending on connectivity.

#### Acceptance Criteria

1. THE Edge_Device SHALL run all ML_Models locally without requiring internet connectivity
2. THE Edge_Device SHALL store all sensor data locally when internet is unavailable
3. THE Edge_Device SHALL generate all Alerts locally without requiring cloud services
4. THE System SHALL provide full user interface functionality offline
5. WHEN internet connectivity becomes available, THE System SHALL perform Data_Sync automatically
6. THE Data_Sync process SHALL upload locally stored data to cloud storage
7. THE Data_Sync process SHALL download ML_Model updates when available
8. IF Data_Sync fails, THEN THE System SHALL retry with exponential backoff
9. THE System SHALL indicate current connectivity status to the Grower

### Requirement 7: Power Resilience

**User Story:** As a grower with unreliable power supply, I want the system to handle power interruptions gracefully, so that I don't lose critical data or monitoring capability.

#### Acceptance Criteria

1. THE Edge_Device SHALL operate on battery backup for minimum 8 hours
2. WHEN mains power is lost, THE System SHALL switch to battery power within 100 milliseconds
3. THE System SHALL save all in-memory data to persistent storage every 5 minutes
4. WHEN power is restored, THE System SHALL resume normal operation automatically
5. THE System SHALL indicate current power source (mains, battery) and battery level
6. WHEN battery level drops below 20%, THE System SHALL generate a low-power Alert
7. THE System SHALL enter low-power mode when battery drops below 10%
8. WHILE in low-power mode, THE System SHALL reduce sensor sampling frequency to conserve energy

### Requirement 8: Indian Substrate Optimization

**User Story:** As a grower using local Indian substrates, I want the system optimized for my materials, so that predictions and recommendations are accurate for my cultivation conditions.

#### Acceptance Criteria

1. THE ML_Model SHALL be trained on paddy straw substrate data
2. THE ML_Model SHALL be trained on sugarcane bagasse substrate data
3. THE ML_Model SHALL be trained on cotton seed hull substrate data
4. THE System SHALL allow Grower to specify substrate type for each cultivation batch
5. THE System SHALL adjust all predictions based on selected substrate type
6. THE System SHALL provide substrate-specific optimal environmental ranges
7. THE System SHALL account for substrate-specific contamination patterns
8. THE System SHALL account for substrate-specific yield characteristics

### Requirement 9: Indian Ambient Condition Adaptation

**User Story:** As a grower operating in Indian climate conditions, I want the system to account for local ambient conditions, so that recommendations are practical and achievable.

#### Acceptance Criteria

1. THE System SHALL operate in ambient temperatures from 15°C to 45°C
2. THE System SHALL operate in relative humidity from 30% to 95%
3. THE System SHALL account for seasonal variations in ambient conditions
4. THE System SHALL provide recommendations achievable with passive cooling methods
5. THE System SHALL provide recommendations achievable with low-cost humidification
6. THE System SHALL adjust optimal ranges based on local climate zone
7. WHEN ambient conditions make optimal parameters unachievable, THE System SHALL recommend best-effort alternatives

### Requirement 10: User Interface and Alerts

**User Story:** As a grower with limited technical expertise, I want a simple interface with clear alerts, so that I can understand system insights and take appropriate action.

#### Acceptance Criteria

1. THE System SHALL provide a visual dashboard showing current environmental conditions
2. THE System SHALL display current Growth_Stage with visual indicators
3. THE System SHALL display active Alerts with priority levels (low, medium, high, critical)
4. THE System SHALL provide Alert descriptions in simple language
5. THE System SHALL provide recommended actions for each Alert type
6. THE System SHALL support Hindi language interface
7. THE System SHALL support English language interface
8. THE System SHALL allow Grower to acknowledge and dismiss Alerts
9. THE System SHALL display historical trends for temperature, humidity, and CO2
10. THE System SHALL display Yield_Forecast and Harvest_Window predictions prominently
11. THE System SHALL provide image gallery showing crop progression over time

### Requirement 11: Image Capture and Analysis

**User Story:** As a grower, I want automated image capture and analysis, so that I can track visual crop development without manual photography.

#### Acceptance Criteria

1. THE Vision_System SHALL capture images with adaptive frequency based on growth stage:
   - Colonization phase (Day 1-15): 4x daily (every 6 hours)
   - Pinning/fruiting phase (Day 15+): 8-12x daily (every 2-3 hours)
   - Frequency adjustment triggered automatically by growth stage classification model
2. THE Vision_System SHALL capture images with minimum 8 megapixel resolution
3. THE Vision_System SHALL adjust exposure automatically for varying light conditions
4. THE Vision_System SHALL store images locally on the Edge_Device
5. WHEN storage capacity reaches 80%, THE System SHALL delete oldest images after Data_Sync
6. THE ML_Model SHALL process each captured image within 5 minutes
7. THE System SHALL extract growth metrics from images (colonization %, pin count, fruiting body size)
8. THE System SHALL detect visual anomalies beyond contamination (substrate drying, uneven growth)

### Requirement 12: Batch Management

**User Story:** As a grower managing multiple cultivation batches, I want to track each batch independently, so that I can manage different growth stages simultaneously.

#### Acceptance Criteria

1. THE System SHALL support monitoring of at least 10 concurrent cultivation batches
2. THE Grower SHALL be able to create a new batch with substrate type, start date, and mushroom variety
3. THE System SHALL assign a unique identifier to each batch
4. THE System SHALL track Growth_Stage independently for each batch
5. THE System SHALL generate Alerts specific to each batch
6. THE System SHALL provide Yield_Forecast for each batch independently
7. THE System SHALL allow Grower to mark a batch as harvested or terminated
8. THE System SHALL maintain historical records for completed batches
9. THE System SHALL allow Grower to view and compare metrics across batches

### Requirement 13: Model Updates and Learning

**User Story:** As a grower, I want the system to improve over time, so that predictions become more accurate as more data is collected.

#### Acceptance Criteria

1. WHEN Data_Sync occurs, THE System SHALL upload anonymized cultivation data to cloud storage
2. THE System SHALL implement controlled deployment with validation and rollback:
   - **Update trigger:**
     - Manual approval via dashboard (farmer opts in)
     - Automatic trigger if contamination detection accuracy drops below 80% for 3 consecutive days
   - **Validation process:**
     - New model runs in shadow mode for 24 hours alongside current model
     - If predictions diverge >20%, flag for manual review before deployment
     - Developer approval required before production deployment
   - **Rollback conditions:**
     - If contamination false positive rate exceeds 15% within 48 hours of new model deployment
     - If system crashes or inference time exceeds 10 minutes
     - Manual rollback option always available via dashboard
   - **Version management:**
     - Store last 3 model versions locally (compressed, ~150-600MB total)
     - Each version tagged with accuracy metrics, deployment date, validation results
3. THE System SHALL allow Grower to provide feedback on prediction accuracy
4. THE System SHALL incorporate Grower feedback into model improvement pipeline
5. THE System SHALL notify Grower when new ML_Models are available

### Requirement 14: Data Privacy and Security

**User Story:** As a grower, I want my cultivation data to remain private and secure, so that my business information is protected.

#### Acceptance Criteria

1. THE System SHALL encrypt all data stored on the Edge_Device
2. THE System SHALL encrypt all data transmitted during Data_Sync
3. THE System SHALL require authentication for user interface access
4. THE System SHALL support password protection with minimum 8 character length
5. THE System SHALL anonymize data before uploading to cloud during Data_Sync
6. THE System SHALL not share identifiable grower information with third parties
7. THE Grower SHALL be able to delete all local data from the Edge_Device
8. THE Grower SHALL be able to request deletion of cloud data

### Requirement 15: Hardware Specifications

**User Story:** As a grower with budget constraints, I want the system to run on affordable edge hardware, so that the solution remains economically viable.

#### Acceptance Criteria

1. THE Edge_Device SHALL run on hardware costing less than ₹15,000 (approximately $180 USD)
2. THE complete hardware bill of materials SHALL be:
   - **Edge compute:** Raspberry Pi 4 (4GB RAM) - ₹5,500
   - **Camera:** Pi Camera Module v2 (8MP) - ₹2,200
   - **Temp/Humidity:** DHT22 (single unit) - ₹300
   - **CO2 sensor:** MH-Z19B - ₹1,500
   - **Connectivity:** SIM800L GSM module + antenna - ₹700
   - **Power:** 5V 3A adapter + 12V 5Ah battery + buck converter - ₹2,500
   - **Storage:** SanDisk 64GB microSD (Class 10) - ₹600
   - **Enclosure:** Weatherproof ABS plastic case (IP54 rating minimum) - ₹800
   - **TOTAL:** ₹14,100 (under ₹15k budget)
3. THE Edge_Device SHALL have minimum 4GB RAM
4. THE Edge_Device SHALL have minimum 64GB storage
5. THE Edge_Device SHALL support USB camera connectivity
6. THE Edge_Device SHALL support I2C or SPI sensor connectivity
7. THE Edge_Device SHALL have WiFi connectivity capability
8. THE Edge_Device SHALL have Ethernet connectivity capability

**Excluded from MVP (Phase 2 additions if needed):**
- Soil moisture sensor (₹150) - substrate moisture monitoring
- Second DHT22 for redundancy (₹300)
- Raspberry Pi 4 8GB upgrade (+₹2,500) - only if running heavier Vision Transformer models

**Recurring Costs:**
- GSM SIM card (prepaid): ₹200-300/month for ~500 SMS alerts
- Cloud storage (Firebase free tier initially): ₹0-500/month
- Electricity (Pi + sensors, 24/7 operation): ~₹50-100/month

### Requirement 16: Calibration and Setup

**User Story:** As a grower setting up the system for the first time, I want a simple calibration process, so that I can start monitoring quickly without technical assistance.

#### Acceptance Criteria

1. THE System SHALL provide a guided setup wizard on first boot
2. THE setup wizard SHALL guide sensor placement and verification
3. THE setup wizard SHALL guide camera positioning and focus verification
4. THE System SHALL perform automatic sensor calibration
5. THE System SHALL verify sensor readings are within expected ranges
6. THE System SHALL capture reference images during setup for baseline comparison
7. THE System SHALL allow Grower to configure Alert preferences during setup
8. THE System SHALL allow Grower to select language preference during setup
9. THE setup wizard SHALL complete in less than 30 minutes

### Requirement 17: Reporting and Analytics

**User Story:** As a grower, I want historical reports and analytics, so that I can learn from past batches and improve my cultivation practices.

#### Acceptance Criteria

1. THE System SHALL generate batch completion reports showing yield, growth duration, and environmental summary
2. THE System SHALL calculate average yield per substrate type across all batches
3. THE System SHALL identify environmental patterns correlated with high yield
4. THE System SHALL identify environmental patterns correlated with contamination
5. THE System SHALL provide month-over-month yield comparisons
6. THE System SHALL export reports in PDF format
7. THE System SHALL export raw data in CSV format
8. THE Grower SHALL be able to view reports offline on the Edge_Device

### Requirement 18: Alert Delivery Mechanisms

**User Story:** As a grower who may not always be near the system, I want multiple alert delivery options, so that I can respond to critical issues promptly.

#### Acceptance Criteria

1. THE System SHALL display Alerts on the local user interface
2. THE System SHALL implement defined escalation ladder based on alert type:
   - **Contamination alerts (urgent):**
     - Dashboard alert (immediate)
     - Push notification (if no acknowledgment in 5 minutes)
     - SMS to primary contact (if no acknowledgment in 15 minutes)
     - SMS to secondary contact (if no acknowledgment in 30 minutes)
     - Email with contamination images (if no acknowledgment in 1 hour)
   - **Harvest window alerts (standard):**
     - Dashboard + push notification (immediate)
     - SMS (if no acknowledgment in 2 hours)
   - **Environmental alerts (low priority):**
     - Dashboard notification (immediate)
     - SMS summary at end of day if unresolved
3. ALL alerts SHALL require farmer acknowledgment via app/SMS
4. THE System SHALL track response times for system health monitoring
5. THE System SHALL support configurable Alert thresholds per Alert type
6. THE System SHALL allow Grower to enable or disable specific Alert types
7. THE System SHALL not send duplicate Alerts for the same issue within 6 hours
8. THE System SHALL maintain an Alert history log for at least 90 days

### Requirement 19: System Health Monitoring

**User Story:** As a grower, I want to know if the monitoring system itself is functioning correctly, so that I can trust the data and alerts I receive.

#### Acceptance Criteria

1. THE System SHALL perform self-diagnostics on startup
2. THE System SHALL verify sensor connectivity every 15 minutes
3. THE System SHALL verify camera functionality every hour
4. WHEN a sensor fails connectivity check, THE System SHALL generate a system health Alert
5. WHEN the camera fails functionality check, THE System SHALL generate a system health Alert
6. THE System SHALL monitor Edge_Device CPU temperature
7. WHEN Edge_Device temperature exceeds 75°C, THE System SHALL generate an overheating Alert
8. THE System SHALL display system health status on the dashboard
9. THE System SHALL log all system errors for troubleshooting

### Requirement 20: Mushroom Variety Support

**User Story:** As a grower cultivating different mushroom varieties, I want variety-specific monitoring, so that predictions account for different growth characteristics.

#### Acceptance Criteria

1. THE System SHALL support oyster mushroom variety monitoring
2. THE System SHALL support button mushroom variety monitoring
3. THE System SHALL support shiitake mushroom variety monitoring
4. THE System SHALL support milky mushroom variety monitoring
5. THE Grower SHALL specify mushroom variety when creating a new batch
6. THE System SHALL adjust Growth_Stage duration estimates based on variety
7. THE System SHALL adjust Harvest_Window predictions based on variety
8. THE System SHALL adjust Yield_Forecast based on variety
9. THE System SHALL provide variety-specific optimal environmental parameters

### Requirement 21: Local Storage Management

**User Story:** As a grower with limited storage capacity, I want intelligent storage management, so that the system continues operating without manual intervention.

#### Acceptance Criteria

**Storage Allocation (64GB microSD card):**
1. THE System SHALL allocate storage as follows:
   - OS + system files: 8GB (Raspberry Pi OS Lite)
   - ML models (3 version rollback): 1GB (compressed TFLite models)
   - Image archive (90-day retention): 3GB (4-8x daily at ~20MB/day)
   - Logs + metrics (30-day retention): 500MB (environmental readings, alerts, system events)
   - Free space buffer: 51.5GB (for temporary expansion, video captures, edge cases)

**Cleanup Logic (automatic):**
2. WHEN storage reaches 80% usage (51.2GB used), THE System SHALL execute cleanup:
   - Action 1: Delete oldest 10 days of images (non-critical images only)
   - Action 2: Compress logs older than 7 days (gzip compression)
   - Action 3: If still >80%, delete oldest 20 days of images
3. Images flagged as "contamination detected" SHALL NEVER be auto-deleted
4. BEFORE deletion, THE System SHALL attempt cloud sync if connectivity available

**Manual Overrides:**
5. THE Grower SHALL be able to mark specific batches as "archive permanently" (prevents auto-cleanup)
6. THE System SHALL provide export function to download specific date ranges or batches before cleanup
7. THE System SHALL display storage health in dashboard: "45% used, 35GB free, next cleanup in 22 days"

**Failure Handling:**
8. WHEN SD card reaches 95% full, THE System SHALL:
   - Stop new image captures
   - Keep environmental logging active
   - Send alert to farmer: "Storage critical - system will stop image capture in 24 hours"
9. WHEN emergency cleanup is needed, THE System SHALL delete all non-contamination images older than 30 days

### Requirement 22: Farmer Feedback Loop and Continuous Model Improvement

**User Story:** As a grower, I want to provide feedback on system predictions, so that the model learns from real-world conditions and improves over time.

#### Acceptance Criteria

**Feedback Mechanism:**
1. EACH contamination/harvest/environmental alert SHALL include feedback prompt:
   - "Was this alert correct?" [Yes / No / Not Sure]
   - If "No": "What did you observe instead?" [Healthy mycelium / Different contamination type / Other issue]
2. Feedback SHALL be submittable via app (1-click) or SMS (reply with Y/N)

**Manual Labeling Interface:**
3. THE System SHALL provide "Review Images" section in dashboard showing recent captures
4. THE Grower SHALL be able to manually label images:
   - Label types: Clean mycelium, Trichoderma (green mold), bacterial blotch, cobweb mold, pin mold, healthy fruiting
5. Labeled images SHALL be stored separately with farmer annotation

**Model Retraining Pipeline:**
6. Farmer corrections SHALL accumulate in local database
7. WHEN 50+ new labeled images are collected, THE System SHALL flag for retraining
8. Developer SHALL download labeled dataset from all deployed farms
9. Developer SHALL retrain model on combined dataset (original + farmer corrections)
10. Updated model SHALL be deployed via shadow mode validation process (see Requirement 13)

**Confidence Scoring:**
11. EVERY ML prediction SHALL include confidence score (0-100%)
12. THE System SHALL display to farmer: "Contamination detected (87% confident)"
13. WHEN confidence <70%, THE System SHALL ask: "Please verify this prediction"
14. High-confidence predictions (>90%) SHALL be trusted by default
15. Low-confidence predictions (50-70%) SHALL be flagged for manual review

**Benefits and Privacy:**
16. THE System SHALL learn from real-world farm conditions, not just lab datasets
17. THE Grower SHALL become active participant in model improvement
18. THE System SHALL reduce false positives over time as model adapts to specific farm
19. THE Grower SHALL be able to opt out of data sharing
20. ALL images SHALL be anonymized before developer access (remove farm location, timestamps, farmer ID)
21. Labeled data SHALL contribute to collective model improvement across all farms

### Requirement 23: Hardware Cost Constraints

**User Story:** As a semi-commercial grower with limited capital, I want the monitoring system to be affordable, so that I can adopt the technology without significant financial risk.

#### Acceptance Criteria

1. THE System SHALL operate on hardware with total cost ≤₹15,000 per farm installation
2. THE hardware bill of materials SHALL include:
   - Raspberry Pi 4 (4GB RAM): ₹5,500
   - Pi Camera Module v2 (8MP): ₹2,200
   - DHT22 temperature/humidity sensor: ₹300
   - MH-Z19B CO2 sensor: ₹1,500
   - SIM800L GSM module with antenna: ₹700
   - Power supply (5V 3A adapter + 12V 5Ah battery + buck converter): ₹2,500
   - SanDisk 64GB microSD card (Class 10): ₹600
   - Weatherproof ABS enclosure (IP54 minimum): ₹800
3. THE total hardware cost SHALL be ₹14,100 with ₹900 buffer for price fluctuations
4. THE following components SHALL be excluded from MVP to meet budget constraints:
   - Redundant DHT22 sensors
   - Raspberry Pi 4 8GB RAM upgrade
   - Soil moisture sensors
   - Additional cameras
5. THE hardware cost SHALL be recoverable within 3-6 months through contamination prevention and yield optimization

### Requirement 24: Recurring Operational Costs

**User Story:** As a grower managing ongoing expenses, I want predictable and minimal operational costs, so that the system remains economically viable long-term.

#### Acceptance Criteria

**Monthly Per-Farm Costs:**
1. THE System SHALL incur monthly operational expenses of ₹1,250-4,400 per farm:
   - Cloud hosting (Firebase/AWS): ₹0-3,000 (free tier initially, scales with usage)
   - SMS alert credits (prepaid): ₹1,000 (~500 alerts/month)
   - Electricity (24/7 operation): ₹50-100
   - GSM SIM card (prepaid): ₹200-300
2. THE System SHALL prioritize free-tier cloud services to minimize hosting costs
3. THE System SHALL implement SMS rate limiting to control alert costs
4. THE System SHALL operate efficiently to minimize electricity consumption

**Annual Costs:**
5. THE System SHALL incur annual expenses of:
   - Domain registration + SSL certificate: ₹1,500
   - Cloud storage (if exceeding free tier): ₹0-6,000
6. THE total operational costs SHALL remain ≤₹5,000/month for 10-farm deployment

### Requirement 25: Development and Model Training Costs

**User Story:** As a system developer, I want to understand the one-time development costs, so that I can plan the initial investment and training data collection.

#### Acceptance Criteria

**One-Time Development Costs:**
1. THE initial development SHALL require ₹20,000-35,000 investment:
   - Manual image labeling (500+ contamination images per type): ₹10,000-15,000
   - Field testing and validation (3-6 months): ₹5,000-10,000
   - Model training compute (GPU cloud instances): ₹5,000-10,000
2. THE data collection SHALL require:
   - 500+ contamination images per contamination type (bacterial, green mold, black mold, cobweb, pests)
   - 1,000+ growth stage images (colonization, pinning, fruiting, harvest)
   - 50-100 bags with ground truth yield data
3. THE data collection period SHALL be 3-6 months for initial model training

**Model Retraining Costs:**
4. THE System SHALL require quarterly model retraining at ₹5,000-10,000 per cycle
5. THE System SHALL require annual major version updates at ₹15,000-25,000
6. THE retraining costs SHALL decrease over time as automated labeling improves

### Requirement 26: Deployment and Scaling Costs

**User Story:** As a business operator, I want to understand total deployment costs at different scales, so that I can plan expansion and pricing strategy.

#### Acceptance Criteria

**Single Farm Pilot:**
1. THE single farm pilot deployment SHALL cost ₹40,000-55,000 total:
   - Hardware: ₹14,100
   - Development: ₹20,000-35,000 (amortized)
   - Installation and training: ₹5,000-10,000
2. THE pilot SHALL validate system effectiveness before scaling

**10-Farm Beta Rollout:**
3. THE 10-farm beta deployment SHALL cost ₹2,00,000-2,50,000 total:
   - Hardware (10 units): ₹1,41,000
   - Development (amortized): ₹20,000-35,000
   - Installation and training (10 farms): ₹30,000-50,000
   - Support and maintenance (6 months): ₹10,000-20,000
4. THE per-farm cost SHALL reduce to ₹19,850-29,300 at 10-farm scale due to economies of scale

**Scaling Economics:**
5. THE System SHALL support 100+ farms with:
   - Per-farm capital cost: <₹50,000 (including amortized development)
   - Per-farm operational cost: <₹3,000/month
6. THE System SHALL achieve profitability through subscription model (₹5,000-8,000/month per farm)

### Requirement 27: Competitive Cost Benchmarking

**User Story:** As a grower evaluating monitoring solutions, I want to understand how this system compares to alternatives, so that I can make an informed investment decision.

#### Acceptance Criteria

**Market Comparison:**
1. THE System SHALL be positioned as follows in the market:
   - **Hedgehog Technologies (Netherlands)**: $15,000-25,000 upfront + $200-500/month (₹12,50,000-20,00,000 + ₹16,000-40,000/month) - 100x more expensive
   - **4AG Robotics (USA)**: $10,000-20,000 upfront + $150-300/month (₹8,00,000-16,00,000 + ₹12,000-24,000/month) - 60x more expensive
   - **AdvanceTech India**: ₹50,000-80,000 upfront + ₹2,000-5,000/month - 3-5x more expensive
   - **Manual operation**: ₹0 upfront + ₹10,000-20,000/month (labor costs) - Higher ongoing costs
   - **Our system**: ₹14,100 upfront + ₹1,250-4,400/month - Most affordable automated solution
2. THE System SHALL be 10-50x cheaper than international alternatives
3. THE System SHALL be 3-5x cheaper than existing Indian alternatives

**Value Proposition:**
4. THE System SHALL provide ROI through:
   - Contamination prevention: 10-20% yield loss avoided (₹20,000-40,000/year value)
   - Harvest optimization: 5-10% quality improvement (₹10,000-20,000/year value)
   - Labor reduction: 2-4 hours/day saved (₹15,000-30,000/year value)
5. THE System SHALL achieve break-even within 6 months for typical semi-commercial farm
6. THE System SHALL deliver 3-6x ROI in first year of operation
