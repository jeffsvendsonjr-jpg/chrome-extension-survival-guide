# The Chrome Extension Developer's Field Guide

> The resource that should exist but doesn't.

A practical, honest guide to building, launching, and sustaining a Chrome extension—written by someone who learned the hard way.

---

## Why This Exists

Google's official documentation is incomplete. Stack Overflow answers are outdated. Most blog posts assume Manifest V2.

This guide covers what actually happens when you try to ship an extension in 2024-2025: the rejections, the cryptic violation notices, the permission justifications that take days to get right, the broken privacy policy link that costs you a week.

---

## Table of Contents

1. [Before You Build](#1-before-you-build)
2. [Building for Approval](#2-building-for-approval)
3. [The Submission Process](#3-the-submission-process)
4. [The Review Process](#4-the-review-process)
5. [Post-Acceptance Checklist](#5-post-acceptance-checklist)
6. [Common Rejections and Fixes](#6-common-rejections-and-fixes)
7. [Monetization](#7-monetization)
8. [Resources](#8-resources)

---

## 1. Before You Build

### Understand What You're Getting Into

The Chrome Web Store review process is rigorous by design. Google is protecting 3+ billion Chrome users from a high volume of low-quality or malicious submissions. Your job is to clearly demonstrate that your extension provides real value and respects user privacy.

**Timeline expectations:**
- First submission: Several business days for review, sometimes longer
- Resubmissions after rejection: Similar timeline
- Updates after approval: Usually faster, but not guaranteed

**Cost:**
- One-time $5 developer registration fee (verify current pricing at developer dashboard)
- No ongoing fees for free extensions

### Check If Your Idea Is Even Allowed

Before writing a single line of code, read the [Chrome Web Store Program Policies](https://developer.chrome.com/docs/webstore/program-policies/). 

**Instant rejection categories:**
- Extensions that exist solely to inject ads
- Extensions that collect user data without clear disclosure
- Extensions that duplicate browser functionality without adding value
- Extensions with misleading functionality or names
- Cryptocurrency miners
- Extensions that interfere with other extensions

### Manifest V3 Is Required for New Submissions

As of late 2024, new Manifest V2 extensions are no longer accepted for the public Chrome Web Store. If you're following an old tutorial, stop. New submissions must be MV3.

**Key MV3 differences:**
- Background pages → Service workers (no persistent background)
- `chrome.webRequest` blocking → `chrome.declarativeNetRequest`
- Remote code execution → Forbidden (no `eval()`, no remote scripts)
- Content Security Policy → Stricter defaults

---

## 2. Building for Approval

### Manifest.json: The Make-or-Break File

Your manifest is the first thing reviewers check. Every permission you request needs justification.

**Minimal manifest structure:**

```json
{
  "manifest_version": 3,
  "name": "Your Extension Name",
  "version": "1.0.0",
  "description": "Clear, accurate description under 132 characters.",
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },
  "permissions": [],
  "action": {
    "default_popup": "popup.html",
    "default_title": "Your Extension"
  }
}
```

### Permission Philosophy: Request Only What You Use

**Critical rule:** If you request a permission and don't use it, you will be rejected.

This isn't hypothetical. ShieldVault was rejected with this exact message:

> *Violation: Requesting but not using the following permission(s): scripting*

**Common permissions and when to use them:**

| Permission | Use Case | Scrutiny Level |
|------------|----------|----------------|
| `activeTab` | Accessing current tab on user action | Lower |
| `storage` | Saving user preferences | Lower |
| `scripting` | Programmatically injecting scripts | Medium |
| `tabs` | Accessing tab URLs/titles | Higher |
| `<all_urls>` | Running on all websites | High |
| `webRequest` | Intercepting network requests | High |

**Host permissions strategy:**

Instead of `<all_urls>`, list specific domains when possible:

```json
"host_permissions": [
  "https://chat.openai.com/*",
  "https://claude.ai/*",
  "https://gemini.google.com/*"
]
```

This shows reviewers you have a specific use case, not a data harvesting operation.

### Content Scripts: A Common Rejection Point

If your extension modifies web pages, you need content scripts. The manifest entry:

```json
"content_scripts": [
  {
    "matches": ["https://example.com/*"],
    "js": ["content-script.js"],
    "run_at": "document_idle"
  }
]
```

**Critical mistakes:**

1. **Using `<all_urls>` without justification** — Reviewers want to know why you need access to every website.

2. **Not using the permissions you request** — If you declare `"scripting"` in permissions but inject via content_scripts in manifest, you may not actually be using the scripting API. Remove unused permissions.

3. **Injecting into frames without reason** — Only add `"all_frames": true` if you actually need it.

### Icons: Don't Skip This

You need three icon sizes minimum:
- 16x16 — Browser toolbar
- 48x48 — Extensions management page
- 128x128 — Chrome Web Store listing

**Format:** PNG with transparency
**Style:** Clear, recognizable at small sizes

---

## 3. The Submission Process

### Pre-Submission Checklist

Before you hit submit:

- [ ] Manifest has only permissions you actually use
- [ ] Every permission has clear justification written
- [ ] Privacy policy is hosted at a live URL (not a PDF, not a Google Doc)
- [ ] Support page/email is real and accessible
- [ ] Store listing description matches what extension actually does
- [ ] All links work (test them right before submission)
- [ ] Screenshots show actual extension functionality
- [ ] Version number is correct

### Creating Your Developer Account

1. Go to [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole)
2. Pay the one-time registration fee
3. Verify your email
4. Enable 2-factor authentication (required for publishing)

### The Store Listing

**Name:** Must accurately describe what your extension does. Keyword stuffing = rejection.

**Summary (132 characters):** The elevator pitch. This appears in search results.

**Description:** Detailed explanation including:
- What the extension does
- How it works
- What permissions it needs and why
- Privacy practices

**Category:** Choose the most accurate one. Wrong category = rejection.

**Screenshots:** At least 1, up to 5. Must show actual functionality.

### Privacy Practices Declaration

This is where many developers encounter problems.

**The form asks:**
- Does your extension collect user data?
- What data do you collect?
- Do you transmit data externally?
- Do you sell user data?

**Answer honestly.** If you collect nothing, say so explicitly. If you store something locally, that might still count as "collection."

### Privacy Policy Requirements

You must host a privacy policy at a publicly accessible URL.

**Minimum requirements:**
- What data you collect (or explicit statement that you don't)
- How data is stored
- How data is used
- Whether data is shared with third parties
- How users can contact you

**Template for zero-collection extensions:**

```markdown
# Privacy Policy for [Extension Name]

Last updated: [Date]

## Data Collection
[Extension Name] does not collect, store, or transmit any user data.

## How It Works
All processing happens locally in your browser. No data leaves your device.

## Third-Party Services
[Extension Name] does not communicate with any external servers or third-party services.

## Contact
Questions? Contact [your email]
```

**Where to host:**
- GitHub Pages (free)
- Your own domain
- Any static hosting

**Do NOT use:**
- Google Docs links
- PDFs
- Notion pages
- Anything that requires login

### Permission Justifications

For each permission, you'll need to explain why. This is freeform text.

**Good justification example (for host permissions):**

> "ShieldVault requires access to AI chat platforms (ChatGPT, Claude, Gemini) to monitor text input fields for accidentally pasted API keys and credentials. The extension only activates on these specific domains and processes all data locally—nothing is transmitted externally."

**Bad justification example:**

> "Needed for the extension to work."

---

## 4. The Review Process

### What Actually Happens

1. **Automated scan** — Checks for known malware patterns, policy violations, manifest issues
2. **Human review** — A real person looks at your code and listing (especially for new developers or broad permissions)
3. **Decision** — Approved, rejected with reason, or request for more information

### Timeline Reality

Google doesn't publish guaranteed review timelines. Based on developer experience:

- **New developers:** Often takes longer
- **Broad permissions:** Can extend review time significantly
- **Resubmissions:** Sometimes faster, but not guaranteed
- **Holiday periods:** Expect delays

**There is no way to expedite review.** No support number to call, no button to push.

### If You Get Rejected

Don't panic. Read the rejection reason carefully.

**Common rejection format:**

```
Violation(s):

Use of Permissions:
Violation reference ID: [Color Word]
Violation: Requesting but not using the following permission(s): [permission]
How to rectify: [Instructions]
```

The "violation reference ID" (like "Purple Potassium") is internal Google tracking. You don't need to do anything with it.

---

## 5. Post-Acceptance Checklist

Congratulations, you're approved. But you're not done.

### Before You Announce Anywhere

**Verify all links are live:**
- [ ] Click your Chrome Web Store URL — does it load?
- [ ] Click your privacy policy URL — does it resolve?
- [ ] Click your support/homepage URL — does it work?
- [ ] Test the install button — does the extension actually install?

If any link is dead, broken, or redirects weird, fix it before you tell anyone. Nothing kills launch momentum like "link doesn't work" comments.

### Product Hunt Submission

**Before you post:**
- [ ] Profile photo uploaded (real or professional)
- [ ] Tagline written (under 60 characters)
- [ ] Description ready (problem → solution → why now)
- [ ] Gallery images/screenshots prepared
- [ ] Maker comment drafted (your story, why you built it)
- [ ] First link is the Chrome Web Store URL
- [ ] All links tested one more time

**After you post:**
- [ ] Respond to every comment
- [ ] Share to LinkedIn, Twitter/X, Reddit (with unique angles, not copy-paste)
- [ ] Coordinate with anyone who said they'd support you

### LinkedIn Announcement

- [ ] Profile is complete (photo, headline, about section)
- [ ] Chrome Web Store link in Featured section
- [ ] Post drafted with the story, not just the link

### Reddit Posts

Relevant subreddits for extension launches:
- r/webdev
- r/chrome_extensions
- r/SideProject
- r/InternetIsBeautiful (if applicable)
- Niche subreddits related to your extension's purpose

**Don't spam.** One thoughtful post per subreddit, tailored to that community.

---

## 6. Common Rejections and Fixes

### "Requesting but not using permission: [X]"

**Cause:** You declared a permission in manifest.json but your code never calls that API.

**Fix:** Remove the permission from your manifest, or add code that actually uses it.

**Example:** You have `"scripting"` in permissions but inject scripts via `content_scripts` in manifest instead of `chrome.scripting.executeScript()`. Remove `"scripting"` since you're not using that API.

### "Insufficient justification for permissions"

**Cause:** Your permission justification was too vague.

**Fix:** Rewrite with specifics:
- What feature requires this permission?
- Why can't you accomplish this with fewer permissions?
- What data do you access and why?

### "Privacy policy is missing or inaccessible"

**Cause:** Your privacy policy URL is broken, requires login, or doesn't exist.

**Fix:** Host a plain HTML or markdown file at a public URL. Test it in an incognito window.

### "Single purpose violation"

**Cause:** Your extension does too many unrelated things.

**Fix:** Either remove features to focus on one purpose, or split into multiple extensions.

### "Deceptive or misleading functionality"

**Cause:** Your store listing says one thing, your extension does another.

**Fix:** Align your description with actual functionality. Don't oversell.

---

## 7. Monetization

### Freemium Model

**Free tier:**
- Core functionality
- Basic patterns/features
- Zero data collection

**Paid tier:**
- Advanced features
- Custom configuration
- Priority updates

### Implementation Options

**Gumroad / LemonSqueezy:**
- Easiest to implement
- Handles payments, licensing, tax
- User buys license key, enters in extension

**Stripe direct:**
- More control
- More work (webhooks, customer portal)
- Better for subscriptions

### License Key Validation

Simple approach:
1. User purchases on Gumroad/LemonSqueezy
2. User receives license key
3. User enters key in extension options
4. Extension validates key against your API or validates locally

**Do NOT:**
- Ship your validation API key in the extension code
- Make paid features trivially bypassable
- Collect more data for paid users than you disclosed

---

## 8. Resources

### Official Documentation
- [Chrome Extensions Developer Guide](https://developer.chrome.com/docs/extensions/)
- [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole)
- [Program Policies](https://developer.chrome.com/docs/webstore/program-policies/)
- [One Stop Support](https://support.google.com/chrome_webstore/contact/one_stop_support)

### Community
- [Chromium Extensions Google Group](https://groups.google.com/a/chromium.org/g/chromium-extensions)
- [r/chrome_extensions](https://reddit.com/r/chrome_extensions)
- [Stack Overflow - google-chrome-extension tag](https://stackoverflow.com/questions/tagged/google-chrome-extension)

### Appeal Template

If you believe you were wrongly rejected:

```
Subject: Appeal for [Extension Name] - [Item ID]

Hello,

I'm writing to appeal the rejection of my extension.

Extension name: [Name]
Item ID: [ID]
Rejection reason: [Paste exact reason]

I believe this rejection may be in error because:
[Specific, factual explanation]

I have made the following changes:
[List changes if applicable]

I'm happy to provide additional documentation or make further modifications if needed.

Thank you for your time.

[Your name]
[Developer account email]
```

---

## About This Guide

This guide was created by Jeff, developer of [ShieldVault](https://chromewebstore.google.com/detail/shieldvault-ai-chat-secre/johfmefhjjmejjlopnndkbhmgdidkfao), after experiencing the full spectrum of Chrome Web Store joy and pain.

ShieldVault is a browser extension that detects and blocks API keys and secrets before they're submitted to AI chat interfaces. It exists because the modern developer workflow involves pasting code into AI tools—and that code sometimes contains secrets that shouldn't leave your machine.

---

## Contributing

Found something outdated? Have a war story to add?

- Open an issue
- Submit a PR
- Start a discussion

This guide gets better when developers share what they've learned.

---

## License

This guide is licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).

You're free to share and adapt it, even commercially, as long as you give credit and share derivatives under the same license.

---

*Last updated: December 2024*

*The most technologically advanced mega corporation on earth runs marginally faster than a 1992 DMV. This guide helps you survive the wait.*
