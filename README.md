# incident-reporting-automation-1w0so

## Overview

This n8n workflow monitors a Google Sheet for new incident reports. It processes each new entry, generating a unique incident ID and categorizing the incident. For phishing incidents, it downloads and analyzes an attached EML file for headers, URLs, and authentication details. Both phishing and other incident types involve scanning extracted URLs or domains with VirusTotal. Finally, it consolidates the analysis into summary fields, generates a dummy Jira ticket reference, and updates a separate Google Sheet with the processed incident details.

## Features

- Automated incident ID generation and date parsing.
- Categorization of incidents, specifically identifying phishing attempts.
- Downloading and analyzing EML files from Google Drive for phishing incidents.
- Extraction and decoding of EML headers (From, Subject, SPF, DKIM, DMARC).
- Authentication results scoring (SPF, DKIM, DMARC) and return-path mismatch detection.
- URL extraction from EML content and description fields, with whitelisting and cleaning.
- Integration with VirusTotal API for URL/domain threat intelligence.
- Consolidation of analysis results into concise summary fields.
- Generation of dummy Jira ticket references (can be extended to real Jira integration).
- Automatic update of a Google Sheet with processed incident data.

## Services Used

- Google Sheets
- Google Drive
- VirusTotal
- Jira (dummy integration)

## Trigger

New or updated row in a Google Sheet ('Sheet1') upon polling.

## Prerequisites

- A Google Sheet ('n8ntest', 'Sheet1') with incident reporting columns (e.g., 'Incident ID', 'Reporter Name', 'Email', 'Incident Type', 'Date of Incident', 'EML File (Drive URL)', 'Description').
- A separate Google Sheet ('n8ntest', 'Sheet2') structured to receive processed incident data (e.g., 'Incident ID', 'Date Reported', 'Reporter', 'EML Analysis', 'VT Result', 'Ticket ID').
- EML files linked in 'EML File (Drive URL)' column must be accessible by the Google Drive service account.
- A VirusTotal API key for URL/domain scanning.

## Credentials

- Google Sheets Trigger account (OAuth2Api) with read access to the input sheet.
- Google Drive account (OAuth2Api) with read access to EML files.
- Google Sheets account (OAuth2Api) with write access to the output sheet.

## Configuration

1. Configure the 'Google Sheets Trigger' node with the correct Google Sheet Document ID and Sheet Name for incident input.
2. Ensure the Google Sheets Trigger account has read permissions for the input sheet.
3. Configure the 'Download EML from Drive' node's Google Drive account with read permissions for the Google Drive folder containing EML files.
4. Provide your VirusTotal API key in an n8n environment variable named `VIRUSTOTAL_API_KEY` or replace the fallback value in the 'VirusTotal Lookup' nodes.
5. Configure the 'Update Sheet - Phishing' and 'Update Sheet - Other' nodes with the correct Google Sheet Document ID and Sheet Name for processed output.
6. Ensure the Google Sheets account for updating has write permissions for the output sheet.

## Usage

1. Activate the n8n workflow.
2. Add new rows to the configured Google Sheet (e.g., 'Sheet1') with incident details, including a Google Drive URL for EML files if applicable.
3. The workflow will automatically detect the new row, process the incident, perform analysis, and update the designated output Google Sheet (e.g., 'Sheet2') with the results.

## Troubleshooting

- If 'Download EML from Drive' fails: Check if the EML file Drive URL is correct and the Google Drive account has appropriate access permissions to the file.
- If 'Extract EML Headers & URLs' fails with 'Binary data 'data' nahi mila': Ensure the 'Download EML from Drive' node executed successfully and returned binary data.
- If VirusTotal lookup returns 'ERROR' or incomplete results: Verify the `VIRUSTOTAL_API_KEY` is correct and active, and check VirusTotal API rate limits.
- Incorrect date format in Google Sheet: The workflow attempts to parse Excel serial dates; ensure the input date column is consistently formatted.
- Incidents not appearing in the output sheet: Check trigger settings, workflow execution logs for errors, and ensure the input sheet is being monitored correctly.

## Security Notes

- The workflow uses a hardcoded fallback VirusTotal API key. It is highly recommended to store the `VIRUSTOTAL_API_KEY` securely as an n8n environment variable.
- EML content is parsed in a Code node, which could expose sensitive information if not handled carefully. Ensure data privacy policies are met.
- Access to Google Sheets and Google Drive is managed via OAuth2 credentials. Ensure these credentials have the principle of least privilege applied.
