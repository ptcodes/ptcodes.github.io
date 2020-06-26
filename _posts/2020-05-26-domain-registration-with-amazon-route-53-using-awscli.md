---
layout: post
title: Domain Registration with Amazon Route 53 using AWS CLI
description: Register domains with Amazon Route 53 from the command line.
categories: [Amazon, AWS, Route 53, awscli]
image: /images/social/route53.png
---

[![Amazon Route53](/images/amazon-route53.png)](/domain-registration-with-amazon-route-53-using-awscli/)

[Amazon Route 53](https://aws.amazon.com/route53/) provides highly available and scalable Domain Name System (DNS), domain name registration, and health-checking web services.
You can register a domain name in different TLDs such as .com or .net. There are two ways of doing so: using the web interface or command line. 
I'm going to use the latter in this blog post.

To follow along you must have the AWS Command Line Interface (CLI) tool installed on your machine. If you don't, here is a step by step 
[guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) from Amazon.

AWS Route 53 uses the us-east-1 endpoint. If you have a different region specified with aws configure (see in ~./aws/config)
then it has to be updated to us-east-1 or you can specify the endpoint explicitly in every command.

<!-- more -->

## Check Domain Availability

Let's use a real example as I need to register a domain name for this website. First of all, we will find a domain name that isn't already in use by someone else with 
the following command: 
```bash
aws route53domains check-domain-availability \
    --region us-east-1 \
    --domain-name ptimofeev.com
```

The result will be:
```json
{
    "Availability": "AVAILABLE"
}
```

Sweet! Now, we can move on and register it.

## Domain Registration

When you register a domain name you are required to provide name contacts such owner, technical, admin and each has multiple details. You can specify everything in the command line but it's easy to make a mistake and a better way is to use a JSON file such as:

```json
{
    "DomainName": "ptimofeev.com",
    "DurationInYears": 1,
    "AutoRenew": true,
    "AdminContact": {
        "FirstName": "Martha",
        "LastName": "Rivera",
        "ContactType": "PERSON",
        "OrganizationName": "Example",
        "AddressLine1": "1 Main Street",
        "City": "Anytown",
        "State": "WA",
        "CountryCode": "US",
        "ZipCode": "98101",
        "PhoneNumber": "+1.8005551212",
        "Email": "mrivera@example.com"
    },
    "RegistrantContact": {
        "FirstName": "Li",
        "LastName": "Juan",
        "ContactType": "PERSON",
        "OrganizationName": "Example",
        "AddressLine1": "1 Main Street",
        "City": "Anytown",
        "State": "WA",
        "CountryCode": "US",
        "ZipCode": "98101",
        "PhoneNumber": "+1.8005551212",
        "Email": "ljuan@example.com"
    },
    "TechContact": {
        "FirstName": "Mateo",
        "LastName": "Jackson",
        "ContactType": "PERSON",
        "OrganizationName": "Example",
        "AddressLine1": "1 Main Street",
        "City": "Anytown",
        "State": "WA",
        "CountryCode": "US",
        "ZipCode": "98101",
        "PhoneNumber": "+1.8005551212",
        "Email": "mjackson@example.com"
    },
    "PrivacyProtectAdminContact": true,
    "PrivacyProtectRegistrantContact": true,
    "PrivacyProtectTechContact": true
}
```

When the file is updated with our details we can run (please pay attention that there will be an associated charge on your Amazon account):

```bash
aws route53domains register-domain \
    --cli-input-json file://register-domain.json \
    --region us-east-1
```

Output:
```json
{
    "OperationId": "af3486b1-8afc-42f2-8419-6ed40713483f"
}
```

It takes around an hour or two for Amazon to finalize domain registration. 
But when checking whether the domain is still available you will see that it is not:

```bash
aws route53domains check-domain-availability \
    --domain-name ptimofeev.com \
    --region us-east-1
```

Output:
```json
{
    "Availability": "UNAVAILABLE"
}
```

If you are registering a domain with Amazon Route 53 for the first time will get an email from Amazon Registrar to confirm your email address.

Once the registration is finalized list domains under your account:
```bash
aws route53domains list-domains \
    --region us-east-1
```

Output:
```json
{
    "Domains": [
        {
            "DomainName": "ptimofeev.com",
            "AutoRenew": true,
            "TransferLock": false,
            "Expiry": "2021-05-26T00:55:11-04:00"
        }
    ]
}
```

And you can see your domain there.

It turns out it's pretty convenient to register domain names from the command lines. It's easy to wrap commands in shell scripts to simplify further usage.
