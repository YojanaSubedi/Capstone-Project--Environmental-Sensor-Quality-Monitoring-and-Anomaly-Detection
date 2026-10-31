Environmental Sensor Quality Monitoring and Anomaly Detection
BBIM713 — Capstone Project (Data Analytics)  
Team: Sensor Squad  
Project context: Auckland regional environmental ground-gauge measurements (2015–2026)

Project overview
Environmental monitoring stations record rainfall, river discharge and river water level. Historical records can contain missing measurements, duplicated timestamps, unusually high readings and long periods of constant or zero values. This project investigates such patterns using exploratory data analysis (EDA), statistical screening, comparisons between neighbouring rainfall gauges and a focused machine-learning experiment.
Research objective: Explore how data-quality checks and anomaly-detection methods can help prioritise environmental sensor measurements for further review. A flagged measurement is not automatically a confirmed sensor failure.

Team
Team member	Role described in Assessment 1
Yojana Subedi	: Project lead and dashboard/visualisation specialist
Binaya Jung Shah : Data engineering and preprocessing specialist
Jenith Khatri: 	Machine-learning engineer and model evaluator


Dataset
The project uses historical sensor CSV files and a separate station inventory containing locations and sensor metadata.
Sensor type	Source files	Observations	Missing measurement values
Rainfall	113	8,002,331	291,808 (3.65%)
River discharge	53	5,355,120	1,410,900 (26.35%)
River water level	55	5,557,680	1,538,974 (27.69%)
Total	221	18,915,131	3,241,682 (17.14%)
The combined data covers 168 unique station IDs. File counts and unique station IDs are different measures. Missing counts refer to `NaN` values within existing records, not timestamps entirely absent from files.

Standardised analytical schema
The sensor CSVs are transformed into a common structure:
`station_id`, `timestamp`, `sensor_type`, `value`, `unit`
Different sensor measurements retain their original meanings and units (e.g., rainfall in mm, discharge in m³/s, level in m). The station inventory is used separately to obtain geographical context.

Methodology

1. Data ingestion, cleaning and EDA
Read and standardise historical rainfall, discharge and level CSV records using Python and Pandas.
Explore missing measurements, zero values and repeated station–timestamp records.
Apply station- and sensor-specific 99.9th-percentile thresholds to flag unusually high historical readings; exclude zero rainfall when fitting rainfall high-value thresholds and require at least 1,000 eligible records.
Retain the distinction between statistically unusual and physically invalid or confirmed faulty observations.
The broad EDA produced 8,913 high-value statistical flags across the three measurement types. These are flagged observations, not 8,913 confirmed faults.

2. Long-zero rainfall and neighbour investigation
Rainfall can legitimately be zero for many hours, so a sustained zero period does not prove a fault. We:
Screened rainfall series for unusually long consecutive zero-reading episodes.
Chose 10 candidate rainfall stations based on the selected long-zero episodes.
Found the five geographically nearest rainfall gauges for each target using station coordinates and Haversine distance.
Compared aligned readings over up to 168 hours of each selected episode.
Counted a neighbour disagreement when the target read 0 mm and at least two of the five neighbours recorded rainfall greater than 0 mm during the same hour.
Ranked these ten candidates by their disagreement proportions and selected four for detailed modelling.
Important: 168 hours was the investigation window, not necessarily the minimum duration of a zero episode. The detailed neighbour comparison was not performed across every hour of all 113 rainfall files.
Selected rainfall station	Station ID	Initial disagreement rate
Cornwallis RAWS @ Parks Depot	740601	35.12%
Puni Hill Reserve	742826	34.52%
Khyber Pass Meteorological	648709	22.62%
Pakiri Block Road	642615	19.05%
A stricter follow-up also checked whether at least two neighbours each recorded 1 mm or more. Localised rainfall can differ between nearby sites, so neighbour disagreement is a screening signal, not proof of a defective gauge.

3. Machine-learning experiment
The focused experiment used 209,699 hourly observations from the four selected rainfall stations. The data was split chronologically per station into:
Training: 146,787 observations (approximately 70%)
Validation: 31,456 observations (approximately 15%)
Testing: 31,456 observations (approximately 15%)
Eight features described current and recent rainfall, changes and rolling statistics, consecutive zero counts, hour and month. Preprocessing parameters were fitted on training data.
Six evaluated models: Random Forest, XGBoost, Isolation Forest, Local Outlier Factor (LOF), One-Class SVM and an LSTM Autoencoder. The LSTM Autoencoder reconstructs sequences; it is not presented as a rainfall forecasting system.

5. Synthetic fault testing
Because the historical dataset did not include verified labels of actual sensor faults, we deliberately injected synthetic examples to support controlled evaluation:
Injected fault type	Separate episodes	Active fault hours	Numerically modified test readings
Spike	157	157	157
Flatline	8	140	51
Stuck at zero	8	162	69
Total	173	459	277
An active fault hour is an hour within a deliberately imposed fault interval. It may not be a modified reading if its original value already matched the imposed value (for example, a natural zero during a stuck-zero interval).

Results
Model comparison on the synthetic-fault test
Model	Test F1 score
Random Forest	72.18%
One-Class SVM	72.02%
Local Outlier Factor	71.36%
XGBoost	67.71%
LSTM Autoencoder	51.56%
Isolation Forest	29.29%
Random Forest achieved 99.37% precision but 56.68% recall (157 true positives, 1 false positive and 120 false negatives) against the final synthetic modified-reading labels. Precision is not accuracy.
At the episode level, Random Forest identified all 157 synthetic spike episodes, but none of the eight flatline or eight stuck-at-zero episodes. The other five evaluated models also missed those 16 persistent episodes. This highlights the risk of evaluating a sensor-quality method solely through aggregate scores.
Historical case study
At Puni Hill Reserve, a separate historical investigation examined a 192-hour zero-rainfall period from 28 January to 4 February 2023. Neighbour comparisons identified 42 higher-priority disagreement hours. This is evidence for further investigation, not confirmation that the sensor was malfunctioning.

Dashboard
Open the Streamlit dashboard https://exyzruvsx7z5gcfsqrwunk.streamlit.app/
The dashboard presents precomputed, retrospective analysis outputs; it does not connect to live sensors or run real-time fault prediction.

Page and its Purpose
Overview:	Summary of the focused rainfall experiment
Sensor:  Explorer	Station time series and model-generated anomaly markers
Model Comparison: 	Compare evaluation metrics across six models
Fault Episodes: 	Review injected fault episodes and model detections
Historical Investigation: Explore the Puni Hill historical zero-rainfall case

Project workflow
Historical gauge CSVs + station inventory
                  | 
         Python data ingestion
                  |  
     Standardisation and broad EDA
                  |   
      Long-zero rainfall screening
                  |
   Ten-station neighbour investigation
                  |   
        Four-station ML experiment
                  |
    Synthetic-fault model evaluation
                  |
       Precomputed result exports
                  |   
          Streamlit dashboard

Repository structure and reproducibility
This repository includes a Streamlit application (`app.py`) and its dependencies (`requirements.txt`). See the repository's actual file listing for available results CSVs, source code and notebooks.
To run the dashboard locally, where the required application files are present:

pip install -r requirements.txt
streamlit run app.py

Data access: The historical raw data may be subject to client permissions and confidentiality restrictions. 

Scope and limitations
The original Assessment 1 proposal included broader operational functions: deduplication, KNN/linear gap reconstruction, a four-category quality taxonomy, a 30-day Sensor Health Index, an operational alerting dashboard and Power BI delivery. The completed research implementation instead emphasises broad EDA, statistical screening, a focused four-station rainfall ML experiment and retrospective results presented in Streamlit.
Not all original proposal components have been demonstrated as completed. In particular:
Full network-wide missing-data reconstruction and the 30-day Sensor Health Index are not presented as implemented.
The four-category operational workflow and automatic live sensor alerts are not presented as deployed.
Results are based on selected stations and synthetic faults; performance on verified real-world failures remains unknown.
Neighbour disagreements and high statistical readings are suspicions, not confirmed failures.
Future work

Evaluate physically informed screening rules and persistent-fault detection for flatlines and stuck-zero periods.
Expand neighbour comparisons across additional stations and longer historical windows.
Obtain verified maintenance/failure annotations to validate real-world usefulness.
Revisit the original imputation, sensor-health scoring and operational alerting objectives where approved and feasible.
Improve reproducibility, permissions documentation and evaluation of processing efficiency.

Acknowledgements and references
This is a BBIM713 student capstone research project. For the full academic context, original proposal and cited literature, see the associated Assessment 1 proposal and Assessment 2 report. No client approval, operational deployment or confirmed sensor failures are implied by this repository.
