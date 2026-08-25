Salesforce Account Creation & Experience Site Object Passing
Overview

This task implements an integration between an external HTML page, Salesforce Apex REST, and a Salesforce Experience Site.
The external HTML page allows the user to enter Account details, creates the Account in Salesforce through an Apex REST endpoint, opens the Experience Site, and then passes the created Account information to the Experience Site using the browser postMessage() API.

Architecture
External HTML Page
       │
       │ Account details
       ▼
Salesforce Apex REST API
       │
       │ Creates Account
       ▼
Salesforce Account
       │
       │ Returns Account Id / Object
       ▼
External HTML Page
       │
       │ window.postMessage()
       ▼
Salesforce Experience Site
       │
       │ LWC receives message
       ▼
Account information



What Was Implemented
1. Account Creation Form

Created an external HTML form to capture:
Account Name
Industry
Phone

The form validates the required fields before submitting the request.
The current form contains the Account creation UI and status/record display areas.

2. Salesforce OAuth Authentication
The HTML page connects to Salesforce using the OAuth Client Credentials flow.
The JavaScript requests an access token from Salesforce and stores the returned token for subsequent API requests.

3. Apex REST Account Creation
The Account information is sent to the Salesforce Apex REST endpoint using a POST request.
The request contains:
{
    accountName: "...",
    industry: "...",
    phone: "..."
}
The Salesforce response is then processed to obtain the newly created Account Id.

4. Experience Site Integration
After clicking Create Account, the Experience Site is opened in a new browser window:
experienceWindow = window.open(EXPERIENCE_SITE_URL, "_blank");
The external page also waits for the Experience Site to send an EXPERIENCE_SITE_READY message before sending the Account information.

5. Cross-Window Communication
Communication between the external HTML page and Experience Site is implemented using:
window.postMessage()
The target origin is explicitly specified:
experienceWindow.postMessage(
    { type: "PASS_ACCOUNT", value: accountObject },
    EXPERIENCE_SITE_ORIGIN
);
This allows the Account information to be transferred from the external page to the Experience Site.

6. Experience Site Ready Handshake
A handshake mechanism was implemented.
The Experience Site sends:
{
    type: "EXPERIENCE_SITE_READY"
}
The external page listens for this event and sends the pending Account object only after the Experience Site is ready.
This avoids sending the data before the Experience Site/LWC has initialized.

7. Moving From Account Id to Account Object
Initially, the implementation passed only the Salesforce Account Id:
{
    type: "PASS_VALUE",
    value: accountId
}
The implementation was then changed conceptually to pass the complete Account object:
{
    type: "PASS_ACCOUNT",
    value: accountObject
}
The object can contain:
{
    Id: result.id,
    Name: data.accountName,
    Industry: data.industry,
    Phone: data.phone
}
This allows the Experience Site to directly access multiple Account fields instead of receiving only the Id.


Data Flow
External HTML → Salesforce
const data = {
    accountName: document.getElementById("accountName").value.trim(),
    industry: document.getElementById("industry").value,
    phone: document.getElementById("phone").value.trim()

};

Salesforce → External HTML
After successful Account creation:
const accountObject = {
    Id: result.id,
    Name: data.accountName,
    Industry: data.industry,
    Phone: data.phone
};

External HTML → Experience Site
experienceWindow.postMessage(
    {
        type: "PASS_ACCOUNT",
        value: accountObject
    },
    EXPERIENCE_SITE_ORIGIN
);

Experience Site Receives Object
The receiver can process:
if (event.data?.type === "PASS_ACCOUNT") {
    const account = event.data.value;
    console.log("Account:", account);
    console.log("Id:", account.Id);
    console.log("Name:", account.Name);
    console.log("Industry:", account.Industry);
    console.log("Phone:", account.Phone);
}

Security Considerations
The postMessage() listener validates the message origin before processing it:
if (event.origin !== EXPERIENCE_SITE_ORIGIN) return;
This ensures that messages from unexpected origins are ignored.

Important: The current HTML contains Salesforce OAuth client credentials. These credentials should not be exposed in a publicly accessible HTML/JavaScript application. For production implementation, the OAuth credential exchange should be moved to a secure server-side layer.
----------

Current Result
---------------------
The completed flow is:
User enters Account details.
External HTML validates the form.
Experience Site is opened.
Salesforce OAuth access token is obtained.
Account is created through Apex REST.
Salesforce returns the created Account information.
Account object is stored as a pending object.
Experience Site sends a ready message.
External HTML sends the Account object using postMessage().
Experience Site receives and processes the Account object.

Technologies Used
-----------------------
HTML
CSS
JavaScript
Salesforce Apex REST
Salesforce OAuth 2.0
Salesforce Experience Cloud
Lightning Web Components
Browser window.postMessage()
Cross-window communication
