[![AWS](https://img.shields.io/badge/AWS-Cost%20Monitoring-orange?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Grafana](https://img.shields.io/badge/Dashboards-Grafana-F46800?logo=grafana)](https://grafana.com)
[![Athena](https://img.shields.io/badge/Data-Athena-0073bb?logo=amazon-aws)](https://aws.amazon.com/athena/)
[![CloudWatch](https://img.shields.io/badge/Metrics-CloudWatch-2C2D72?logo=amazon-aws)](https://aws.amazon.com/cloudwatch/)
[![Glue](https://img.shields.io/badge/ETL-Glue-7D3C98?logo=amazon-aws)](https://aws.amazon.com/glue/)
[![FinOps](https://img.shields.io/badge/Focus-FinOps-blue)](https://www.finops.org/)
[![CI](https://img.shields.io/github/actions/workflow/status/lauj0rge/aws-cost-observability/ci.yml?label=CI%20Build)](https://github.com/lauj0rge/aws-cost-observability/actions)

# aws-cost-observability

[![AWS](https://img.shields.io/badge/AWS-Cost%20Monitoring-orange?logo=amazon-aws\&logoColor=white)](https://aws.amazon.com/)
[![Grafana](https://img.shields.io/badge/Dashboards-Grafana-F46800?logo=grafana)](https://grafana.com)
[![Athena](https://img.shields.io/badge/Data-Athena-0073bb?logo=amazon-aws)](https://aws.amazon.com/athena/)
[![CloudWatch](https://img.shields.io/badge/Metrics-CloudWatch-2C2D72?logo=amazon-aws)](https://aws.amazon.com/cloudwatch/)
[![FinOps](https://img.shields.io/badge/Focus-FinOps-blue)](https://www.finops.org/)

A modular collection of **Grafana dashboards and CloudFormation templates** to visualize and analyze AWS cost and performance data.
This project extends the original [COAST](https://github.com/aws-samples/coast-grafana-cost-intelligence-dashboards) framework, streamlined for easier deployment, clarity, and practical FinOps use.

---

## Overview

**aws-cost-observability** unifies AWS **Cost and Usage Report (CUR)** data with **CloudWatch metrics** through **Amazon Managed Grafana** and **Athena**.
It provides engineering teams with visibility into both **cost efficiency** and **resource performance**, promoting accountability and optimization across accounts and business units.

### Why It Matters

* Tracks **cost vs. performance** in real time.
* Promotes **shared cost visibility** for engineers and finance.
* Integrates into **Amazon Managed Grafana** or your existing Grafana stack.
* Uses only **native AWS services** (S3, Athena, CloudWatch, Glue, Grafana).

---

## Architecture

```
AWS CUR (S3)
     ↓
AWS Glue  →  Athena  →  Grafana Dashboards
     ↑             ↑
 CloudWatch ───────┘
```

This architecture runs inside a **centralized monitoring account**, aggregating CUR and CloudWatch data from multiple linked AWS accounts.

---

## Setup

### 1. Prepare CUR Data

Use the **Cloud Intelligence Dashboards (CID)** setup to centralize CUR data:

* [Create CUR destination bucket](https://docs.aws.amazon.com/guidance/latest/cloud-intelligence-dashboards/deployment-in-global-regions.html#deploy-in-global-regions-create-destination-for-cur)
* [Enable CUR 2.0 replication](https://docs.aws.amazon.com/guidance/latest/cloud-intelligence-dashboards/deployment-in-global-regions.html#deploy-in-global-regions-create-cur-and-replication)

### 2. Configure Cross-Account CloudWatch Metrics

Enable CloudWatch **cross-account observability** so performance metrics from multiple AWS accounts flow into the same workspace.
See [AWS docs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Unified-Cross-Account.html) for step-by-step setup.

### 3. Deploy via CloudFormation

Deploy the COAST CloudFormation stack in your **monitoring account**:

```bash
aws cloudformation deploy \
  --template-file cloudformation/provision-coast-services.yaml \
  --stack-name cost-observability \
  --capabilities CAPABILITY_NAMED_IAM
```

This creates the **Amazon Managed Grafana workspace**, Athena plugin, policies, and CUR data sources.

---

## Grafana Configuration

1. Add an **IdP** (recommended: AWS IAM Identity Center).
2. Log into the Grafana workspace using your configured identity.
3. Import dashboards from the `grafana_dashboards/` directory.
4. Update Athena table variables (`CURTable`) as needed.
5. Optionally, create **views per business unit** for access segmentation.

Example:

```sql
CREATE OR REPLACE VIEW engineering_team_view AS 
SELECT *
FROM cid_data_export.cur2
WHERE line_item_usage_account_id IN ('111222333444', '222333444555');
```

---

## Dashboards

| Dashboard                 | Description                                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| **Amazon EC2**            | Cost, usage, and performance metrics per account, region, and tag.                          |
| **Amazon Athena**         | Visualizes Athena query cost vs performance metrics (ProcessedBytes, ExecutionTime).        |
| **Auto Scaling**          | Combines autoscaling group cost + CloudWatch metrics.                                       |
| **Amazon EKS Split Cost** | Merges CUR split cost data with CloudWatch container insights for full EKS cost visibility. |

All dashboards are JSON-based and compatible with Amazon Managed Grafana or self-hosted Grafana.

---

## Cost Awareness

You pay only for the AWS services used:

* **Grafana**: [Amazon Managed Grafana pricing](https://aws.amazon.com/grafana/pricing/)
* **Athena**: [Query pricing](https://aws.amazon.com/athena/pricing/)
* **CloudWatch**: [Standard metrics pricing](https://aws.amazon.com/cloudwatch/pricing/)

Typical cost range: **$10–$300/month**, depending on CUR size and dashboard usage.

---

## Extend

Add dashboards for:

* **AWS Glue** and **S3** costs
* **Business Unit split CUR views**
* **Anomaly Detection** using Athena queries

Integrate results into existing FinOps workflows or automated reports.

---

## Acknowledgement

This project builds upon the AWS open-source [COAST dashboards](https://github.com/aws-samples/coast-grafana-cost-intelligence-dashboards).
All content adapted for educational and practical reuse in DevOps and FinOps environments.

