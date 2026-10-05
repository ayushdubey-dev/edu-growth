# 🎓 Edu Growth
### AI-Driven Batch Performance & Student Analytics System

> **Team Ctrl Freaks** · Domain: EdTech / Machine Learning · Doc Version 1.0

Edu Growth analyzes a **99-column student performance dataset** covering six subjects, four practical labs, attendance, assignment behavior, extracurricular participation, and prior CGPA. It is designed to surface subject and unit-level patterns, estimate the final semester grade from historical examples, and identify unusual records for review. The current spreadsheet is a student-level snapshot; it does not contain teacher efficacy, risk labels, or assessment dates.

---

## 📑 Table of Contents
1. [Problem Statement](#-problem-statement)
2. [Solution](#-solution)
3. [Dataset: 99 Columns](#-dataset-99-columns)
4. [Analysis Workflow](#-analysis-workflow)
5. [Model Plan](#-model-plan)
6. [Database Design](#-database-design-postgresql)
7. [API Overview](#-api-overview)
8. [Tech Stack](#-tech-stack)
9. [Setup](#-setup)
10. [Project Structure](#-project-structure)
11. [Team](#-team)
12. [Roadmap](#-roadmap)

---

## ⚠️ Problem Statement

| Problem | What happens today |
|---|---|
| **Coarse evaluation** | Overall grades can hide differences between subjects and the five unit scores recorded for each subject. |
| **Disconnected signals** | Attendance, assessment marks, assignment delays, lab performance, and participation are often reviewed separately. |
| **Late intervention** | A final grade alone does not show which currently available signals may warrant an earlier human review. |
| **Unclear patterns** | Staff need a consistent way to compare students and subjects without treating a model score as a diagnosis. |

## 💡 Solution

- **One validated student profile** – ingest the spreadsheet and check required columns, types, duplicates, missing values, and plausible ranges.
- **Subject and unit insights** – compare ST1, ST2, PUT, unit marks, assignments, quizzes, and attendance across the six subjects.
- **Practical performance view** – compare execution and viva scores with submission delays across the four labs.
- **Final-grade estimation** – train a supervised model using `final_semester_grade` as the target and only information available before that outcome.
- **Exploratory student groups and outliers** – use PCA, clustering, and anomaly scores to support review, not to assign definitive risk labels.
- **Human-reviewed support** – show contributing signals and keep medical leave as sensitive context, never as a penalty or an automated decision.

## 🗺 Analysis Workflow

```mermaid
flowchart TD
    A["Google Sheet / CSV<br/>99 student fields"] --> B["Schema validation<br/>IDs, types, ranges, missingness"]
    B --> C["EDA and preprocessing<br/>encode categories, impute, scale"]
    C --> D["Subject and unit analysis"]
    C --> E["PCA feature transformation"]
    E --> F["Grade regression<br/>target: final_semester_grade"]
    E --> G["Exploratory clustering<br/>K-Means"]
    E --> H["Outlier review<br/>Isolation Forest"]
    D --> I["Reviewed analytics and reports"]
    F --> I
    G --> I
    H --> I
```

## 📋 Dataset: 99 Columns

**Source:** [Edu Growth student dataset (Google Sheets)](https://docs.google.com/spreadsheets/d/18E6kDb3bGOOatyn9IRaZRoQnjmRVLRNUnkMCi7aCUPo/edit?usp=sharing). Export a CSV copy to `data/raw/` before running analysis. The schema groups below add up to 99 columns.

| Group | Count | Columns |
|---|---:|---|
| Student information and add-ons | 11 | `roll_no`, `full_name`, `class_section`, `overall_attendance_pct`, `theory_attendance_pct`, `practical_attendance_pct`, `previous_cgpa`, `medical_leave_days`, `society_participation_pc`, `sports_activity_level`, `final_semester_grade` |
| Subject attendance | 6 | `coa_attendance_pct`, `maths4_attendance_pct`, `dstl_attendance_pct`, `ds_attendance_pct`, `python_attendance_pct`, `cyber_attendance_pct` |
| Lab attendance | 4 | `lab_ds_attendance_pct`, `lab_python_attendance_pct`, `lab_coa_attendance_pct`, `lab_cyber_attendance_pct` |
| COA assessments | 11 | `coa_st1_marks`, `coa_st2_marks`, `coa_put_marks`, `coa_unit_1_marks`–`coa_unit_5_marks`, `coa_assignment_score`, `coa_assignment_delay_hours`, `coa_quiz_score` |
| Maths4 assessments | 11 | `maths4_st1_marks`, `maths4_st2_marks`, `maths4_put_marks`, `maths4_unit_1_marks`–`maths4_unit_5_marks`, `maths4_assignment_score`, `maths4_assignment_delay_hours`, `maths4_quiz_score` |
| DSTL assessments | 11 | `dstl_st1_marks`, `dstl_st2_marks`, `dstl_put_marks`, `dstl_unit_1_marks`–`dstl_unit_5_marks`, `dstl_assignment_score`, `dstl_assignment_delay_hours`, `dstl_quiz_score` |
| DS assessments | 11 | `ds_st1_marks`, `ds_st2_marks`, `ds_put_marks`, `ds_unit_1_marks`–`ds_unit_5_marks`, `ds_assignment_score`, `ds_assignment_delay_hours`, `ds_quiz_score` |
| Python assessments | 11 | `python_st1_marks`, `python_st2_marks`, `python_put_marks`, `python_unit_1_marks`–`python_unit_5_marks`, `python_assignment_score`, `python_assignment_delay_hours`, `python_quiz_score` |
| Cybersecurity assessments | 11 | `cyber_st1_marks`, `cyber_st2_marks`, `cyber_put_marks`, `cyber_unit_1_marks`–`cyber_unit_5_marks`, `cyber_assignment_score`, `cyber_assignment_delay_hours`, `cyber_quiz_score` |
| Lab performance | 12 | For each of `ds`, `python`, `coa`, and `cyber`: `lab_<subject>_execution_score`, `lab_<subject>_viva_score`, `lab_<subject>_submission_delay_hours` |
| **Total** | **99** | Includes `final_semester_grade`, the supervised-learning target |

The source sheet's actual value formats, score scales, missing-value conventions, and row count must be profiled during EDA. Do not infer scale limits or category encodings from the column names alone.

## 📊 Analysis Capabilities

- **Student and cohort summaries:** compare attendance, prior CGPA, marks, labs, and participation overall and by `class_section`.
- **Subject and unit diagnostics:** compare ST1, ST2, PUT, unit marks, assignment scores/delays, and quiz scores for each subject.
- **Lab diagnostics:** summarize execution, viva, attendance, and submission-delay measures for each lab.
- **Grade prediction:** predict `final_semester_grade` from eligible pre-outcome fields; do not include identifiers or the target among predictors.
- **Exploratory PCA and clustering:** reduce correlated numeric features and examine student groupings; clusters require interpretation and validation.
- **Anomaly review:** use Isolation Forest to flag unusual feature combinations for a person to inspect. An anomaly score is not a validated risk label or diagnosis.

This is a cross-sectional dataset unless additional dated snapshots are supplied. It cannot establish learning velocity, recovery after interventions, or sudden changes over time. It contains no teacher identifiers or teacher outcomes, so teacher-efficacy scoring and automatic faculty assignment are not supported. Peer-mentor suggestions would also need explicit eligibility, capacity, and safeguarding rules before implementation.

---

## 🤖 Machine Learning Models

| Objective | Algorithm | Input Features | Output & Target Metrics |
|---|---|---|---|
| **Final semester grade estimate** | Baseline classifier or regressor, selected after inspecting target values | Eligible attendance, prior CGPA, assessment, assignment, quiz, lab, and participation fields | Predicted `final_semester_grade`; report validation metrics appropriate to its actual type |
| **Anomaly review** | Isolation Forest | Scaled numeric features selected for the use case | Anomaly score for human review; not a ground-truth risk class |
| **Exploratory grouping** | PCA followed by K-Means | Scaled, leakage-checked academic features | Candidate clusters to interpret and validate; no fixed labels or cluster count assumed |

### Preprocessing & Feature Engineering
1. Validate the 99-column schema, duplicate `roll_no` values, data types, score ranges, and missingness before imputation.
2. Exclude `roll_no` and `full_name` from model features. Treat `class_section` as categorical and consider it when splitting evaluation data.
3. Exclude `final_semester_grade` from predictors; use it only as the supervised target. Confirm every predictor is available before the grade is known to prevent leakage.
4. Encode `sports_activity_level`, impute only after inspecting missingness, and scale numeric features for PCA and distance-based models.
5. Keep `medical_leave_days` out of automated risk or anomaly scoring by default. If used for analysis, restrict access and interpret it as sensitive context, not a performance penalty.
6. Use held-out validation and report measured results; no performance target is claimed until the dataset has been evaluated.

---

## 🗄 Database Design (PostgreSQL)

The supplied CSV/Sheet is the source of truth. A database is optional for initial EDA; if PostgreSQL is added, retain the source fields in an import table and store generated outputs separately. The dataset contains no teacher records.

```mermaid
erDiagram
    DATASET_ROWS ||--o{ MODEL_OUTPUTS : "scored by"

    DATASET_ROWS {
        int row_id PK
        string roll_no
        string full_name
        string class_section
        decimal previous_cgpa
        string final_semester_grade
        string remaining_source_fields
    }
    MODEL_OUTPUTS {
        int output_id PK
        int row_id FK
        string model_name
        string model_version
        string predicted_grade
        decimal anomaly_score
        int cluster_id
        datetime created_at
    }
```

`remaining_source_fields` represents the other source columns, not one literal column. Restrict access to names, roll numbers, and medical leave data; do not expose them in model exports or general analytics.

---

## 🔌 API Overview

> Proposed endpoints only; the current FastAPI files are scaffolds and these routes are not implemented yet.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/v1/datasets/import` | Validate and import the 99-column CSV |
| `GET` | `/api/v1/students/{roll_no}` | Student profile with subject, unit, attendance, and lab summaries |
| `GET` | `/api/v1/students/{roll_no}/grade-prediction` | Estimated `final_semester_grade` with model version |
| `GET` | `/api/v1/students/{roll_no}/anomaly` | Exploratory anomaly score, not a risk label |
| `GET` | `/api/v1/analytics/subjects/{subject_code}` | Aggregate subject and unit analysis |
| `GET` | `/api/v1/analytics/pca` | Explained variance and component loadings |
| `GET` | `/api/v1/analytics/cohorts` | Validated exploratory cluster summaries |

---

## 🧰 Tech Stack

| Layer | Technology |
|---|---|
| **Data analysis** | Jupyter notebooks, pandas, NumPy |
| **ML** | scikit-learn (preprocessing, PCA, regression/classification, K-Means, Isolation Forest) |
| **Backend (planned)** | Python FastAPI and Uvicorn |
| **Database (optional)** | PostgreSQL for imported rows and model outputs |
| **Desktop UI (planned)** | JavaFX |

---

## 🚀 Installation & Setup (No Docker)

**Prerequisites:** Python 3.10+, PostgreSQL 14+, JDK 17+ with Maven or Gradle, Node.js (for the web dashboard) and Git.

1. **Clone the repository** from GitHub and open the project folder.
2. **Create the database** – create a PostgreSQL database named `edugrowth` and run the schema file from the `database` folder.
3. **Backend** – inside the `backend` folder, create a Python virtual environment, activate it and install the packages listed in `requirements.txt`.
4. **Configure environment** – create a `.env` file in `backend` with your PostgreSQL connection URL and the path of the ML models folder.
5. **Start the API** – run the FastAPI app with Uvicorn on port 8000. Interactive API docs will be available at `http://localhost:8000/docs`.
6. **ML models** – inside the `ml` folder, install its requirements and run the training scripts (risk classifier, score predictor, anomaly detector, clustering). Trained models are exported to `ml/models`.
7. **Desktop app** – inside `frontend-desktop`, build and run the JavaFX app with Maven.
8. **Student web dashboard** – inside `frontend-web`, install dependencies and start the dev server.

---

## 📁 Project Structure

```text
data/
├── raw/
├── processed/
└── pca_transformed/
notebooks/
├── 01_eda_data_cleaning.ipynb
├── 02_pca_dimensionality_reduction.ipynb
├── 03_cgpa_prediction_model.ipynb
├── 04_mentor_clustering_model.ipynb
└── 05_risk_isolation_forest.ipynb
ml_pipeline/
├── __init__.py
├── config.py
├── preprocessor.py
├── pca_transformer.py
├── trainer.py
└── utils.py
artifacts/
app/
├── __init__.py
├── main.py
├── api/
│   ├── __init__.py
│   └── v1/
│       ├── __init__.py
│       ├── router.py
│       └── endpoints/
│           ├── __init__.py
│           ├── pca_analytics.py
│           ├── mentor.py
│           ├── cgpa.py
│           ├── risk.py
│           └── velocity.py
├── core/
│   ├── config.py
│   ├── security.py
│   └── database.py
├── services/
│   ├── __init__.py
│   ├── pca_service.py
│   ├── prediction_service.py
│   └── mentor_service.py
└── schemas/
    ├── __init__.py
    ├── student_schema.py
    ├── pca_schema.py
    └── response_schema.py
frontend_javafx/
tests/
├── test_pca_pipeline.py
└── test_endpoints.py
requirements.txt
.env.example
README.md
```

The data and artifact directories start empty. Keep local datasets, trained model files, and `.env` secrets out of version control; `.env.example` is the safe configuration template.

---

## 👥 Team

**Team Ctrl Freaks**

| Member | Domain | Mentor(s) |
|---|---|---|
| Suhani Agarwal | Machine Learning | Harsh Raj, Anjali Sirohi |
| Syed Rafiuddin Altamash | Machine Learning | Harsh Raj, Anjali Sirohi |
| Akash Raghuvanshi | Machine Learning | Harsh Raj, Anjali Sirohi |
| Pranav Prajapati | Machine Learning | Harsh Raj, Anjali Sirohi |
| Raunak Agrahari | Frontend | Akshat Sharma |
| Vivek Soni | Backend | Shreya Singh |
| Prashant Singh | Designing | — |

**Module ownership:** Suhani & Pranav – preprocessing and feature engineering; ML team – risk classifier, score predictor, anomaly detection, clustering, mentor switcher; Raunak – JavaFX desktop client; Vivek – FastAPI and PostgreSQL; Prashant – UI/UX design.

---

## 🛣 Roadmap

- [x] System architecture and blueprint
- [x] 10 data pillars and 15 engines defined
- [x] Database design
- [ ] Data ingestion (CSV upload) and preprocessing pipeline
- [ ] Efficacy calculation and dynamic mentor allocation
- [ ] ML model training and export
- [ ] FastAPI endpoints
- [ ] JavaFX teacher portal and student web dashboard
- [ ] Alerts and PDF reports
- [ ] SHAP explainability, What-If simulator, role-based access

---

## 📄 License
Developed by Team Ctrl Freaks for academic purposes. Add a license (for example, MIT) before public release.
