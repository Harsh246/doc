# Integration Guide for Resolution Script with Decision Trees in Knowmax

This document provides instructions on how to integrate the resolution script with decision trees (DTs) in the Knowmax platform. The resolution script is responsible for creating tickets in the SuperOffice system based on the information gathered during the DT execution.

## Prerequisites

Before proceeding with the integration, ensure that the following prerequisites are met:

1. Access to the Knowmax platform and the ability to create or modify decision trees.
2. The necessary credentials for authentication with the SuperOffice API, which should be stored as an organization variable named `so_uat_auth` in Knowmax Org settings variables.

## Step 1: Create Required Variables

In each decision tree where you want to integrate the resolution script, you need to identify and create variables based on the ticket creation requirements:

- `soqueue`: This variable represents the queue in SuperOffice where the ticket should be created.
- `soticketstatus`: This variable represents the status of the ticket in SuperOffice.

You can define the variables in the variable section of your DT nodes.

### Example:

![Screenshot from 2024-05-23 15-32-54](https://github.com/Harsh246/doc/assets/68140807/7148cedc-8e9d-459e-97e8-0c5b27f0d676)


## Step 2: Update Variables (Optional)

If needed, you can update the values of `soqueue` and `soticketstatus` based on user input or other conditions within your DT. You can use the `km_set_var` function to update the variable values.

### Example:

![image](https://github.com/Harsh246/doc/assets/68140807/a7f453df-deb9-441c-a29e-79961fec6fc4)



```javascript
km_set_var("soqueue", "FIBER_TECH_SALTTV_SO_");
km_set_var("soticketstatus", 'Resolved');
```

## Step 3: Integrate the Resolution Script

Make sure to configure the "Mark ticket id as required" if needed to capture the MSISDN.

![Screenshot from 2024-05-23 16-07-31](https://github.com/Harsh246/doc/assets/68140807/c9a9128e-3c35-4a0a-89a3-3aae084d1623)


After setting the required variables on the desired final resolution node, you can integrate the resolution script into your DT, which will be the same for all the resolution nodes. The script should be added to the resolution section of your DT.

### Example Script:

```javascript
// Retrieve essential information from org and session variables
const basic_auth = km_get_var('so_uat_auth'); // Base64 encoded user:password from Org Settings
const msisdn = km_get_var('k_ticketId'); // Captured MSISDN
const username = km_get_var('k_username') || 'support'; // Username or some sample placeholder
const language = km_get_var('k_language') || 'FR'; // Language of Decision Tree in bcp47 code
const department = km_get_var('k_department') || 'test'; // Agent department in Knowmax
const resolution = km_get_resolution() || 'sample'; // Final resolution
const remarks = km_get_remarks() || 'sample'; // Final remarks
const history = km_get_history() || 'history'; // Session history
const so_queue = km_get_var('soqueue') || 'default'; // SO queue
const so_ticket_status = km_get_var('soticketstatus') || 'closed'; // SO ticket status

// Construct the request URL
const url = 'https://cs-sit.test.salt.ch/SuperOffice/api/v1/Agents/CRMScript/ExecuteScriptByIncludeId';

// Prepare the request headers
const headers = {
  'Content-Type': 'application/json',
  Authorization: `Basic ${basic_auth}`
};

// Request body
const body = {
  CRMScriptIncludeId: 'CREATE_TICKET_KNOWMAX',
  Parameters: {
    msisdn: msisdn,
    category: so_queue,
    ticketstatus: so_ticket_status,
    username: username,
    language: 'FR',  // figure out for different languages
    department: department,
    remarks: resolution || remarks, // Use either remarks or resolution
    history: history
  }
};

// POST request
try {
  const response = km_post_api(url, headers, JSON.stringify(body));

  // Check if the request was successful
  if (response?.ok) {
    const data = JSON.parse(response.response);
    const details = JSON.parse(data);

    km_append_notes(`Ticket Created Successfully: ${details?.ticketId}`);
  } else {
    // if the api request is not successful add some message in the remarks
    km_append_notes('Script Not Successful'); // unable to create SO ticket 
  }
} catch (err) {
  // if the script fails add some message in the remarks
  km_append_notes('Script Not Successful');
}
```

This script retrieves various variables from the DT session, constructs the request payload, and sends a POST request to the specified API endpoint. The response from the API is then processed, and the appropriate remarks are appended to the session.

## Conclusion

By following the steps outlined in this documentation, you can successfully integrate the resolution script with decision trees in the Knowmax platform. This integration will enable the creation of tickets in the SuperOffice system based on the information gathered during the DT execution.
