# Undetectable Chrome Driver

GitHub Repository: [Link](https://github.com/UltrafunkAmsterdam/undetected-chromedriver)

## Introduction

The `undetected-chromedriver` repository is a custom implementation of Selenium's Chrome WebDriver designed to bypass detection mechanisms employed by various anti-bot services, such as Distil Networks, Imperva, DataDome, and CloudFlare's IUAM. citeturn0search0

**How It Works:**

`undetected-chromedriver` operates by modifying the standard ChromeDriver in ways that prevent websites from recognizing automation scripts. It automatically downloads and patches the ChromeDriver binary to avoid triggering anti-bot services. This allows developers to run Selenium-based automation tasks without being blocked or challenged by protective measures on websites. citeturn0search0

**Differences from Pure Selenium Framework in Python:**

1. **Detection Avoidance:** Standard Selenium WebDriver instances can be easily detected by sophisticated anti-bot systems, leading to blocks or CAPTCHAs. `undetected-chromedriver` specifically addresses these detection methods, enabling scripts to run more stealthily. citeturn0search0

2. **Automatic Patching:** While using Selenium typically requires manual management of the ChromeDriver binary, `undetected-chromedriver` automates the process by downloading and patching the driver to enhance its undetectability. citeturn0search0

3. **Additional Features:** `undetected-chromedriver` offers extra functionalities, such as support for Brave Browser and other Chromium-based browsers, provided some configurations are adjusted. It also includes methods like `click_safe()` and `children()` to improve interaction with web elements.

In summary, `undetected-chromedriver` extends the capabilities of the standard Selenium framework by focusing on evading detection from anti-bot services, automating driver management, and providing additional tools for web automation tasks. 

> [!IMPORTANT]
> This framework doesn't change your IP address, so, when you running it from your home or even ISP servers, changes to bypass bot detection systems are low.

### How it's actually working?

Repository structure:

```sh
.
├── cdp.py
├── devtool.py
├── dprocess.py
├── __init__.py
├── options.py
├── patcher.py
├── reactor.py
└── webelement.py
```

> [!NOTE]
> I have provided some descriptions about how different part of this module is working. For read more about them, see other markdown documents in this directory!
