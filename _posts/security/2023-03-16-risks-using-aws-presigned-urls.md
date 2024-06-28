---
layout: post
title:  "Pre-signed at your service"
date:   2023-03-16 12:37:33 +0100
categories: security cloud
---

## Risks with pre-signed URLs in AWS S3
[In this article](https://labs.withsecure.com/publications/pre-signed-at-your-service) I cover some potential malicious use cases of AWS pre-signed URLs. I go on to cover some issues with default logging in S3 that could make it impossible for folks to narrow down what S3 object was accessed and when. I then go on to cover how a pre-signed URL’s active state sometimes relies on sessions, rather than the entity that created them. This fact could lead to misunderstanding the core component that needs to be secured in the event of a malicious action. I conclude by showing some log examples of what a malicious pre-signed access could look like when accessing objects in S3, with the required logging turned on. And what a revoke policy would looks like, when revoking the compromised role sessions.

Plain text link to original article: https://labs.withsecure.com/publications/pre-signed-at-your-service
