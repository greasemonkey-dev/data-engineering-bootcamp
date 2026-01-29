# Chapter 10: Infrastructure as Code & CI/CD

## Learning Objectives
By the end of this chapter, you will be able to:
1. **Implement** Infrastructure as Code using Terraform to provision cloud resources in a reproducible, version-controlled manner
2. **Design** CI/CD pipelines for data infrastructure that automatically test, validate, and deploy changes
3. **Create** automated testing strategies for data pipelines including schema validation, data quality checks, and integration tests
4. **Apply** GitOps principles to manage data infrastructure and pipeline deployments through pull requests and code review

## Introduction

It's Monday, 9:15 AM. New team member's first day. I'm giving him access to our data platform.

"Okay, so to set up the development environment, first go to the GCP console..."

I open my browser. Navigate to Cloud Composer. Click through 14 menus to find the right settings. Screenshot them. Send to him.

"Now create a Cloud SQL instance. Make sure you use PostgreSQL 13, not 14. Machine type should be db-n1-standard-2. Storage is 100GB. Backups daily at 3 AM. High availability enabled..."

I'm literally reading from a 47-point checklist I maintain in a Google Doc. Every time GCP changes their UI, I update the screenshots. I've updated it 23 times this year.

"How long does this take?" he asks.

"Oh, about 4-5 hours if you don't make mistakes. 2 days if you do."

He looks horrified.

My manager walks by. "Why are you doing that manually? Don't we have Terraform?"

"We have what?"

"Terraform. Infrastructure as Code. You define infrastructure in config files, run one command, everything gets created. Takes 10 minutes."

"That... exists?"

I'd spent the last year clicking through GCP console creating resources one by one. Manually. Like a caveman with a mouse.

That afternoon, I learned about Infrastructure as Code. By Friday, I'd automated our entire infrastructure setup. From 2 days of clicking to 10 minutes of running `terraform apply`.

Here's what nobody tells you: If you're clicking in the cloud console, you're doing it wrong.

## Section 1: Terraform - The Day I Stopped Clicking

### The Production Resource I Accidentally Deleted (Because I Couldn't Remember What I Clicked)

Let me tell you about the worst Tuesday of my career.

We had a production BigQuery dataset. Critical data. 47 tables. Complex IAM permissions. The BI team relied on it for all their dashboards.

Something broke. I couldn't remember how I'd set up the permissions. I checked my notes. "Give analytics team access." That's it. That's all I wrote six months ago.

What permissions? Editor? Viewer? Custom role? Which service accounts? I had no idea.

So I did what any desperate engineer does: I tried to recreate the permissions based on what seemed right.

I clicked. I granted. I revoked. I tested.

"Did that fix it?"

"No."

More clicking. More testing.

Then: "Uh, now I can't see the tables at all."

Oh no.

I'd somehow deleted the permissions that gave me access. To a production dataset. That I was the admin of.

I spent the next 6 hours:
1. Begging GCP support to restore my access (3 hours)
2. Trying to remember what permissions existed before (impossible)
3. Recreating everything from scratch (2 hours)
4. Testing with angry BI team members (1 hour)

The worst part? I still wasn't sure I'd recreated it correctly. I just kept clicking until people stopped complaining.

If I'd had Infrastructure as Code, I could've just run `terraform apply` and restored everything in 2 minutes.

### Here's What I Wish I'd Known: Code > Clicking

**Infrastructure as Code (IaC):** Define infrastructure in configuration files instead of clicking in consoles.

**Benefits:**
- **Reproducible:** Run the same code, get the same infrastructure
- **Version controlled:** See history of all changes
- **Documented:** The code is the documentation
- **Testable:** Validate before applying
- **Collaborative:** Code review infrastructure changes

**Terraform** is the most popular IaC tool. It works with AWS, GCP, Azure, and 100+ other providers.

### Example: Creating Infrastructure with Terraform

**The old way (clicking):**
1. Open GCP Console
2. Navigate to BigQuery → Create Dataset
3. Enter name: `analytics_prod`
4. Region: `us-central1`
5. Default table expiration: 30 days
6. Click "Create"
7. Navigate to IAM
8. Click "Add Member"
9. Enter email: `analytics-team@company.com`
10. Role: `BigQuery Data Viewer`
11. Click "Save"
12. Repeat for 6 more service accounts...
13. 45 minutes later, you're done
14. Tomorrow, try to remember what you did

**The new way (Terraform):**

```hcl
# main.tf - Define everything in code

# BigQuery dataset
resource "google_bigquery_dataset" "analytics_prod" {
  dataset_id = "analytics_prod"
  location   = "us-central1"

  default_table_expiration_ms = 2592000000  # 30 days

  access {
    role          = "READER"
    user_by_email = "analytics-team@company.com"
  }

  access {
    role          = "READER"
    user_by_email = "dashboard-service@company.iam.gserviceaccount.com"
  }

  access {
    role          = "WRITER"
    user_by_email = "etl-pipeline@company.iam.gserviceaccount.com"
  }

  labels = {
    environment = "production"
    team        = "data-engineering"
  }
}

# BigQuery table
resource "google_bigquery_table" "user_events" {
  dataset_id = google_bigquery_dataset.analytics_prod.dataset_id
  table_id   = "user_events"

  time_partitioning {
    type  = "DAY"
    field = "event_timestamp"
  }

  clustering = ["country", "event_type"]

  schema = file("schemas/user_events.json")
}
```

Run it:
```bash
terraform init      # Initialize
terraform plan      # Preview changes
terraform apply     # Create resources
```

**Result:** Everything created in 2 minutes. And now it's in git. Forever documented. Can recreate anytime.

### Terraform Best Practices

#### Pattern 1: Organize by Environment

```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   └── terraform.tfvars  # dev-specific values
│   ├── staging/
│   │   ├── main.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       └── terraform.tfvars
└── modules/
    ├── bigquery/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── composer/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

Same code, different values per environment.

#### Pattern 2: Use Modules for Reusability

```hcl
# modules/bigquery_dataset/main.tf
variable "dataset_id" {}
variable "location" {}
variable "readers" { type = list(string) }

resource "google_bigquery_dataset" "dataset" {
  dataset_id = var.dataset_id
  location   = var.location

  dynamic "access" {
    for_each = var.readers
    content {
      role          = "READER"
      user_by_email = access.value
    }
  }
}

# Use the module
module "analytics_dataset" {
  source = "./modules/bigquery_dataset"

  dataset_id = "analytics"
  location   = "us-central1"
  readers    = [
    "analytics-team@company.com",
    "bi-dashboard@company.iam.gserviceaccount.com"
  ]
}
```

#### Pattern 3: Store State Remotely

```hcl
# backend.tf - Store state in GCS, not locally
terraform {
  backend "gcs" {
    bucket = "company-terraform-state"
    prefix = "data-platform/prod"
  }
}
```

**Why:** If state is local, only you can run Terraform. Remote state enables team collaboration.

### Scars I've Earned: Infrastructure as Code Edition

**Scar #1: Manually modified resources Terraform created**
```bash
# DON'T
# Create with Terraform
terraform apply

# Manually modify in console
# GCP Console: Change dataset location

# Try to update with Terraform
terraform apply
# Terraform wants to recreate! (destroys data)
```
**What it cost me:** Accidentally deleted production dataset, 4-hour restore from backup

**Scar #2: Didn't use remote state, kept it local**
```bash
# DON'T
# My laptop: terraform.tfstate (local file)
# Teammate's laptop: different terraform.tfstate
# Now we have two sources of truth, infrastructure drifts apart
```
**What it cost me:** Two team members created conflicting resources, took a day to untangle

**Scar #3: Didn't review terraform plan before apply**
```bash
# DON'T
terraform apply -auto-approve  # YOLO mode
# Whoops, just deleted production database
```
**What it cost me:** Destroyed staging environment that had test data people needed

## Section 2: CI/CD for Data Pipelines - The Deploy That Broke Everything

### The Manual Deployment That Worked On My Laptop

Friday, 3 PM. I'd just finished a new Airflow DAG. Tested locally. Worked perfectly. Ready to deploy.

My deployment process:
1. SSH into Airflow server
2. `git pull origin main`
3. Restart Airflow scheduler
4. Hope for the best

I ran it. Logged out. Went home.

Sunday morning, 6 AM. Phone rings. Production Airflow is down. All DAGs stopped.

I SSH in. Check logs:

```
ImportError: No module named 'great_expectations'
```

What? Great Expectations is definitely installed. I used it in my new DAG.

On my laptop.

I forgot to add it to `requirements.txt`. My local environment had it from a different project. Production didn't.

Zero DAGs running. Because I deployed untested code directly to production. By SSHing and running git pull.

### Here's What I Wish I'd Known: Deployments Should Be Automated, Not Manual

**CI/CD (Continuous Integration / Continuous Deployment):**
- **CI:** Automatically test every code change
- **CD:** Automatically deploy code that passes tests

**For data pipelines:**
- Test DAG syntax before deploying
- Validate data schemas
- Run data quality checks
- Deploy only if tests pass

### Example: GitHub Actions CI/CD for Airflow

```yaml
# .github/workflows/deploy-airflow.yml
name: Deploy Airflow DAGs

on:
  push:
    branches: [main]
    paths:
      - 'dags/**'
      - 'requirements.txt'

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest

      - name: Test DAG syntax
        run: |
          python -m pytest tests/test_dag_validation.py

      - name: Test DAG integrity
        run: |
          python -c "from airflow.models import DagBag; \
                     dag_bag = DagBag('dags/'); \
                     assert len(dag_bag.import_errors) == 0"

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3

      - name: Deploy to Cloud Composer
        run: |
          gcloud composer environments storage dags import \
            --environment=prod-composer \
            --location=us-central1 \
            --source=dags/

      - name: Notify Slack
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -d '{"text":"✅ Airflow DAGs deployed to production"}'
```

Now:
1. Push to main
2. Tests run automatically
3. If tests pass, deploy automatically
4. If tests fail, deployment blocked
5. Team gets notified

No more SSHing. No more "oops forgot requirements.txt."

### Testing Data Pipelines

```python
# tests/test_etl_pipeline.py
import pytest
from dags.etl_pipeline import transform_user_data

def test_transform_handles_missing_columns():
    """Pipeline should handle missing optional columns"""
    input_data = [
        {'user_id': 1, 'email': 'user@example.com'}
        # Missing 'phone' column
    ]

    result = transform_user_data(input_data)

    assert result[0]['phone'] is None  # Should default to None
    assert result[0]['email'] == 'user@example.com'

def test_transform_rejects_invalid_emails():
    """Pipeline should reject invalid emails"""
    input_data = [
        {'user_id': 1, 'email': 'not-an-email'}
    ]

    with pytest.raises(ValueError, match="Invalid email"):
        transform_user_data(input_data)

def test_dag_has_no_cycles():
    """DAG should not have circular dependencies"""
    from airflow.models import DagBag

    dag_bag = DagBag('dags/')
    assert len(dag_bag.import_errors) == 0

    for dag_id, dag in dag_bag.dags.items():
        # Check for cycles
        assert len(list(dag.task_dict.values())) > 0
```

Run tests before every deploy:
```bash
pytest tests/
# All tests pass? Deploy.
# Any test fails? Block deployment.
```

### Scars I've Earned: CI/CD Edition

**Scar #1: Deployed without testing**
```bash
# DON'T
git push origin main
ssh prod-server
git pull
# Imported module that doesn't exist in prod
# Everything breaks
```
**What it cost me:** Sunday morning emergency, all DAGs down for 3 hours

**Scar #2: No rollback strategy**
```bash
# DON'T
# Deploy new version
# It breaks
# No way to quickly revert
# Spend 2 hours debugging in production
```
**What it cost me:** Production downtime while I fixed bugs that passed "local testing"

**Scar #3: Deployed to production first**
```bash
# DON'T
# Skip staging environment
# Deploy untested code directly to production
# "It worked on my laptop"
```
**What it cost me:** Broke production data quality checks, bad data reached dashboards

## Summary

Modern data engineering requires automation:

- **Infrastructure as Code (Terraform):** Define infrastructure in code, version control everything, no more clicking
- **CI/CD pipelines:** Automatically test and deploy, block bad code from reaching production
- **Automated testing:** Validate DAG syntax, data schemas, transformation logic before deployment
- **GitOps:** All changes via pull requests, code review infrastructure updates

The difference between junior and senior engineers? Seniors automate everything they have to do twice.

## Reflection Questions

1. How much time do you spend manually creating/configuring infrastructure? Could that be automated?

2. Your teammate pushes broken code to production. How would CI/CD have prevented it?

3. What's your rollback strategy if a deployment breaks? Can you revert in 1 minute or 1 hour?

## Next Steps

Next chapter: Capstone Project—time to build an end-to-end data platform using everything you've learned.
