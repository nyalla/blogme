---
template: blog-post
title: Outlook email crawling using Graph APIs
slug: Outlook-email-crawling-using-Graph-API
date: 2020-03-23 15:47
description: Outlook email crawling using Graph APIs
---
MS Graph APIs documentation is so well maintained and easily understandable in one read. But the actual implementation like when we are creating an app and giving permissions and testing the permissions are bit time-consuming process.

So our usecase is to get the emails from outlook for the given email id and need to post to the messaging queue. The overall goal looks pretty simple but it takes so much of time to actual working POC.

> **Note**: This is for Outlook Graph API where the mailbox is in **CLOUD**.

Reading user mailboxes has to be done in 2 ways.

1.  Case 1: OAuth with user behalf
2.  Case 2: OAuth without user (Admin consent)

OAuth with user behalf requires the user has to enter his sign-in details and had to generate a token and can communicate with Graph APIs. This is the external intervention of the user to enter sign in details.

This is of 3 steps

1.  creating code (requires user sign in details)
2.  creating token from the created code in step 1
3.  Accessing graph APIs with a token generated from step 2

But in the case of email scrapping, this method is not efficient as we have to generate tokens and give the details manually.

[https://docs.microsoft.com/en-us/outlook/rest/get-started](https://docs.microsoft.com/en-us/outlook/rest/get-started)

Oauth without user makes better options in email scrapping usecase as there is no intervention of the user to create any manual intervention. But it requires admin consent.

Allowing admin consent makes the app to read all mailboxes irrespective of any user in that tenant. It will be privacy issue with other mail boxes. We can restrict this with applying policies that give limited access to the mailboxes.

Just follow all steps in MS documentation.

https://docs.microsoft.com/en-us/graph/auth-v2-service

Just make sure you have given all the mailbox access permissions are provided for the created app.

[https://docs.microsoft.com/en-us/graph/auth-limit-mailbox-access](https://docs.microsoft.com/en-us/graph/auth-limit-mailbox-access)  -- For privacy restrictions

Once app creation and Admin consent permission for the app is done, the actual REST calls to get data has to be done.

So first we need to generate the token to get permissions for calling Graph APIs.

    POST:
        https://login.microsoftonline.com/{tenant id}/oauth2/v2.0/token
    Headers:
        grant_type : client_credentials
        client_id  : {client_id}
        client_secret : {client_secret}
        scope           : https://graph.microsoft.com/.default
Once you hit the above URL, you will get the token in response if the call is success. clientid/tenantid/clientsecret you will find in app details page which you created in earlier steps.

Then we have to hit the below API to get actual email content which we are looking for.

    GET:
        https://graph.microsoft.com/v1.0/users/{email id}/mailfolders/inbox/messages?$select=subject,from,receivedDateTime&$top=2&$orderby=receivedDateTime%20DESC
    Headers:
        Authorization   :       Bearer ey...(token id generated above)
Based on your favourite programming language you can call the above the APIs to get the emails.

But here we have to poll continuously to get the emails instantly. To overcome this frequent polling to email box, we can keep an event subscription to get notified whenever new emails arrive to the mailbox.

## Mailbox event subscription:

One prerequisite we have here is we have to provide an endpoint to the subscription where it has to be notified to. So first create an endpoint where that has to accesses without any issues from the internet.

Creation of subscription and more details you can find it here.

[https://docs.microsoft.com/en-us/graph/webhooks](https://docs.microsoft.com/en-us/graph/webhooks)

From the above link we can get a full outlook on subscription APIs and what we need from that documentation is one API where we tell the MS to send an event whenever new emails get into the given email box.

    POST https://graph.microsoft.com/v1.0/subscriptions
    Content-Type: application/json
    Header: Authorization Bearer .....  (Has to generate with Application consent app id)
    {
     "changeType": "created,updated",
      "notificationUrl": "web hook url", //end point we are developed as prerequisite
      "resource": "/users/{user email or guid}/mailfolders('inbox')/messages",
      "expirationDateTime": "2016-03-20T11:00:00.0000000Z",
      "clientState": "SecretClientState"
    }
You have to provide the expiration time for the event. Maximum expiration time is 3 days.

**Note**: Make sure about the response you are sending back to subscription API when above API is called.

After this we will get the data as body to our endpoint whenever any new email comes into the inbox.
Sample response to our endpoint will be something like below,

    {
      "value": [
        {
          "subscriptionId":"<subscription_guid>",
          "subscriptionExpirationDateTime":"2016-03-19T22:11:09.952Z",
          "clientState":"secretClientValue",
         "changeType":"created",
          "resource":"users/{user_guid}@<tenant_guid>/messages/{long_id_string}", //long_id_string is the message id
          "tenantId": "tenanet id",
          "resourceData":
          {
            "@odata.type":"#Microsoft.Graph.Message",
            "@odata.id":"Users/{user_guid}@<tenant_guid>/Messages/{long_id_string}",
            "@odata.etag":"W/\"CQAAABYAAADkrWGo7bouTKlsgTZMr9KwAAAUWRHf\"",
            "id":"<long_id_string>"
          }
        }
      ]
      }

> PS: Follow the MS documentation very clearly and step by step.
