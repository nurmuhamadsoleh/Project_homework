# Project_homework
Link Akses Website Project HomeWork ---> https://nurmuhamadsoleh.github.io/Project_homework/ Atau https://project_homework.surge.sh
Bisa cek Link di bawah ini Database Contact Forms ---> https://docs.google.com/spreadsheets/d/1U-Vlc6eHEhYv955UemVXJOI33uvs0JnlArrABggc2RU/edit?usp=sharing

Penggunaan Contact Form Dengan Google Sheets

Submit a Form to Google Sheets
How to create an HTML form that stores the submitted form data in Google Sheets using plain 'ol JavaScript (ES6), Google Apps Script, Fetch and FormData.

1. Create a new Google Sheet
First, go to Google Sheets and Start a new spreadsheet with the Blank template.
Rename it Email Subscribers. Or whatever, it doesn't matter.
Put the following headers into the first row:
A	B	C	...
1	timestamp	email		

2. Create a Google Apps Script
Click on Tools > Script Editor… which should open a new tab.
Rename it Submit Form to Google Sheets. Make sure to wait for it to actually save and update the title before editing the script.
Now, delete the function myFunction() {} block within the Code.gs tab.
Paste the following script in it's place and File > Save:
var sheetName = 'Sheet1'
var scriptProp = PropertiesService.getScriptProperties()

function intialSetup () {
  var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet()
  scriptProp.setProperty('key', activeSpreadsheet.getId())
}

function doPost (e) {
  var lock = LockService.getScriptLock()
  lock.tryLock(10000)

  try {
    var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'))
    var sheet = doc.getSheetByName(sheetName)

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0]
    var nextRow = sheet.getLastRow() + 1

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header]
    })

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow])

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  catch (e) {
    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
      .setMimeType(ContentService.MimeType.JSON)
  }

  finally {
    lock.releaseLock()
  }
}

3. Run the setup function
Next, go to Run > Run Function > initialSetup to run this function.
In the Authorization Required dialog, click on Review Permissions.
Sign in or pick the Google account associated with this projects.
You should see a dialog that says Hi {Your Name}, Submit Form to Google Sheets wants to...
Click Allow
4. Add a new project trigger
Click on Edit > Current project’s triggers.
In the dialog click No triggers set up. Click here to add one now.
In the dropdowns select doPost
Set the events fields to From spreadsheet and On form submit
Then click Save
5. Publish the project as a web app
Click on Publish > Deploy as web app….
Set Project Version to New and put initial version in the input field below.
Leave Execute the app as: set to Me(your@address.com).
For Who has access to the app: select Anyone, even anonymous.
Click Deploy.
In the popup, copy the Current web app URL from the dialog.
And click OK.

6. Input your web app URL
Open the file named index.html. On line 12 replace <SCRIPT URL> with your script url:

<form name="submit-to-google-sheet">
  <input name="email" type="email" placeholder="Email" required>
  <button type="submit">Send</button>
</form>

<script>
  const scriptURL = '<SCRIPT URL>'
  const form = document.forms['submit-to-google-sheet']

  form.addEventListener('submit', e => {
    e.preventDefault()
    fetch(scriptURL, { method: 'POST', body: new FormData(form)})
      .then(response => console.log('Success!', response))
      .catch(error => console.error('Error!', error.message))
  })
</script>
As you can see, this script uses the the Fetch API, a fairly new promise-based mechanism for making web requests. It makes a "POST" request to your script URL and uses FormData to pass in our data as URL paramters.

Because Fetch and FormData aren't fully supported, you'll likely want to include their respective polyfills. See section #8.


7. Adding additional form data
To capture additional data, you'll just need to create new columns with titles matching exactly the name values from your form inputs. For example, if you want to add first and last name inputs, you'd give them name values like so:

<form name="submit-to-google-sheet">
  <input name="email" type="email" placeholder="Email" required>
  <input name="firstName" type="text" placeholder="First Name">
  <input name="lastName" type="text" placeholder="Last Name">
  <button type="submit">Send</button>
</form>
Then create new headers with the exact, case-sensitive name values:

A	B	C	D	...
1	timestamp	email	firstName	lastName	
8. Related Polyfills
Some of this stuff is not yet fully supported by browsers or doesn't work on older ones. Here are some polyfill options to use for better support.

Promise Polyfill
Fetch Polyfill
FormData Polyfill
Since the FormData polyfill is published as a Node package and needs to be compiled for browsers to work with, a good option for including these is using Browserify's CDN called wzrd.in. This service compiles, minifies and serves the latest version of these scripts for us.

You'll want to make sure these load before the main script handling the form submission. e.g.:

<script src="https://wzrd.in/standalone/formdata-polyfill"></script>
<script src="https://wzrd.in/standalone/promise-polyfill@latest"></script>
<script src="https://wzrd.in/standalone/whatwg-fetch@latest"></script>

<script>
  const scriptURL = '<SCRIPT URL>'
  const form = document.forms['submit-to-google-sheet']
  ...
</script>
