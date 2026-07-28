---
title: "How to Fix “Sitemap Could Not Be Read” for Hashnode in Google Search Console"
datePublished: 2026-06-12T10:20:23.671Z
cuid: cmqarzlyl000104if0xfy924c
slug: fix-hashnode-sitemap-could-not-be-read-google-search-console
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/386b2625-e5e6-4af8-8f9f-f6a3facbca19.png
tags: blogging, seo, hashnode, sitemap, google-search-console

---

My Hashnode sitemap opened perfectly in the browser:

```text
https://blog.technopathy.club/sitemap.xml
```

The XML looked valid, all URLs were present, and there was no obvious problem.

Google Search Console still refused to process it.

The error was:

> **Sitemap could not be read**

In the submitted sitemap overview, Google showed:

> **Couldn't fetch**

The solution turned out to be a single query parameter:

```text
/sitemap.xml?sitemap=1
```

For my publication, the working URL is:

```text
https://blog.technopathy.club/sitemap.xml?sitemap=1
```

## The Google Search Console Error

I first submitted the standard Hashnode sitemap:

```text
https://blog.technopathy.club/sitemap.xml
```

Google Search Console detected no pages and displayed:

> **Sitemap could not be read**

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/6950b8a6-2913-461f-81c4-d21593adfe15.png align="center")

The sitemap overview showed:

*   Type: `Unknown`
    
*   Status: `Couldn't fetch`
    
*   Discovered pages: `0`
    
*   Discovered videos: `0`
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c6e15e96-8c96-4ff2-acea-aa322455cd8f.png align="center")

This was confusing because the same sitemap URL worked without any visible issue in a normal browser.

## The Sitemap Looked Fine in the Browser

Opening this URL directly displayed a normal XML sitemap:

```text
https://blog.technopathy.club/sitemap.xml
```

That made the failure look like a Google Search Console issue at first.

But a browser successfully displaying XML does not prove that the server is returning the correct HTTP headers.

Browsers are often tolerant and render the response based on its contents. Search engines and validators may be stricter.

## The Actual Problem: The Wrong Content-Type Header

My troubleshooting path eventually led to the sitemap validator at:

[XML Sitemap Validator](https://www.xml-sitemaps.com/validate-xml-sitemap.html)

The validator reported the exact error:

```text
Incorrect http header content-type: "text/html; charset=utf-8" (expected: "application/xml")
```

It also marked the sitemap as invalid:

```text
Sitemap is valid: No
```

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/a1b988d9-891c-46b7-bdad-a3bada786d5f.png align="center")

That explained the contradiction:

*   the sitemap content looked correct in the browser,
    
*   but the server returned it as `text/html`,
    
*   while the validator expected `application/xml`,
    
*   and Google Search Console therefore refused to process it.
    

## The One-Line Fix

Instead of submitting:

```text
/sitemap.xml
```

submit:

```text
/sitemap.xml?sitemap=1
```

For my Hashnode publication, that means:

```text
https://blog.technopathy.club/sitemap.xml?sitemap=1
```

For another Hashnode publication, use:

```text
https://your-domain.example/sitemap.xml?sitemap=1
```

That small query parameter makes Hashnode return the working sitemap response.

## The Successful Result

After submitting the parameterized URL, Google Search Console accepted it.

The overview showed:

*   Type: `Sitemap`
    
*   Status: `Success`
    
*   Discovered pages: `39`
    
*   Discovered videos: `0`
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/254aa226-f34b-4e1c-b03a-1b1e74d4019d.png align="center")

The detail page confirmed:

> **Sitemap processed successfully**

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/965e441b-fdba-43f0-8c07-67510de96eab.png align="center")

The difference was only this:

```text
/sitemap.xml
```

became:

```text
/sitemap.xml?sitemap=1
```

## How to Submit the Correct Hashnode Sitemap

Open Google Search Console and go to:

```text
Indexing → Sitemaps
```

Submit:

```text
sitemap.xml?sitemap=1
```

Or use the full URL:

```text
https://your-domain.example/sitemap.xml?sitemap=1
```

Google should then recognize the type as `Sitemap`, fetch the file, and discover the contained pages.

## Verify the Headers Yourself

You can inspect the response headers with `curl`.

Check the standard URL:

```bash
curl -I "https://your-domain.example/sitemap.xml"
```

Then compare it with:

```bash
curl -I "https://your-domain.example/sitemap.xml?sitemap=1"
```

The broken response may contain:

```http
Content-Type: text/html; charset=utf-8
```

For a proper XML sitemap response, you would expect an XML content type such as:

```http
Content-Type: application/xml
```

You can also inspect the response body:

```bash
curl -L "https://your-domain.example/sitemap.xml?sitemap=1"
```

The sitemap should contain valid XML and normally begin with something similar to:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
```

## No Proxy or Custom Wrapper Required

Before finding this solution, I considered creating a wrapper on a separate subdomain that would:

*   fetch the Hashnode sitemap,
    
*   replace the incorrect HTTP header,
    
*   return it as `application/xml`,
    
*   and expose the corrected response to Google.
    

That would have worked, but it would also have added unnecessary infrastructure and another possible point of failure.

The query parameter solves the problem without:

*   PHP,
    
*   a reverse proxy,
    
*   a redirect,
    
*   a custom sitemap generator,
    
*   or additional hosting.
    

## Final Fix

The standard Hashnode sitemap may look correct in a browser but still fail in Google Search Console because of the HTTP `Content-Type` header.

Do not submit:

```text
https://your-domain.example/sitemap.xml
```

Submit:

```text
https://your-domain.example/sitemap.xml?sitemap=1
```

This fixes the Google Search Console errors:

> **Sitemap could not be read**

and:

> **Couldn't fetch**

In my case, Google then processed the sitemap successfully and discovered all 39 pages.

* * *

I hope this quick fix saves you some time if Google Search Console refuses to process your Hashnode sitemap even though it looks perfectly valid in the browser.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy indexing! ¯\\\_(ツ)\_/¯