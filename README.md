# 🚀 GCP Cost Agent: Your AI-Powered GCP Cost Guru

**A smart AI agent that analyzes your Google Cloud spending by querying BigQuery billing data.**

This open-source Python project, built with the Google Agents Development Kit (ADK), provides an interactive agent powered by the Gemini-1.5-flash model. It connects directly to your GCP billing data in BigQuery to answer natural language questions about your cloud expenditures. Ask for your total monthly costs, get breakdowns by project or service, and receive clear, formatted responses—all from your terminal.

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/downloads/release/python-3100/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/your-username/gcp-cost-agent.svg?style=social)](https://github.com/your-username/gcp-cost-agent/stargazers)

---

## ✨ Features

*   **Natural Language Queries:** Ask questions about your GCP costs in plain English (e.g., "What was our total bill for July 2023?").
*   **Detailed Cost Analysis:** Get cost breakdowns by:
    *   **Total Monthly Cost:** View the sum of costs and credits for any given month.
    *   **Project:** See which projects are contributing the most to your bill.
    *   **Service:** Understand which Google Cloud services are being utilized the most.
*   **Intelligent Tool Use:** The agent automatically selects the right BigQuery tool based on your query, whether you're asking for a summary or a detailed breakdown.
*   **Multi-Currency Support:** Reports costs with the currency provided in your billing data.
*   **Smart Error Handling:** Provides helpful feedback if you query for a month with no billing data or use an invalid format.
*   **Extensible & Customizable:**
    *   Easily modify or add new BigQuery queries in `tools.yaml`.
    *   Refine the agent's behavior and instruction set in `agent.py`.
*   **Built with Google ADK:** Leverages the latest from Google for building robust and scalable AI agents.

---

## 📋 Prerequisites

Before you begin, you'll need to set up your Google Cloud environment and a local Python environment.

### 1. Google Cloud Platform (GCP) Setup

The agent relies on having your GCP billing data exported to a BigQuery table.

**Step 1: Enable Billing Export to BigQuery**

You must have billing export enabled so that your detailed usage and cost data is sent to a BigQuery dataset.

1.  Go to the **Billing** section in your Google Cloud Console.
2.  Select your billing account.
3.  In the navigation menu, click on **Billing export**.
4.  Click **Edit settings** for **Detailed usage cost**.
5.  Choose a **Project** to host the BigQuery dataset.
6.  Select an existing **Dataset** or create a new one. The agent assumes a table name like `gcp_billing_export_v1_XXXXXX_XXXXXX_XXXXXX`, which is the standard format.
7.  Click **Save**.

It may take a few hours for the first export to appear.

**Step 2: Create a Service Account**

For security best practices, the agent should authenticate using a service account with limited permissions.

1.  In the Cloud Console, go to **IAM & Admin > Service Accounts**.
2.  Click **+ CREATE SERVICE ACCOUNT**.
3.  Give it a name (e.g., `gcp-cost-agent-sa`).
4.  Grant it the following roles:
    *   `BigQuery Data Viewer`: To read data from the billing export table.
    *   `BigQuery Job User`: To run queries.
5.  Create a key for the service account (JSON format) and download it to a secure location on your machine.

**Step 3: Set Up Authentication**

Set the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to the path of the JSON key file you downloaded.

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your/keyfile.json"
```

### 2. Python Environment

*   **Python 3.10 or higher** is required.
*   `pip` for package management.

---

## 🚀 Installation & Quick Start

Get the agent running in just a few steps.

**1. Clone the Repository**

```bash
git clone https://github.com/your-username/gcp-cost-agent.git
cd gcp-cost-agent
```

**2. Install Dependencies**

A `requirements.txt` file is included with all necessary packages.

```bash
pip install -r requirements.txt
```
*(Note: If `requirements.txt` is not yet created, you will need `google-adk-agents`, `toolbox-core`, `google-cloud-bigquery`, and `python-dotenv`)*

**3. Configure Your Environment**

Copy the example environment file and fill in your GCP project details.

```bash
cp .env.example .env
```

Now, open `.env` and update it with your BigQuery billing export table details.

```dotenv
# .env
# The full path to your BigQuery table
# e.g., your-gcp-project.your_dataset.gcp_billing_export_v1_XXXXXX_XXXXXX_XXXXXX
GCP_BILLING_TABLE="your-gcp-project.your_dataset.gcp_billing_export_v1_XXXXXX_XXXXXX_XXXXXX"
```

**4. Run the Agent!**

You're all set! Start the agent by running:

```bash
python agent.py
```

The agent will initialize and greet you with a message. You can now start asking questions.

**Example Test Query:**

```
>> What was the total cost for 202307?
```

If everything is configured correctly, the agent will execute a query against your BigQuery table and return the total cost.

---

## 💬 Usage

Interact with the agent by typing your questions directly into the terminal. The agent understands time periods formatted as `YYYYMM`.

### Example 1: Get Total Monthly Cost

**Query:**
```
>> Show me the total cost for August 2023.
```

**Agent Interaction:**
The agent detects the month (`202308`) and uses the `get_monthly_cost_summary` tool.

**Output:**
```
The total cost for the month 202308 was 1234.56 USD.
```

### Example 2: Get Cost Breakdown by Project

**Query:**
```
>> What were the most expensive projects in 202308?
```

**Agent Interaction:**
The agent uses the `get_cost_by_project` tool for the specified month.

**Output:**
```
Here is the cost breakdown by project for 202308:
1.  Project `production-environment` (ID: `prj-prod-123`): 500.25 USD
2.  Project `staging-environment` (ID: `prj-staging-456`): 350.10 USD
3.  Project `dev-testing` (ID: `prj-dev-789`): 150.00 USD
...and so on.
```

### Example 3: Handling No Data

If you query a month with no billing information, the agent provides a clear message.

**Query:**
```
>> What was the cost for 202501?
```

**Agent Interaction:**
The tool returns an empty result.

**Output:**
```
I could not find any billing data for the invoice month 202501. Please check the month or try a different one.
```

---

## 🛠️ Customization

The agent is designed to be easily extended.

### Editing `tools.yaml`

This file is the heart of the agent's capabilities. You can modify existing queries or add new ones.

**To change the BigQuery table:**

If your table name doesn't match the one in your `.env` file, you can update the `table_name` placeholder in each tool's query.

**To add a new tool:**

1.  Define a new tool with a unique name (e.g., `get_cost_by_sku`).
2.  Write the BigQuery SQL query. Use placeholders like `{invoice_month}` for dynamic parameters.
3.  Provide a clear `description` so the agent knows when to use your new tool.

**Example - New Tool to Get Cost by SKU:**
```yaml
# tools.yaml
- tool: get_cost_by_sku
  description: "Provides a cost breakdown by SKU for a given invoice month."
  args:
    - invoice_month
  type: bq_sql_query
  query: |
    SELECT
      sku.description,
      SUM(cost) as total_cost,
      currency
    FROM `{table_name}`
    WHERE invoice.month = '{invoice_month}'
    GROUP BY 1, 3
    ORDER BY total_cost DESC
    LIMIT 10;
```

### Extending Agent Instructions

The agent's core logic and instructions are in `agent.py`. You can modify the `instructions` variable to change its personality, response format, or how it handles specific edge cases.

For example, you could instruct it to always round to the nearest dollar or to handle multi-currency results differently.

---

## 🔍 Troubleshooting

*   **Error: `google.api_core.exceptions.Forbidden: 403`**
    *   **Cause:** Your service account or user credentials do not have the required IAM permissions (`BigQuery Data Viewer`, `BigQuery Job User`).
    *   **Solution:** Ensure the `GOOGLE_APPLICATION_CREDENTIALS` environment variable is set correctly and that the service account has the right roles.

*   **Agent Response: "I could not find any billing data..."**
    *   **Cause:** The `invoice_month` you provided does not exist in the billing table, or the table was empty for that period.
    *   **Solution:** Double-check that you are using the `YYYYMM` format and that billing data exists for that month in your BigQuery table.

*   **Error on Startup: `FileNotFoundError: .env`**
    *   **Cause:** You did not create the `.env` file.
    *   **Solution:** Run `cp .env.example .env` and fill in your project details.

---

## 🤝 Contributing

We welcome contributions! Please follow these steps to contribute:

1.  **Fork the Repository:** Create your own fork of the project.
2.  **Create a Branch:** Make your changes in a dedicated branch.
    ```bash
    git checkout -b my-new-feature
    ```
3.  **Add Tests:** If you add new functionality, please include tests using `pytest`.
4.  **Run CI Checks:** Ensure your code passes linting and testing checks.
5.  **Create a Pull Request:** Submit a PR with a clear description of your changes.

---

## 📜 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.
