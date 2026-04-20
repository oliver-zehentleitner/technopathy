---
title: "How to create a Binance API Key and API Secret?"
datePublished: 2026-04-20T14:53:42.536Z
cuid: cmo7bey2u00e62eh5hrn4bg3p
slug: how-to-create-a-binance-api-key-and-api-secret
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c77746b5-193c-4a14-a575-75dc4fba8c76.webp
tags: api, binance

---

**You want to connect a script or trading bot to the Binance API?**

**In order to proceed, you'll need an API key/secret pair. I'll explain how to create this quickly and concisely in the following steps!**

* * *

### Create API key/secret pair for Binance.com

First, please log in to [binance.com](https://www.binance.com). If you don't have a Binance user account yet, you can sign up via my [referral link](https://www.binance.com/en/activity/referral-entry/CPA?fromActivityPage=true&ref=CPA_008DXU7CWB). With this we both get 100 USDT cashback vouchers for a 50 USD deposit.

Okay, let's get started!

#### Step 1

Click on the account button:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/b4a96a50-aec9-4903-8ef8-e6fe157a9c3a.png align="center")

#### Step 2

And then on "API Management":

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/393608b0-b391-4c11-b99e-4db328dfef3f.png align="center")

Now you are on the page where you can create, edit and delete your API key/secret pairs and define their respective permissions and IP whitelisting.

#### Step 3

To create an API access, next click on the yellow "create API" button in the upper right corner:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/039b6652-8f9e-42f6-aeb7-2a1e21f1318c.png align="center")

#### Step 4

Now a modal window pops up and you can decide which API key type you want to create. The definitely easier and faster process is the "System generated" HMAC-SHA256 type — the advantage of the "Self-generated" RSA type (private/public key pair) is that technically not even Binance knows your key for signing and RSA key pairs can also be stored with higher security, as they can be stored in a LUKS container or HSM and the mandatory entry of a passphrase can be activated to make the keys usable.

At the moment RSA signatures are not yet supported by the [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite), as soon as this is the case we will adapt the instructions here.

So we continue with "System generated" and click "Next":

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/62238773-31c7-427b-a8ce-0ea03ac84f98.png align="center")

#### Step 5

Since you can create multiple API Keys with different permissions and IP whitelists, you need to enter a label in this step. This will help you to identify your API key/secret pairs. To continue click on "Next":

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/21a49245-a8d0-454d-88ea-19766df6208b.png align="center")

#### Step 6

For security purposes, Binance wants you to play the Binance Sliding game:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/fedbe39f-2152-4faf-a8e9-fc4b0063eddc.png align="center")

#### Step 7

First you need to request an email verification code:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/6fcec1a5-089d-43e5-94da-ddabb76b29df.png align="center")

#### Step 8

Check your email inbox (and spam folder if necessary), Binance will now immediately send you an email with a verification code:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/507deb47-6a30-45ea-9b99-8b78080c0fda.png align="center")

#### Step 9

Copy the verification code from the email and paste it here:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/4e4e59ec-f353-4589-b5cb-6f736479fa1b.png align="center")

#### Step 10

Open Google's Authenticator app on your smartphone and enter the code for your user account. To continue, click on "Submit":

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f88cb4c6-0caf-47e4-8fb8-cf2e70cc202b.png align="center")

#### Step 11

Immediately copy the API key/secret pair to a safe place, it will be shown to you only once! If you do not want to have read-only access to the API, you have not yet finished configuring the API key/secret pair.

All further permissions you have to enable explicitly and as soon as you grant further permissions, an IP whitelist is mandatory!

#### Step 12

Click on "Edit restrictions":

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/79cbdc2a-46b4-4cca-846f-2544f00746cb.png align="center")

#### Step 13

Before you can enable the API Restrictions check boxes, you must select "Restrict access to trusted IPs only (Recommended)" under "IP access restrictions" and whitelist one or more IPs:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/50dfaf00-fcae-4b72-a230-2aec0a66fe73.png align="center")

#### Step 14

Now you can grant the desired permissions:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/81cd235f-a77d-4391-8647-32d76b439ad0.png align="center")

#### Step 15

When you have set everything, click on "Save":

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/73926bb0-bde8-4c40-8e4d-d5b74d6f658a.png align="center")

To save the new permissions and the IP whitelist you have to run the "Security Verification" from step 7 to step 10 again.

Now you can use your API key/secret pair!

* * *

I hope you found this tutorial informative and enjoyable!

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!

* * *

*Image source:* [*pixabay.com*](https://pixabay.com)