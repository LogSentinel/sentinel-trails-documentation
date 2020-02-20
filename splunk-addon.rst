Splunk Add-on
=============
Install from Splunkbase
https://splunkbase.splunk.com/app/[LS_APP_ID]
***********************

1. Setup page: enter the LogSentinel Server URL

    ![Server URL](./splunk/1-setup.png)

2. Search your sources from the Search page

    ![Search sources](./splunk/2-search.png)

3. Use the Field Extractor to extract the following fields:
    * `actorId` (optional)
    * `action` (optional)
    * `actorDisplayName` (optional)
    * `entity` (optional)
    * `entityType` (optional)
    * `details` (AUTOMATICALLY EXTRACTED)
    * `logLevel` (optional)
    * `binaryContent` (optional)
    * `actorDepartment` (optional)
    * `entryType` (optional)

    ![Field Extractor](./splunk/3-extract-fields.png)

4. Make sure the `actorId` and `action` fields are non-empty

    ![Non-empty fields](./splunk/4-new-fields.png)

5. Create an alert for the search you just created

    ![Create alert](./splunk/5-create-alert.png)

6. Setup the alert as real-time type

    ![Alert settings](./splunk/6-alert-settings.png)

    The `Application ID`, `Secret` and `Organization ID` can be found at this LogSentinel page:
    https://app.logsentinel.com/api-credentials 
    ![Credentials](./splunk/7-credentials.png)