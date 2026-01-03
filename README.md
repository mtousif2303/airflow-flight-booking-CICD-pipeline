# Airflow Flight Booking CI/CD Pipeline

A production-ready data pipeline for processing flight booking data using Apache Airflow, Google Cloud Composer, Dataproc Serverless, and BigQuery with automated CI/CD deployment via GitHub Actions.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Usage](#usage)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

## 🎯 Overview

This project implements an end-to-end data pipeline that:
- Monitors Google Cloud Storage (GCS) for new flight booking CSV files
- Processes data using Apache Spark on Dataproc Serverless
- Performs data transformations and aggregations
- Stores results in BigQuery for analytics
- Automatically deploys to development and production environments via GitHub Actions

## 🏗️ Architecture

```
┌─────────────────┐
│   GitHub Repo   │
│  (develop/main) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ GitHub Actions  │
│    CI/CD        │
└────────┬────────┘
         │
         ├──────────────┬──────────────┐
         ▼              ▼              ▼
   ┌─────────┐   ┌──────────┐   ┌─────────┐
   │   GCS   │   │ Composer │   │  Spark  │
   │ Buckets │   │   DAGs   │   │  Jobs   │
   └─────────┘   └──────────┘   └─────────┘
                       │
                       ▼
              ┌────────────────┐
              │ Airflow DAG    │
              │  Execution     │
              └────────┬───────┘
                       │
         ┌─────────────┼─────────────┐
         ▼             ▼             ▼
   ┌─────────┐  ┌──────────┐  ┌──────────┐
   │   GCS   │  │ Dataproc │  │ BigQuery │
   │ Sensor  │  │Serverless│  │  Tables  │
   └─────────┘  └──────────┘  └──────────┘
```

## ✨ Features

### Data Processing
- **Automated File Detection**: GCS sensor monitors for new flight booking files
- **Spark Transformations**: 
  - Weekend booking classification
  - Lead time categorization (Last-Minute, Short-Term, Long-Term)
  - Booking success rate calculation
  - Route-based insights aggregation
  - Booking origin analysis

### Infrastructure
- **Multi-Environment Support**: Separate dev and prod environments
- **Serverless Processing**: Uses Dataproc Serverless for cost optimization
- **Scalable Storage**: BigQuery for analytics-ready data
- **Automated Deployments**: GitHub Actions CI/CD pipeline

### Analytics Outputs
Three BigQuery tables are generated:
1. **Transformed Data**: Enhanced flight booking data with derived columns
2. **Route Insights**: Aggregated metrics per flight route
3. **Booking Origin Insights**: Success rates and lead times by booking source


The GitHub CI/CD Actions
==================

<img width="3456" height="2000" alt="image" src="https://github.com/user-attachments/assets/0da258eb-2d5c-4851-93a6-0018b9b2e4f0" />

The DAG

<img width="1721" height="1031" alt="image" src="https://github.com/user-attachments/assets/c8be3cf7-960a-4a20-b0d9-1a15039b8c51" />

The tables are created in BigQuery as output of Dag 

<img width="2954" height="1838" alt="image" src="https://github.com/user-attachments/assets/99bffa68-4627-4eb8-88b0-c46b8e44dc9a" />




## 📦 Prerequisites

### Required Services
- Google Cloud Project with billing enabled
- Google Cloud Composer (Airflow managed service)
- Google Cloud Storage buckets
- BigQuery datasets
- Dataproc API enabled
- GitHub repository

### Required Permissions
Your service account needs:
- `roles/composer.worker`
- `roles/composer.user`
- `roles/dataproc.editor`
- `roles/bigquery.dataEditor`
- `roles/storage.objectAdmin`

### Local Development
- Python 3.8+
- gcloud CLI installed and configured
- Git

## 📁 Project Structure

```
airflow-flight-booking-CICD-pipeline/
├── .github/
│   └── workflows/
│       └── cicd.yml                    # GitHub Actions workflow
├── airflow_job/
│   └── airflow_job.py                  # Airflow DAG definition
├── spark_job/
│   └── spark_transformation_job.py     # PySpark transformation script
├── variables/
│   ├── dev/
│   │   └── variables.json              # Dev environment variables
│   └── prod/
│       └── variables.json              # Prod environment variables
├── .gitignore
└── README.md
```

## 🚀 Setup Instructions

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/airflow-flight-booking-CICD-pipeline.git
cd airflow-flight-booking-CICD-pipeline
```

### 2. Set Up Google Cloud Resources

#### Create GCS Buckets

```bash
# Main project bucket
gsutil mb -l us-central1 gs://airflow-projects-gds-03012026

# Create folder structure
gsutil mkdir gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/
gsutil mkdir gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/spark-job/
gsutil mkdir gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/source-dev/
gsutil mkdir gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/source-prod/
```

#### Create BigQuery Datasets

```bash
# Development dataset
bq mk --location=us-central1 flight_data_dev

# Production dataset
bq mk --location=us-central1 flight_data_prod
```

#### Create Composer Environments

```bash
# Development environment
gcloud composer environments create airflow-gds \
    --location us-central1 \
    --python-version 3

# Production environment
gcloud composer environments create airflow-prod \
    --location us-central1 \
    --python-version 3
```

### 3. Create Service Account for GitHub Actions

```bash
PROJECT_ID="project-040088dd-8c9a-464e-96f"

# Create service account
gcloud iam service-accounts create github-actions-sa \
    --display-name="GitHub Actions Service Account" \
    --project=$PROJECT_ID

# Grant necessary roles
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:github-actions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/composer.user"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:github-actions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/storage.objectAdmin"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:github-actions-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
    --role="roles/composer.worker"

# Create and download key
gcloud iam service-accounts keys create github-actions-key.json \
    --iam-account=github-actions-sa@${PROJECT_ID}.iam.gserviceaccount.com
```

### 4. Configure GitHub Secrets

Add the following secrets to your GitHub repository:

**Settings → Secrets and variables → Actions → New repository secret**

| Secret Name | Description | Value |
|-------------|-------------|-------|
| `GCP_SA_KEY` | Service account JSON key | Contents of `github-actions-key.json` |
| `GCP_PROJECT_ID` | Google Cloud Project ID | `project-040088dd-8c9a-464e-96f` |

### 5. Grant Dataproc Service Account Permissions

```bash
# Grant permissions to the default compute service account
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:68634013284-compute@developer.gserviceaccount.com" \
    --role="roles/dataproc.worker"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:68634013284-compute@developer.gserviceaccount.com" \
    --role="roles/bigquery.dataEditor"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:68634013284-compute@developer.gserviceaccount.com" \
    --role="roles/storage.objectViewer"
```

## ⚙️ Configuration

### Environment Variables

Configure environment-specific variables in `variables/dev/variables.json` and `variables/prod/variables.json`:

**Development (`variables/dev/variables.json`):**
```json
{
  "env": "dev",
  "gcs_bucket": "airflow-projects-gds-03012026",
  "bq_project": "project-0400ssasq12388dd-8c9a-464e-96f",
  "bq_dataset": "flight_data_dev",
  "tables": {
    "transformed_table": "transformed_flight_data",
    "route_insights_table": "route_insights",
    "origin_insights_table": "booking_origin_insights"
  }
}
```

**Production (`variables/prod/variables.json`):**
```json
{
  "env": "prod",
  "gcs_bucket": "airflow-projects-gds-03012026",
  "bq_project": "project-0421008sssa8dd-8c9a-464e-96f",
  "bq_dataset": "flight_data_prod",
  "tables": {
    "transformed_table": "transformed_flight_data",
    "route_insights_table": "route_insights",
    "origin_insights_table": "booking_origin_insights"
  }
}
```

### Airflow DAG Configuration

The DAG is configured with:
- **Schedule**: Manual trigger (`schedule_interval=None`)
- **Retries**: 1 retry with 5-minute delay
- **Timeout**: 300 seconds for GCS file sensor
- **Dataproc Version**: 2.2

## 🚢 Deployment

### Automated Deployment (Recommended)

The pipeline uses GitHub Actions for automated deployment:

#### Deploy to Development
```bash
# Create and push to develop branch
git checkout -b develop
git add .
git commit -m "Deploy to dev"
git push origin develop
```

This triggers:
1. Upload variables to dev Composer bucket
2. Import variables into Airflow dev environment
3. Upload Spark job to GCS
4. Deploy DAG to dev Composer

#### Deploy to Production
```bash
# Merge to main branch
git checkout main
git merge develop
git push origin main
```

This triggers:
1. Upload variables to prod Composer bucket
2. Import variables into Airflow prod environment
3. Upload Spark job to GCS
4. Deploy DAG to prod Composer

### Manual Deployment

```bash
# Upload Spark job
gsutil cp spark_job/spark_transformation_job.py \
    gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/spark-job/

# Import variables
gcloud composer environments run airflow-gds \
    --location us-central1 \
    variables import -- /home/airflow/gcs/data/dev/variables.json

# Upload DAG
gcloud composer environments storage dags import \
    --environment airflow-gds \
    --location us-central1 \
    --source airflow_job/airflow_job.py
```

## 💻 Usage

### 1. Upload Source Data

Upload your flight booking CSV file to GCS:

```bash
# For development
gsutil cp flight_booking.csv \
    gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/source-dev/

# For production
gsutil cp flight_booking.csv \
    gs://airflow-projects-gds-03012026/airflow-flight-booking-CICD-Project/source-prod/
```

**Expected CSV Schema:**
```
flight_day, purchase_lead, flight_duration, length_of_stay, 
num_passengers, booking_complete, booking_origin, route
```

### 2. Trigger the DAG

#### Via Airflow UI
1. Navigate to Composer environment URL
2. Find DAG: `flight_booking_dataproc_bq_dag`
3. Click "Trigger DAG"

#### Via gcloud CLI
```bash
gcloud composer environments run airflow-gds \
    --location us-central1 \
    dags trigger -- flight_booking_dataproc_bq_dag
```

### 3. Monitor Execution

```bash
# Check DAG status
gcloud composer environments run airflow-gds \
    --location us-central1 \
    dags list

# View task logs
gcloud composer environments run airflow-gds \
    --location us-central1 \
    tasks logs -- flight_booking_dataproc_bq_dag run_spark_job_on_dataproc_serverless <execution_date>
```

### 4. Query Results in BigQuery

```sql
-- View transformed data
SELECT * FROM `project-0400dsdsd88dd-8c9a-464e-96f.flight_data_dev.transformed_flight_data`
LIMIT 100;

-- Analyze route insights
SELECT 
    route,
    total_bookings,
    avg_flight_duration,
    avg_stay_length
FROM `project-04008sdsd8dd-8c9a-464e-96f.flight_data_dev.route_insights`
ORDER BY total_bookings DESC;

-- Check booking origin performance
SELECT 
    booking_origin,
    total_bookings,
    success_rate,
    avg_purchase_lead
FROM `project-04008sdsd8dd-8c9a-464e-96f.flight_data_dev.booking_origin_insights`
ORDER BY success_rate DESC;
```

## 📊 Monitoring

### Airflow Monitoring

Access the Airflow UI:
```bash
# Get Airflow UI URL
gcloud composer environments describe airflow-gds \
    --location us-central1 \
    --format="value(config.airflowUri)"
```

Key metrics to monitor:
- DAG run status
- Task execution time
- Failure rate
- Retry attempts

### Dataproc Monitoring

View batch jobs in Console:
```
Cloud Console → Dataproc → Batches
```

Or via CLI:
```bash
gcloud dataproc batches list --region=us-central1
```

### BigQuery Monitoring

Monitor query performance:
```bash
# View recent queries
bq ls -j -a -n 100

# Check dataset size
bq show --format=prettyjson flight_data_dev
```

### GitHub Actions Monitoring

View workflow runs:
```
GitHub Repository → Actions → Flight Booking CICD
```

## 🔧 Troubleshooting

### Common Issues

#### 1. Composer Environment Not Found

**Error:**
```
ERROR: No such environment found: airflow-dev
```

**Solution:**
```bash
# List available environments
gcloud composer environments list --locations=us-central1

# Update workflow with correct environment name
```

#### 2. Permission Denied Errors

**Error:**
```
PERMISSION_DENIED: Organization Policy API has not been used
```

**Solution:**
```bash
# Enable required APIs
gcloud services enable orgpolicy.googleapis.com
gcloud services enable composer.googleapis.com
gcloud services enable dataproc.googleapis.com

# Grant permissions to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:SERVICE_ACCOUNT_EMAIL" \
    --role="roles/composer.worker"
```

#### 3. GCS File Not Found

**Error:**
```
GCSObjectExistenceSensor timeout
```

**Solution:**
- Verify file path: `gs://bucket/path/to/file.csv`
- Check file exists: `gsutil ls gs://bucket/path/`
- Ensure correct environment variable in variables.json

#### 4. Dataproc Batch Fails

**Error:**
```
Spark job failed with exit code 1
```

**Solution:**
```bash
# Check Dataproc logs
gcloud dataproc batches describe BATCH_ID --region=us-central1

# View detailed logs
gcloud logging read "resource.type=cloud_dataproc_batch AND resource.labels.batch_id=BATCH_ID" \
    --limit 50 \
    --format json
```

#### 5. BigQuery Write Errors

**Error:**
```
Access Denied: BigQuery BigQuery: Permission denied
```

**Solution:**
```bash
# Grant BigQuery permissions to Dataproc service account
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:DATAPROC_SA@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/bigquery.dataEditor"
```

### Debug Commands

```bash
# Check Composer environment health
gcloud composer environments describe airflow-gds --location us-central1

# View Airflow logs
gcloud composer environments run airflow-gds \
    --location us-central1 \
    logs -- -f

# List Dataproc batches
gcloud dataproc batches list --region=us-central1 --limit=10

# Verify GCS permissions
gsutil ls gs://airflow-projects-gds-03012026/

# Test BigQuery access
bq ls flight_data_dev
```

## 🤝 Contributing

### Development Workflow

1. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make changes and test locally**
   ```bash
   # Test Spark job locally
   python spark_job/spark_transformation_job.py \
       --env=dev \
       --bq_project=PROJECT_ID \
       --bq_dataset=flight_data_dev \
       --transformed_table=transformed_flight_data \
       --route_insights_table=route_insights \
       --origin_insights_table=booking_origin_insights
   ```

3. **Commit changes**
   ```bash
   git add .
   git commit -m "feat: add new transformation logic"
   ```

4. **Push to develop branch for testing**
   ```bash
   git push origin feature/your-feature-name
   # Create PR to develop branch
   ```

5. **After testing, merge to main for production**
   ```bash
   # Merge develop → main via PR
   ```

### Code Standards

- Follow PEP 8 for Python code
- Use meaningful variable names
- Add comments for complex logic
- Update documentation for new features
- Test in dev before deploying to prod

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.



## 🙏 Acknowledgments

- Apache Airflow community
- Google Cloud Dataproc documentation
- GitHub Actions examples



**Version:** 1.0.0  
**Last Updated:** January 3, 2026  
**Status:** Production Ready ✅
