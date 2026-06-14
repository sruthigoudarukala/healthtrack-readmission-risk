🏥 HealthTrack Readmission Risk System


An end-to-end AI pipeline that predicts 30-day patient readmission risk, generates clinical alert notes, and automates daily care coordination — built for HealthTrack Regional Hospital.




📋 Project Overview

This project was built as part of a Field Data Engineer (FDE) assignment. The goal was to take raw hospital discharge data, audit it, train a machine learning model, generate AI-powered clinical alerts, and automate the daily workflow — all within a single cohesive pipeline.


🗂️ Repository Structure

FileDescriptionhealthtrack_profiling(part-A).ipynbData profiling & audit findingshealthtrack_gx(PART-B).ipynbGreat Expectations data validationhealthtrack_model(PART-C&D).ipynbML model training + LangChain alert generationhealthtrack_n8n_workflow.jsonn8n automation workflowhealthtrack_handoff.docxCMO handoff documenthealthtrack_raw.csvOriginal raw dataset (300 rows)healthtrack_clean.csvCleaned dataset (230 rows)healthtrack_alerts_today.csvAI-generated alerts for High-risk patientshealthtrack_profile.htmlydata-profiling reporthealthtrack_gx_report.htmlGreat Expectations validation report


🔬 Part A — Data Profiling & Audit

Tool: ydata-profiling

Dataset: 300 patient discharge records, 12 columns

Audit Findings

IssueCount%ImpactMissing patient_id258.33%Cannot link alerts to patientsMissing readmission_30d3010.0%Blocks model training (target label)Sentinel bills (-9999)155.0%EHR placeholder, not real billing data

Highest readmission department: Oncology at 51%


30 patient records are missing their readmission outcome, which prevents the AI model from learning from 10% of this year's discharges and risks missing early intervention opportunities for high-risk patients.




✅ Part B — Data Validation (Great Expectations)

Tool: Great Expectations v1.18.1

5 Expectations defined:

pythonexpect_column_values_to_not_be_null("patient_id")           # ❌ Failed (25 missing)
expect_column_values_to_not_be_null("readmission_30d")      # ❌ Failed (30 missing)
expect_column_values_to_be_in_set("readmission_30d", [0,1]) # ✅ Passed
expect_column_values_to_be_between("total_bill_usd", 500, 200000) # ❌ Failed (15 sentinel)
expect_column_values_to_be_in_set("department", [...])      # ✅ Passed

Result: 2/5 passed (40%) — confirming the audit findings from Part A.


🤖 Part C — Readmission Risk Model

Tool: scikit-learn — GradientBoostingClassifier

Clean dataset: 230 rows after removing 70 invalid records

Train/Test split: 80/20 (random_state=42)

Features Used


age, length_of_stay, prev_admissions_12m, num_medications
One-hot encoded: department, discharge_disposition, insurance_type


Model Performance

MetricScoreAccuracy0.783Precision0.643Recall0.643F1 Score0.643ROC-AUC0.754


Recall is the most critical metric here. A False Negative (missed high-risk patient) can result in a preventable readmission costing $15,200+. A False Positive simply triggers an unnecessary follow-up call.



Risk Tier Distribution

TierPatientsThreshold🔴 High87 (37.8%)probability ≥ 0.65🟡 Medium5 (2.2%)probability 0.40–0.64🟢 Low138 (60%)probability < 0.40


💬 Part D — LangChain Clinical Alert Generation

Tool: LangChain + Groq (llama-3.1-8b-instant)

Alerts generated for: 87 High-risk patients only

Each alert is a 3-sentence clinical note:


Risk tier + top 2 contributing factors
Recommended action
Professional closing with urgency level


Alert Priority Logic:


URGENT → prev_admissions_12m >= 2 AND risk_tier == High
STANDARD → all other High-risk patients


Output: healthtrack_alerts_today.csv (7 columns)


⚙️ Part E — n8n Automation Workflow

Tool: n8n (cloud)

Schedule: Every day at 7:30 AM

Workflow Nodes

Schedule Trigger (7:30 AM)
    ↓
Google Sheets (fetch alerts data)
    ↓
Code Node (quality gate — check for nulls & sentinel values)
    ↓
IF Node (status == OK?)
    ├── TRUE → Slack #care-coordination (daily summary)
    │              ↓
    │           Code Node (format URGENT patient list)
    │              ↓
    │           Slack #care-coordination (URGENT alerts)
    │
    └── FALSE → Slack #data-ops (pipeline blocked alert)


📄 Part F — CMO Handoff Document

A plain-English, one-page document written for the Chief Medical Officer covering:


What was found in the data
How the risk model works
What happens every morning (5 automated steps)
What to do when a High-risk flag appears



🚀 How to Run

Prerequisites

bashpip install pandas ydata-profiling great-expectations scikit-learn langchain langchain-groq

Steps


Clone the repo
Place healthtrack_raw.csv in the root directory
Run notebooks in order: Part A → Part B → Part C&D
Import healthtrack_n8n_workflow.json into your n8n instance
Configure Google Sheets and Slack credentials in n8n



🛠️ Tech Stack

ToolPurposePython / PandasData processingydata-profilingData profilingGreat ExpectationsData validationscikit-learnML model trainingLangChain + GroqAI alert generationn8nWorkflow automationSlackCare team notificationsGoogle SheetsData source for n8n


👩‍💻 Author

Sruthi Goudarukala

Field Data Engineer — HealthTrack Regional Hospital Assignment

GitHub
