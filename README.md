# Invoice Processing Automation Workflow

## Overview

This n8n workflow automates the complete process of extracting and organizing invoice data from email attachments. The system monitors a specified Gmail account for incoming emails with invoice attachments, processes them using AI-powered document analysis, and automatically updates a Google Sheets database with structured invoice information.

## Workflow Architecture

The workflow consists of 9 interconnected nodes that handle the entire invoice processing pipeline:

### 1. **Gmail Trigger**
- **Type**: `n8n-nodes-base.gmailTrigger`
- **Function**: Monitors Gmail account every minute for new unread emails
- **Configuration**: 
  - Poll interval: Every minute
  - Filter: Unread emails only
  - Credentials: Gmail OAuth2 authentication required

### 2. **Extracting Invoice (Gmail Get)**
- **Type**: `n8n-nodes-base.gmail`
- **Function**: Retrieves the complete email message and downloads attachments
- **Configuration**:
  - Operation: Get message
  - Downloads attachments with prefix "attachment_"
  - Processes message based on trigger ID

### 3. **Analyze Document (Google Gemini)**
- **Type**: `@n8n/n8n-nodes-langchain.googleGemini`
- **Function**: Uses Google Gemini 2.5 Flash model to analyze invoice documents
- **Configuration**:
  - Model: `models/gemini-2.5-flash`
  - Processes binary attachment data
  - Custom prompt for structured invoice data extraction

### 4. **Information Extractor**
- **Type**: `@n8n/n8n-nodes-langchain.informationExtractor`
- **Function**: Standardizes and validates extracted data format
- **Configuration**:
  - Uses predefined JSON schema for consistent output
  - Handles missing values appropriately
  - Works with Google Gemini Chat Model

### 5. **Google Gemini Chat Model**
- **Type**: `@n8n/n8n-nodes-langchain.lmChatGoogleGemini`
- **Function**: Provides AI language model support for information extraction
- **Configuration**: Connected to Information Extractor node

### 6. **Code Node**
- **Type**: `n8n-nodes-base.code`
- **Function**: Transforms complex line items array into Google Sheets-compatible format
- **Configuration**:
  - Flattens nested JSON structure
  - Converts line items array to concatenated string
  - Handles null values appropriately

### 7. **Append Row in Sheet (Google Sheets)**
- **Type**: `n8n-nodes-base.googleSheets`
- **Function**: Adds extracted invoice data to Google Sheets
- **Configuration**:
  - Operation: Append new row
  - Document ID: `1ITgzlN5MlE5P0oN0X4MKdISXpozVhVgUrm9ggunNUsI`
  - Auto-maps input data to sheet columns

### 8. **Merge Node**
- **Type**: `n8n-nodes-base.merge`
- **Function**: Combines original email data with processed invoice data
- **Configuration**: Merges data streams for notification email

### 9. **Gmail2 (Send Notification)**
- **Type**: `n8n-nodes-base.gmail`
- **Function**: Sends confirmation email with original invoice attachment
- **Configuration**:
  - Recipient: `hossam.elesawy@student.guc.edu.eg`
  - Subject: "Invoices sheet Update"
  - Includes original attachment

## Data Structure

The workflow extracts and organizes the following invoice fields:

| Field Name | Data Type | Description |
|------------|-----------|-------------|
| `invoice_number` | String | Unique invoice identifier |
| `invoice_date` | Date (YYYY-MM-DD) | Invoice issue date |
| `due_date` | Date (YYYY-MM-DD) | Payment due date |
| `vendor_name` | String | Supplier/vendor company name |
| `vendor_address` | String | Vendor's business address |
| `vendor_tax_id` | String | Vendor's tax identification number |
| `customer_name` | String | Customer/buyer company name |
| `customer_address` | String | Customer's billing address |
| `customer_tax_id` | String | Customer's tax identification number |
| `currency` | String | Invoice currency (USD, EUR, etc.) |
| `subtotal` | Number | Amount before taxes |
| `tax_amount` | Number | Total tax amount |
| `total_amount` | Number | Final invoice amount |
| `line_items` | String | Formatted string of all line items |

### Line Items Format
Line items are converted to a readable string format:
```
Product Name | qty: 2 | unit: 50.00 | total: 100.00; Next Item | qty: 1 | unit: 25.00 | total: 25.00
```

## Required Tools and Credentials

### APIs and Services
- **Google Gmail API**: For email monitoring and sending
- **Google Gemini API**: For AI-powered document analysis
- **Google Sheets API**: For database operations
- **Google PaLM API**: For language model functionality

### n8n Node Packages
- `n8n-nodes-base` (Gmail, Code, Merge nodes)
- `@n8n/n8n-nodes-langchain` (AI/ML nodes)

### Authentication Requirements
1. **Gmail OAuth2**: Configure Gmail account credentials
2. **Google PaLM API**: Set up Gemini API access
3. **Google Sheets OAuth2**: Configure Google Sheets access permissions

## Expected Google Sheets Structure

The target Google Sheet should have the following column headers in order:

```
invoice_number | invoice_date | due_date | vendor_name | vendor_address | vendor_tax_id | customer_name | customer_address | customer_tax_id | currency | subtotal | tax_amount | total_amount | line_items
```

## Setup Instructions

1. **Import Workflow**: Import the JSON configuration into your n8n instance
2. **Configure Credentials**:
   - Set up Gmail OAuth2 connection
   - Configure Google Gemini API credentials  
   - Set up Google Sheets OAuth2 access
3. **Update Configuration**:
   - Modify target email address in Gmail2 node
   - Update Google Sheets document ID if using different sheet
   - Adjust polling frequency if needed
4. **Test Workflow**: Send a test email with invoice attachment to verify functionality

## Usage

1. **Send Invoice**: Email an invoice (PDF/image) to the monitored Gmail account
2. **Automatic Processing**: Workflow triggers within 1 minute of email receipt
3. **Data Extraction**: AI analyzes document and extracts structured data
4. **Sheet Update**: New row added to Google Sheets with invoice data
5. **Notification**: Confirmation email sent with original attachment

## ⚠️ Important Disclaimers and Warnings

### Import Warning
**CRITICAL**: When importing this workflow into your n8n instance, some nodes may become undefined or lost due to:
- Missing node packages in your n8n installation
- Version compatibility issues between node packages
- Credential configuration differences
- Custom node dependencies not available in your environment

### Troubleshooting Missing Nodes
If nodes appear as "undefined" after import:
1. **Check Node Packages**: Ensure all required `@n8n/n8n-nodes-langchain` packages are installed
2. **Update n8n**: Make sure you're running a compatible n8n version
3. **Reinstall Packages**: Try reinstalling the langchain node package
4. **Manual Recreation**: You may need to manually recreate missing nodes using available alternatives

### Additional Considerations
- **API Quotas**: Monitor Google API usage limits for Gemini and Sheets
- **Email Volume**: High email volumes may require polling frequency adjustments
- **Document Types**: Workflow optimized for standard invoice formats (PDF, images)
- **Error Handling**: Implement additional error handling for production use
- **Data Privacy**: Ensure compliance with data protection regulations when processing invoices

### Performance Notes
- Processing time depends on document complexity and API response times
- Large attachments may require timeout adjustments
- Consider implementing retry logic for failed API calls

## Support

For issues related to:
- **Node Compatibility**: Check n8n community forums
- **API Limitations**: Refer to Google Cloud documentation
- **Workflow Logic**: Review node connections and data flow
