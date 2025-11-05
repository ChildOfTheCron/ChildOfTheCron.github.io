---
layout: post
title:  "Messing With Conditional Access Policy Templates In Entraid"
date:   2025-05-11 18:11:33 +0100
categories: security cloud
---

## Background

Conditional Access Policies are pretty complex when you have more than a couple. Especially when you need to analyze the overall effect they may have on your estate. Many important security features are provided by conditional access, including MFA, blocking legacy authentication or enforcing some type of authentication method based on device or location.

To me there doesn’t seem to be an easy way to analyze conditional access policies, without going through each one, then cross-referencing each policy with the overall collection of policies, to figure out if an Identity is or is not allowed to do something. 

This is a chore, but from my experimentation, there exists very few cheat codes that could quickly tell you what a conditional access policy was about. However while messing around I found that the "templateId" field may be somewhat helpful in this context. Using it you are able to tell which template the conditional access policy was created from. And this templateId remains the same across all Azure tenants.

## Conditional Access Policy Templates

A week ago I was experimenting with conditional access policies that are generated from templates. Microsoft tries to make it easy to roll out conditional access policies. These templates are one of the ways in which they do this. You are able to generate a conditional access policy quickly from the template, and once it is created, you can modify the template to fit your exact needs.

I was hoping that there was some way to tell if a conditional access policy was generated from a template. Better yet, I was hoping that there was a way to tell if a conditional access policy was generated from a template, and exactly which template the policy came from.

Even though conditional access policies (that are generated from a template) could be modified after creation, I was hopeful that at least knowing what template was used, would help in more quickly identifying what the conditional access policy was broadly being used for.

I pulled a non-template based conditional access policy's information using MS Graph API and noticed that one of the fields was called templateId, with a value of “null”.

    {
        "id": "65e06963-UUID-OF-CAP",
        "templateId": "null",
        "displayName": "Test-Cap-No-Tempalte",
        "createdDateTime": "2025-11-05TT21:48:26:913458Z",
        "modifiedDateTime": "null",
        [...]
    }
When generating a conditional access policy from a template, this value was (unsurprisingly) populated by a UUID.

    {
        "id": "6aef4e9f-UUID-OF-CAP"
        "templateId": "c7503427-338e-4c5e-902d-abe252abfb43"
        "displayName": "[Test] Require multifactor authentication for admins"
        [...]
    }
    
Next I deleted and re-created the policy from the same template to see if the UUID remained consistent between policy creations, and they did! Following up with [the documentation here](https://learn.microsoft.com/en-us/graph/api/resources/conditionalaccesspolicy?view=graph-rest-1.0) to confirm what I saw, we can see that the templateID field “[Specifies the unique identifier of a Conditional Access template. Inherited from entity.](https://learn.microsoft.com/en-us/graph/api/resources/conditionalaccesspolicy?view=graph-rest-1.0)” As a side note, I am pretty sure this used to also say “immutable” before. Or perhaps it said it elsewhere in the documentation but I can no longer find it. But since we ran our tests we can now see that unique means unique and consistent between creations.

One final check I wanted to do was see if this uniqueness held true across different Azure tenants. By that I mean, **does a templateid UUID from one Azure tenant match the same template UUID in another one**? So I created some test conditional access policies in another tenant and reran my tests. Sure enough, the templateIDs **remained the same across different Azure tenants**.

What this means is that regardless of Azure tenant, one can use these template UUIDs to determine from which template the conditional access policy was created. These UUIDs cannot be changed, unlike the conditional access policy name field, so they are reliable when analyzing policies.

For the sake of completeness, and because I couldn’t find this information documented anywhere on the internet, I am listing all the UUIDs for the conditional access templates below. Hopefully it’ll help others if they ever need to cross-reference which template a conditional access policy came from.  

## Appendix

Please note: You can’t see these UUIDs from the Azure Portal, either once the policy is created or during creation. You’ll need to use the API to grab the data. I have not looked into using Powershell for this, but maybe that’d also work.

| UUID | CAP Template Name |
|--|--|
| c7503427-338e-4c5e-902d-abe252abfb43 | Require multifactor authentication for admins |
| 0b2282f9-2862-4178-88b5-d79340b36cb8 | Block legacy authentication |
| b8bda7f8-6584-4446-bce9-d871480e53fa | Securing security info registration |
| a3d0a415-b068-4326-9251-f9cdf9feeb64 | Require multifactor authentication for all users |
| d8c51a9a-e6b1-454d-86af-554e7872e2c1 | Require multifactor authentication for Azure management |
| 927c884e-7888-4e81-abc4-bd56ded28985 | Require compliant or hybrid Azure AD joined device or multifactor authentication for all users |
| a297dd1a-21fe-4016-99a0-ba43ba64378c | Require MDM-enrolled and compliant device to access cloud apps for all users (Preview) |
| a4072ac0-722b-4991-981b-7f9755daef14 | Require multifactor authentication for guest access |
| 6b619f55-792e-45dc-9711-d83ec9d7ae90 | Require multifactor authentication for risky sign-ins |
| 634b6de7-c38d-4357-a2c7-3842706eedd7 | Require password change for high-risk users |
| 4e39a309-931e-4cb1-a371-e2beea168002 | Block access for unknown or unsupported device platform |
| 62e51ccc-c9c3-4554-ac70-066172c81007 | No persistent browser session |
| 81fd2072-4876-42b6-8157-c6000693046b | Use application enforced restrictions for O365 apps |
| 76c03f19-ea37-4656-a772-a183b4ddb81d | Require phishing-resistant multifactor authentication for admins |
| 6364131e-bc4a-47c4-a20b-33492d1fff6c | Require multifactor authentication for Microsoft admin portals |
| 16aaa400-bfdf-4756-a420-ad2245d4cde8 | Block access to Office365 apps for users with insider risk |
| a297dd1a-21fe-4016-99a0-ba43ba64378c | Require MDM-enrolled and compliant device to access cloud apps for all users (Preview) |





