---
name: aws-billing-report
description: Generate professional AWS billing Excel reports with detailed monthly cost breakdowns matching AWS Bills page exactly, including Data Transfer and all usage types. Use when users request AWS billing reports, cost exports, monthly cost reports, or mention keywords like "aws billing", "aws费用", "billing report", "kolya billing", "生成AWS账单报表", "导出AWS费用". Uses Cost and Usage Report (CUR) for most detailed billing data.
---

# AWS Billing Report

## Overview

Generate professionally formatted Excel reports with **detailed** AWS billing data that matches the AWS Bills page exactly:
- **Sheet1**: Service-level summary with monthly costs
- **Sheet2**: Detailed drill-down with three-level hierarchy (Service → Region → Usage Type)
- Complete breakdown including Data Transfer, EC2 instances, and all usage types
- Year-over-year comparison with growth percentages
- Microsoft 365 professional styling (Aptos font, light gray headers)
- Matches the AWS Console Bills page format exactly

**Data Source:** Cost and Usage Report (CUR) - provides the most detailed billing information available from AWS.

**Full automation:** All prerequisites checking, CUR setup, data fetching, and report generation are handled automatically through Claude Code interaction.

## Usage

### Automated Workflow

When the user requests a billing report, Claude Code will automatically:

#### Step 1: Collect Required Information

Ask the user for the following information if not provided:

```
To generate the AWS billing report with detailed breakdown (including Data Transfer), I need:
1. AWS account ID (e.g., 123456789012)
2. AWS profile name (optional - will use AWS_PROFILE environment variable or default credentials if not provided)
3. Year(s) for the report (e.g., 2026, or multiple years like 2025,2026)
4. Output path for the Excel report (e.g., /Users/username/ or /Users/username/Documents/aws_billing_2026.xlsx)
```

**Note:** If AWS profile name is not provided, the script will attempt to use:
1. `AWS_PROFILE` environment variable
2. Default AWS credentials configured via `aws configure`

#### Step 2: Auto-Check and Setup CUR

Claude Code will automatically check if CUR is configured:

```bash
python scripts/setup_cur.py --check-only
```

**If CUR is already configured:**
- ✓ Proceed to Step 3

**If CUR is not configured:**
- Claude will automatically run the setup:
  ```bash
  python scripts/setup_cur.py
  ```
- Setup takes ~5 minutes and creates:
  - S3 bucket: `aws-cur-{account_id}`
  - CUR report definition with Parquet format
  - Daily update schedule
  - Location: `s3://aws-cur-{account_id}/cur/billing-report-detailed/`

**Important:** ⏰ First CUR report takes **24 hours** to generate after initial setup. Claude will inform the user and stop here if this is the first setup.

```
✓ CUR configuration complete!

⏰ IMPORTANT: The first billing report will be generated in 24 hours.
   After that, reports update daily automatically.

Location: s3://aws-cur-{account_id}/cur/billing-report-detailed/

Please run this skill again after 24 hours to generate your report.
```

#### Step 3: Auto-Fetch Billing Data

Claude Code will automatically fetch detailed billing data from CUR:

```bash
python scripts/fetch_aws_billing.py <account_id> <year> --profile <profile_name> > billing_<year>.json 2>/dev/null
```

The script will:
- Read Parquet files from S3 CUR location
- Extract all usage types (Data Transfer, EC2 instances, S3 storage, etc.)
- Aggregate by month and service
- Create detailed drill-down records for Sheet2
- Output JSON format compatible with Excel generator

**For multi-year reports:**
Claude will fetch data for each year and combine them for year-over-year comparison.

#### Step 4: Auto-Generate Excel Report

Claude Code will automatically generate the professionally formatted Excel file:

```bash
python scripts/generate_excel_report.py <output_path> billing_<year1>.json [billing_<year2>.json ...]
```

The generated report contains:

**Sheet1 - Service Summary:**
- Service-level costs for each month
- Clean service names matching AWS Bills page
- Includes Data Transfer as a separate service
- Total row with auto-calculated formulas

**Sheet2 - Usage Details:**
- Three-level drill-down hierarchy:
  - **Level 1**: Service (e.g., "Data Transfer ($0.34)")
  - **Level 2**: Region (e.g., "US West (Oregon)")
  - **Level 3**: Usage Type (e.g., "AWS Data Transfer USW2-DataTransfer-Regional-Bytes")
- For each usage type:
  - Description (matching AWS Bills page)
  - Quantity with unit (e.g., "18.387 GB")
  - Rate with unit (e.g., "$0.0100 per GB")
  - Cost

#### Step 5: Deliver Report

Claude Code will automatically:
1. Save the report to the specified output path
2. Verify the file was created successfully
3. Display a summary of the report contents

```
✅ AWS Billing Report Generated Successfully!

📁 File: /Users/username/aws_billing_2026.xlsx (15.8 KB)
📊 Account: 123456789012
📅 Year: 2026
💰 Total Cost (Feb): $675.06 USD

📋 Sheet1 - Service Summary:
   • 22 AWS services
   • Monthly cost breakdown (Jan-Dec)
   • Includes Data Transfer ($0.34)

🔍 Sheet2 - Usage Details:
   • Grouped by service and region
   • 105 detailed usage records
   • Includes: Usage Type, description, quantity, rate, cost

💡 Top 5 Services (Feb):
   1. OpenSearch Service            $126.17
   2. Elastic Container Service     $104.39
   3. QuickSight                    $99.48
   4. Bedrock Service               $94.14
   5. Relational Database Service   $63.15

✅ Format matches AWS Bills page exactly
✅ Professional Microsoft 365 style (Aptos font)
✅ Three-level drill-down: Service → Region → Usage Type
✅ Data accuracy: 99.94%
```

#### Step 6: Auto-Cleanup Temporary Files

Claude Code will automatically clean up intermediate files after report generation:

```bash
rm -f billing_*.json
```

**Files cleaned:**
- `billing_<year>.json` - Intermediate billing data files
- Any other temporary JSON files generated during processing

**Note:** The final Excel report file (`aws_billing_<year>.xlsx`) is saved to your specified output path and is NOT cleaned up.

```
🧹 Cleaning up temporary files...
✅ Cleanup complete
```

## Report Structure

### Sheet1 - Service Summary

Each row represents one AWS service, with columns for each month:

| Description | Jan | Feb | Mar | ... | Dec | Total |
|------------|-----|-----|-----|-----|-----|-------|
| OpenSearch Service | 0.00 | 126.17 | 0.00 | ... | 0.00 | =SUM(B2:M2) |
| Elastic Container Service | 0.00 | 104.39 | 0.00 | ... | 0.00 | =SUM(B3:M3) |
| QuickSight | 0.00 | 99.48 | 0.00 | ... | 0.00 | =SUM(B4:M4) |
| Data Transfer | 0.00 | 0.34 | 0.00 | ... | 0.00 | =SUM(B5:M5) |
| ... | ... | ... | ... | ... | ... | ... |
| Total | =SUM(B2:B22) | =SUM(C2:C22) | ... | ... | =SUM(N2:N22) |

**Service names are cleaned** to match AWS Bills page:
- "AmazonES" → "OpenSearch Service"
- "AmazonECS" → "Elastic Container Service"
- "AWSLambda" → "Lambda"
- "AWSELB" → "Elastic Load Balancing"

### Sheet2 - Usage Details (Drill-Down)

Three-level hierarchy matching AWS Bills page:

```
--- Data Transfer ($0.34) ---
  US West (Oregon)                                      $0.34
    AWS Data Transfer USW2-DataTransfer-Regional-Bytes
      $0.010 per GB - regional data transfer...    18.387 GB    $0.18
    AWS Data Transfer USW2-DataTransfer-Regional-Bytes
      $0.010 per GB - regional data transfer...    9.969 GB     $0.10
    AWS Data Transfer USW2-DataProcessing-Bytes
      $0.50 per GB - custom log data ingested...   0.085 GB     $0.04

--- OpenSearch Service ($126.17) ---
  US West (Oregon)                                      $126.17
    USW2-IndexingOCU
      $0.24 per OCU-hours for IndexingOCU...       192.000      $46.08
    USW2-SearchOCU
      $0.24 per OCU-hours for SearchOCU...         192.000      $46.08
```

**Columns:**
- Service: Service name (shown in section header)
- Region: AWS region (e.g., "US West (Oregon)")
- Usage Type: Specific usage type with service prefix
- Description: Full description from AWS Bills
- Quantity: Amount used with unit (e.g., "18.387 GB", "192.000")
- Rate: Unit price (e.g., "$0.0100 per GB", "$0.24")
- Cost: Total cost for this usage type

## Prerequisites

The following are automatically handled by Claude Code during execution:

1. **AWS Credentials**: User must have AWS profile configured with appropriate permissions
2. **Python Packages**: boto3, pyarrow, pandas, openpyxl (Claude will check and prompt to install if missing)
3. **CUR Configuration**: Automatically checked and configured by Claude if needed
4. **Permissions Required**:
   - S3: CreateBucket, PutBucketPolicy, ListBucket, GetObject
   - CUR: PutReportDefinition, DescribeReportDefinitions
   - Billing: Cost and Usage Report read access

## Error Handling

Claude Code will automatically handle common errors:

**Missing AWS credentials:**
- Check if profile exists
- Prompt user to run `aws sso login` if needed

**CUR not configured:**
- Automatically run setup
- Inform user about 24-hour wait for first report

**Missing Python packages:**
- Display installation command
- Prompt user to install required packages

**No CUR data available:**
- Check if 24 hours have passed since CUR setup
- Inform user to wait longer

**Invalid account ID or permissions:**
- Display clear error message
- Suggest checking account ID and IAM permissions

## Technical Reference

For developers or advanced users who need to understand the underlying scripts:

### Script: setup_cur.py
- **Purpose**: One-time CUR configuration
- **Creates**: S3 bucket, bucket policy, CUR report definition
- **Format**: Parquet with daily updates
- **Location**: `s3://aws-cur-{account_id}/cur/billing-report-detailed/`

### Script: fetch_aws_billing.py
- **Purpose**: Fetch and aggregate billing data from CUR
- **Input**: Account ID, year, AWS profile
- **Output**: JSON with summary, by_service, by_usage_type, usage_details
- **Authentication**: Supports --profile flag, AWS_PROFILE env var, or default credentials

### Script: generate_excel_report.py
- **Purpose**: Generate formatted Excel report
- **Input**: Output path, one or more billing JSON files
- **Output**: Excel file with Service Summary and Usage Details sheets
- **Features**: Professional styling, auto-calculated formulas, cleaned service names

## Tips

- **First-time setup**: CUR configuration takes ~5 minutes + 24 hours for first report generation
- **Daily updates**: After initial setup, CUR data updates automatically every day
- **Multi-year analysis**: Provide multiple years (e.g., "2025,2026") to get year-over-year comparison
- **Large accounts**: Processing may take 2-5 minutes for accounts with many services
- **S3 costs**: CUR storage costs ~$0.01-0.10 per month
- **Currency**: Reports automatically detect currency from AWS (typically USD)
- **Accuracy**: Report data typically matches AWS Bills page within 0.1% (due to rounding)

## Troubleshooting

**Q: CUR data seems outdated**
A: CUR updates daily. Data may be 1-2 days old depending on AWS processing time.

**Q: Some services missing from report**
A: Ensure CUR has been running for the full month. First month may have incomplete data.

**Q: Excel formulas not working**
A: Enable automatic calculation in Excel: Formulas → Calculation Options → Automatic

**Q: Year-over-year shows "N/A"**
A: This occurs when previous year had $0 cost (prevents division by zero).

**Q: Data Transfer amount differs slightly from AWS Bills**
A: Minor differences (<0.1%) are due to rounding and timing of CUR data processing.
