# Chrome-Extension-with-Google-Sheets-Integration
 integrate a Chrome extension (we developed it) with Google Sheets using Apps Script. The ideal candidate will have a solid understanding of Google Apps Script and how to access all information in a google sheet (what data is in the sheet, how to write to the sheet, how to detect the user clicked on a cell etc). We'll handle the business logic on our side
 ==============
To integrate a Chrome extension with Google Sheets using Google Apps Script, we will create an Apps Script Web App that interacts with Google Sheets via the Google Sheets API. The Chrome extension will communicate with this web app to retrieve data from the sheet, write data to the sheet, and detect interactions like which cell was clicked.
Steps to integrate:

    Create a Google Apps Script Web App: This web app will handle the interaction between your Chrome extension and Google Sheets. The web app will expose functions for reading from and writing to Google Sheets.

    Modify the Chrome extension: You'll need to modify your extension to send HTTP requests (via fetch or XMLHttpRequest) to the Apps Script web app, which will handle the interaction with Google Sheets.

    Handle Cell Clicks: This part will allow the Chrome extension to send data to the web app when a user clicks on a specific cell in the Google Sheet.

1. Google Apps Script Web App:

First, create a Google Apps Script project.
Apps Script Project:

    Go to Google Apps Script and create a new project.
    Under File, create a new project.
    Replace the content in the Code.gs file with the following code:

const SPREADSHEET_ID = 'your_google_sheet_id'; // Replace with your sheet ID

/**
 * Get the content of the Google Sheet.
 * @param {string} range The range of cells to read.
 * @returns {Array} The content of the sheet.
 */
function getSheetData(range) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getSheets()[0];
  const data = sheet.getRange(range).getValues();
  return data;
}

/**
 * Write data to the Google Sheet.
 * @param {string} range The range of cells to write to.
 * @param {Array} values The data to write to the cells.
 */
function writeSheetData(range, values) {
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getSheets()[0];
  sheet.getRange(range).setValues(values);
}

/**
 * Respond to a cell click event from the Chrome Extension.
 * This function receives the clicked cell's data.
 * @param {string} cellValue The value of the clicked cell.
 */
function handleCellClick(cellValue) {
  // You can handle business logic here
  Logger.log('Cell clicked: ' + cellValue);
  // For example, updating the clicked cell with a message
  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID).getSheets()[0];
  sheet.getRange('A1').setValue('Cell clicked: ' + cellValue); // Example update
}

/**
 * Web App doGet function to serve the requests from the Chrome extension
 */
function doGet(e) {
  return HtmlService.createHtmlOutput("Welcome to Google Sheets Web App");
}

/**
 * Web App doPost function to handle requests from the Chrome extension
 */
function doPost(e) {
  const params = JSON.parse(e.postData.contents);
  
  if (params.type === 'read') {
    const data = getSheetData(params.range);
    return ContentService.createTextOutput(JSON.stringify(data)).setMimeType(ContentService.MimeType.JSON);
  }
  
  if (params.type === 'write') {
    writeSheetData(params.range, params.values);
    return ContentService.createTextOutput('Data written successfully').setMimeType(ContentService.MimeType.TEXT);
  }
  
  if (params.type === 'click') {
    handleCellClick(params.cellValue);
    return ContentService.createTextOutput('Cell clicked: ' + params.cellValue).setMimeType(ContentService.MimeType.TEXT);
  }

  return ContentService.createTextOutput('Invalid request').setMimeType(ContentService.MimeType.TEXT);
}

    In this script, getSheetData reads data from the sheet, writeSheetData writes to the sheet, and handleCellClick simulates handling cell click events. The doGet and doPost methods handle HTTP GET and POST requests respectively.

    Publish the script as a Web App:
        Go to Publish > Deploy as web app.
        Choose Anyone for the access settings (you can also restrict access if you want).
        Deploy the script.

After deploying, you will get a Web App URL. You will use this URL to interact with your Google Sheets from your Chrome extension.
2. Chrome Extension:

In your Chrome extension, you'll send HTTP requests to the Apps Script Web App to interact with Google Sheets.
Manifest (manifest.json):

Ensure you have the appropriate permissions and settings in your extension's manifest:

{
  "manifest_version": 3,
  "name": "Google Sheets Integration",
  "description": "Integrates Google Sheets with a Chrome extension",
  "version": "1.0",
  "permissions": [
    "identity",
    "https://www.googleapis.com/*",
    "https://script.google.com/macros/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "images/icon16.png",
      "48": "images/icon48.png",
      "128": "images/icon128.png"
    }
  },
  "oauth2": {
    "client_id": "YOUR_GOOGLE_OAUTH_CLIENT_ID",
    "scopes": ["https://www.googleapis.com/auth/spreadsheets"]
  }
}

Background Script (background.js):

This script will handle communication between your Chrome extension and Google Apps Script.

// background.js

const webAppUrl = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec'; // Replace with your Web App URL

// Function to fetch data from Google Sheets via Apps Script Web App
async function fetchSheetData(range) {
  const response = await fetch(webAppUrl, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ type: 'read', range: range })
  });

  const data = await response.json();
  console.log('Data from Google Sheets:', data);
  return data;
}

// Function to write data to Google Sheets via Apps Script Web App
async function writeToSheet(range, values) {
  const response = await fetch(webAppUrl, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ type: 'write', range: range, values: values })
  });

  const result = await response.text();
  console.log('Write result:', result);
}

// Function to handle cell clicks
async function handleCellClick(cellValue) {
  const response = await fetch(webAppUrl, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ type: 'click', cellValue: cellValue })
  });

  const result = await response.text();
  console.log('Cell click handled:', result);
}

// Example usage
chrome.action.onClicked.addListener(() => {
  // Read from a range (e.g., A1:C10)
  fetchSheetData('A1:C10').then(data => console.log(data));
});

Popup HTML (popup.html):

This file will provide the user interface for interacting with Google Sheets.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Google Sheets Integration</title>
  <script src="popup.js"></script>
</head>
<body>
  <h1>Google Sheets Integration</h1>
  <button id="fetchData">Fetch Data</button>
  <button id="writeData">Write Data</button>
  <button id="handleClick">Handle Cell Click</button>
</body>
</html>

Popup Script (popup.js):

Handle the UI interactions and send the appropriate messages to the background script.

// popup.js

document.getElementById('fetchData').addEventListener('click', () => {
  chrome.runtime.sendMessage({ action: 'fetchData', range: 'A1:B2' });
});

document.getElementById('writeData').addEventListener('click', () => {
  chrome.runtime.sendMessage({
    action: 'writeData',
    range: 'A1:B2',
    values: [['New Data 1', 'New Data 2'], ['Another Data 1', 'Another Data 2']]
  });
});

document.getElementById('handleClick').addEventListener('click', () => {
  chrome.runtime.sendMessage({ action: 'handleClick', cellValue: 'A1' });
});

Conclusion:

    Google Apps Script: Handles reading, writing, and interaction with Google Sheets.
    Chrome Extension: Uses fetch to send requests to the Apps Script Web App.
    This structure ensures your extension and Google Sheets are seamlessly integrated.

Make sure to test the extension and Apps Script Web App thoroughly to ensure everything works as expected.
