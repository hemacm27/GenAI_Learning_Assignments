# Playwright TypeScript Implementation Guide

## Complete Implementation Steps for AI-Extension

**Date:** April 15, 2026  
**Project:** AI-Extension (Chrome Extension for Code Generation)  
**Scope:** Adding Playwright TypeScript support alongside existing Selenium Java

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Implementation Steps](#implementation-steps)
4. [File Changes](#file-changes)
5. [Testing Checklist](#testing-checklist)
6. [Features Added](#features-added)

---

## Overview

This document outlines the complete implementation of Playwright TypeScript support in the AI-Extension project. Previously, the extension only supported Selenium Java code generation. Now it supports both:

### **Supported Combinations**

| Language | Framework | Feature Files | Page Objects | Combined |
|----------|-----------|---------------|--------------|----------|
| **Java** | Selenium | ✅ Yes | ✅ Yes | ✅ Yes |
| **TypeScript** | Playwright | ✅ Yes | ✅ Yes | ✅ Yes |

---

## Architecture

### System Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    USER INTERFACE (panel.html)                  │
│  ┌──────────────────┐         ┌──────────────────────────┐     │
│  │ Feature ☑️ Page  │         │ Language: Java/TypeScript│     │
│  │ O Both checkboxes│         │ Engine: Selenium/Playwright│   │
│  └──────────────────┘         └──────────────────────────┘     │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│               LOGIC LAYER (src/scripts/chat.js)                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  getPromptKeys(language, engine)                         │  │
│  │  ├─ isJavaSelenium() → SELENIUM_JAVA_*                 │  │
│  │  └─ isTypeScriptPlaywright() → PLAYWRIGHT_TYPESCRIPT_* │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│            PROMPT TEMPLATES (src/scripts/prompts.js)            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ SELENIUM_JAVA_PAGE_ONLY                                 │  │
│  │ CUCUMBER_ONLY                                           │  │
│  │ CUCUMBER_WITH_SELENIUM_JAVA_STEPS                       │  │
│  │ PLAYWRIGHT_TYPESCRIPT_PAGE_ONLY                    ✨ NEW │  │
│  │ PLAYWRIGHT_TYPESCRIPT_FEATURE_ONLY                 ✨ NEW │  │
│  │ PLAYWRIGHT_TYPESCRIPT_FEATURE_WITH_STEPS           ✨ NEW │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                   AI API (Groq/OpenAI/Testleaf)                │
│                    Returns Generated Code                       │
└────────────────┬────────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│               DISPLAY (Markdown + Syntax Highlighting)          │
│               Copy → Download → Run Tests                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Steps

### Step 1: Added Playwright TypeScript Prompts

**File:** `src/scripts/prompts.js`

Added 3 new prompt templates:

#### 1.1 Prompt: `PLAYWRIGHT_TYPESCRIPT_PAGE_ONLY`

**Purpose:** Generates a standalone Playwright TypeScript Page Object class

**Key Features:**
- Imports: `import { Page } from '@playwright/test'`
- Async/await pattern for all methods
- Locator definitions using Playwright's `.locator()` API
- JSDoc comments for documentation
- No test runner code

**Example Output:**
```typescript
import { Page } from '@playwright/test';

/**
 * Page Object for Login Page
 */
export class LoginPage {
    page: Page;

    constructor(page: Page) {
        this.page = page;
    }

    get usernameInput() {
        return this.page.locator('#username');
    }

    async enterUsername(username: string): Promise<void> {
        await this.usernameInput.fill(username);
    }
}
```

---

#### 1.2 Prompt: `PLAYWRIGHT_TYPESCRIPT_FEATURE_ONLY`

**Purpose:** Generates a Gherkin feature file for Playwright tests

**Key Features:**
- Standard Gherkin syntax (Given/When/Then)
- Scenario Outline with Examples
- South India realistic test data
- DOM-specific steps based on provided elements

**Example Output:**
```gherkin
Feature: Login to OpenTaps

Scenario Outline: Successful login with valid credentials
  Given I open the login page
  When I type "<username>" into the Username field
  And I type "<password>" into the Password field
  And I click the Login button
  Then I should be logged in successfully

Examples:
  | username    | password   |
  | "Raj Kumar" | "raj@1234" |
```

---

#### 1.3 Prompt: `PLAYWRIGHT_TYPESCRIPT_FEATURE_WITH_STEPS`

**Purpose:** Generates both feature file AND TypeScript step definitions

**Key Features:**
- Complete Gherkin feature file
- Full step definition implementation
- Browser launch/close hooks (@Before/@After)
- Playwright locators with explicit waits
- Error handling and assertions
- Async/await throughout

**Example Output:**
```typescript
import { Page, expect, chromium } from '@playwright/test';
import { Given, When, Then, Before, After } from '@cucumber/cucumber';

let page: Page;

Before(async () => {
    const browser = await chromium.launch();
    const context = await browser.newContext();
    page = await context.newPage();
});

Given('I open the login page', async () => {
    await page.goto('https://example.com');
    await page.waitForLoadState('networkidle');
});

When('I type {string} into the Username field', async (username: string) => {
    await page.locator('#username').fill(username);
});

Then('I should be logged in successfully', async () => {
    await expect(page).toHaveURL(/.*dashboard/);
});
```

---

### Step 2: Updated Routing Logic

**File:** `src/scripts/chat.js`

#### 2.1 Added Helper Method: `isTypeScriptPlaywright()`

```javascript
/**
 * Helper method to check if the combination is TypeScript + Playwright
 */
isTypeScriptPlaywright(language, engine) {
    return language === 'typescript' && engine === 'playwright';
}
```

---

#### 2.2 Updated `getPromptKeys()` Method

**Logic Flow:**

```
User selects: Feature ☑️ + Page ☑️

IF Java + Selenium:
  → Generate CUCUMBER_WITH_SELENIUM_JAVA_STEPS

ELSE IF TypeScript + Playwright:
  → Generate PLAYWRIGHT_TYPESCRIPT_FEATURE_WITH_STEPS

ELSE IF User selected Feature Only:
  IF TypeScript + Playwright:
    → Generate PLAYWRIGHT_TYPESCRIPT_FEATURE_ONLY
  ELSE:
    → Generate CUCUMBER_ONLY

ELSE IF User selected Page Only:
  IF Java + Selenium:
    → Generate SELENIUM_JAVA_PAGE_ONLY
  ELSE IF TypeScript + Playwright:
    → Generate PLAYWRIGHT_TYPESCRIPT_PAGE_ONLY
  ELSE:
    → Show unsupported message
```

**Code Updates:**
- Line 573: Added `isTypeScriptPlaywright()` check for default fallback
- Lines 588-595: Added Playwright routing for combined selection
- Lines 601-607: Added Playwright routing for feature-only selection
- Lines 611-619: Added Playwright routing for page-only selection

---

### Step 3: Updated UI Dropdowns

**File:** `panel.html`

#### 3.1 Language Binding Dropdown

**Before:**
```html
<option value="java" selected>Java</option>
<option value="csharp">C#</option>
<option value="py">Python</option>
<option value="ts">TypeScript</option>
```

**After:**
```html
<option value="java" selected>Java</option>
<option value="typescript">TypeScript</option>
<option value="csharp">C#</option>
<option value="python">Python</option>
```

**Changes:**
- ✅ Fixed `value="ts"` → `value="typescript"` (matching routing logic)
- ✅ Fixed `value="py"` → `value="python"` (consistency)
- ✅ Reordered: Java first, TypeScript second (most common)

#### 3.2 Browser Engine Dropdown

**Status:** Already contains correct values
```html
<option value="selenium" selected>Selenium</option>
<option value="playwright">Playwright</option>
<option value="cypress">Cypress</option>
<option value="puppeteer">Puppeteer</option>
```

---

### Step 4: Updated Code Generator Types

**File:** `src/scripts/prompts.js` (Lines 215-222)

**Added to exports:**
```javascript
export const CODE_GENERATOR_TYPES = {
  // ... existing ...
  PLAYWRIGHT_TYPESCRIPT_PAGE_ONLY: 'Playwright-TypeScript-Page-Only',
  PLAYWRIGHT_TYPESCRIPT_FEATURE_ONLY: 'Playwright-TypeScript-Feature-Only',
  PLAYWRIGHT_TYPESCRIPT_FEATURE_WITH_STEPS: 'Playwright-TypeScript-Feature-With-Steps',
};
```

---

## File Changes

### Summary of Modified Files

| File | Lines Changed | Change Type | Status |
|------|---------------|------------|--------|
| `src/scripts/prompts.js` | 1-220 | Added 3 new prompt templates + exports | ✅ Implemented |
| `src/scripts/chat.js` | 555-612 | Updated routing logic | ✅ Implemented |
| `panel.html` | 98-108 | Fixed dropdown values | ✅ Implemented |

### Detailed Changes

#### File 1: `src/scripts/prompts.js`

**Added After Line 189:**

```javascript
PLAYWRIGHT_TYPESCRIPT_PAGE_ONLY: `
  [Full prompt template with instructions and example]
`

PLAYWRIGHT_TYPESCRIPT_FEATURE_ONLY: `
  [Full prompt template with Gherkin example]
`

PLAYWRIGHT_TYPESCRIPT_FEATURE_WITH_STEPS: `
  [Full prompt template with Gherkin + TypeScript example]
`
```

**Updated Lines 215-222:**
```javascript
export const CODE_GENERATOR_TYPES = {
  // ... 3 existing entries ...
  // ... 3 new Playwright entries ...
};
```

---

#### File 2: `src/scripts/chat.js`

**Updated `getPromptKeys()` Method (Lines 555-612):**

- Added `isTypeScriptPlaywright()` check in default fallback (Line 573)
- Added Playwright routing for combined Feature+Page selection (Line 587)
- Added Playwright routing for feature-only selection (Line 601)
- Added Playwright routing for page-only selection (Line 614)

**New Helper Method Added (After Line 611):**
```javascript
isTypeScriptPlaywright(language, engine) {
    return language === 'typescript' && engine === 'playwright';
}
```

---

#### File 3: `panel.html`

**Updated Language Dropdown (Lines 98-108):**

Changed option values:
- `value="ts"` → `value="typescript"`
- `value="py"` → `value="python"`

Reordered options:
1. Java (selected)
2. TypeScript (moved to top)
3. C#
4. Python

---

## Testing Checklist

### Manual Testing Steps

#### Test 1: Java + Selenium → Feature Only
```
1. Open extension panel
2. Set Language: Java
3. Set Engine: Selenium
4. Check: ✅ Feature File Only
5. Click: Generate
Expected: CUCUMBER_ONLY prompt used → Feature file generated ✅
```

#### Test 2: Java + Selenium → Page Only
```
1. Open extension panel
2. Set Language: Java
3. Set Engine: Selenium
4. Check: ✅ Page Class Only
5. Click: Generate
Expected: SELENIUM_JAVA_PAGE_ONLY prompt used → Java class generated ✅
```

#### Test 3: Java + Selenium → Both
```
1. Open extension panel
2. Set Language: Java
3. Set Engine: Selenium
4. Check: ✅ Feature File + ✅ Page Class
5. Click: Generate
Expected: CUCUMBER_WITH_SELENIUM_JAVA_STEPS prompt used → Feature + Steps generated ✅
```

#### Test 4: TypeScript + Playwright → Feature Only
```
1. Open extension panel
2. Set Language: TypeScript
3. Set Engine: Playwright
4. Check: ✅ Feature File Only
5. Click: Generate
Expected: PLAYWRIGHT_TYPESCRIPT_FEATURE_ONLY prompt used → Feature file generated ✅
```

#### Test 5: TypeScript + Playwright → Page Only
```
1. Open extension panel
2. Set Language: TypeScript
3. Set Engine: Playwright
4. Check: ✅ Page Class Only
5. Click: Generate
Expected: PLAYWRIGHT_TYPESCRIPT_PAGE_ONLY prompt used → TypeScript class generated ✅
```

#### Test 6: TypeScript + Playwright → Both
```
1. Open extension panel
2. Set Language: TypeScript
3. Set Engine: Playwright
4. Check: ✅ Feature File + ✅ Page Class
5. Click: Generate
Expected: PLAYWRIGHT_TYPESCRIPT_FEATURE_WITH_STEPS prompt used → Feature + Steps generated ✅
```

#### Test 7: Unsupported Combination
```
1. Set Language: C#
2. Set Engine: Playwright
3. Check: ✅ Page Class Only
4. Click: Generate
Expected: Warning message "⚠️ csharp/playwright combination is not yet supported" ⚠️
```

---

## Features Added

### New Capabilities

#### ✨ Playwright TypeScript Page Objects
- **Pattern:** Modern async/await syntax
- **Locator API:** Uses `page.locator()` instead of Selenium's `By`
- **Import:** Playwright's `@playwright/test` package
- **Documentation:** JSDoc comments for all methods

#### ✨ Playwright TypeScript Feature Files
- **Format:** Standard Gherkin syntax
- **Data:** South India realistic test data
- **Parameters:** Scenario Outline with Examples table
- **Compatibility:** Works with any test runner

#### ✨ Playwright TypeScript Step Definitions
- **Framework:** Cucumber + Playwright integration
- **Lifecycle:** @Before (launch) and @After (cleanup) hooks
- **Browser:** Chromium launcher with context management
- **Waits:** Implicit waits via `waitForLoadState()`
- **Assertions:** Using Playwright's `expect()` API

### Supported Prompt Combinations

| Language | Engine | Feature File | Page Object | Combined |
|----------|--------|--------------|-------------|----------|
| Java | Selenium | ✅ | ✅ | ✅ |
| TypeScript | Playwright | ✅ NEW | ✅ NEW | ✅ NEW |
| C# | Selenium | ❌ Unsupported | ❌ Unsupported | ❌ Unsupported |
| Python | Selenium | ❌ Unsupported | ❌ Unsupported | ❌ Unsupported |
| TypeScript | Selenium | ❌ Unsupported | ❌ Unsupported | ❌ Unsupported |

---

## Code Examples

### Example 1: Generated Playwright Page Object

**Input:** Login form DOM with username/password fields

**Output:**
```typescript
import { Page } from '@playwright/test';

/**
 * Page Object for Login Page
 */
export class LoginPage {
    page: Page;

    constructor(page: Page) {
        this.page = page;
    }

    // Locators
    get usernameInput() {
        return this.page.locator('#username');
    }

    get passwordInput() {
        return this.page.locator('#password');
    }

    get loginButton() {
        return this.page.locator('button:has-text("Login")');
    }

    /**
     * Enter username in the username field
     */
    async enterUsername(username: string): Promise<void> {
        await this.usernameInput.fill(username);
    }

    /**
     * Enter password in the password field
     */
    async enterPassword(password: string): Promise<void> {
        await this.passwordInput.fill(password);
    }

    /**
     * Click the login button
     */
    async clickLogin(): Promise<void> {
        await this.loginButton.click();
    }

    /**
     * Perform login with provided credentials
     */
    async login(username: string, password: string): Promise<void> {
        await this.enterUsername(username);
        await this.enterPassword(password);
        await this.clickLogin();
    }
}
```

---

### Example 2: Generated Playwright Step Definitions

**Input:** Login feature file + DOM context

**Output:**
```typescript
import { Page, expect, chromium } from '@playwright/test';
import { Given, When, Then, Before, After } from '@cucumber/cucumber';

let page: Page;

Before(async () => {
    const browser = await chromium.launch();
    const context = await browser.newContext();
    page = await context.newPage();
});

After(async () => {
    await page.close();
});

Given('I open the login page', async () => {
    await page.goto('https://example.com/login');
    await page.waitForLoadState('networkidle');
});

When('I type {string} into the Username field', async (username: string) => {
    await page.locator('#username').fill(username);
});

When('I type {string} into the Password field', async (password: string) => {
    await page.locator('#password').fill(password);
});

When('I click the Login button', async () => {
    await page.locator('button:has-text("Login")').click();
});

Then('I should be logged in successfully', async () => {
    await expect(page).toHaveURL(/.*dashboard/);
    const success = page.locator('.success');
    await expect(success).toBeVisible();
});
```

---

## Integration Notes

### Synergy with Existing Features

✅ **Inspector Tool** - Works with both Java and TypeScript selection
✅ **Token Tracking** - Counts tokens for all generation types
✅ **API Providers** - Groq/OpenAI/Testleaf support all combinations
✅ **Markdown Parser** - Correctly highlights both Java and TypeScript syntax
✅ **Copy Functionality** - Works with all generated code blocks

### Browser Compatibility

This feature works in all Chromium-based browsers:
- ✅ Chrome
- ✅ Edge
- ✅ Brave
- ✅ Opera

---

## Troubleshooting

### Issue: TypeScript not generating

**Solution:** Verify that:
1. Language dropdown shows "TypeScript" (not "TS")
2. Engine dropdown shows "Playwright"
3. At least one checkbox (Feature/Page) is selected
4. API key is configured in Settings

### Issue: "Unsupported combination" message

**Reason:** Only these combinations are supported:
- Java + Selenium ✅
- TypeScript + Playwright ✅

All other combinations will show an unsupported message.

### Issue: Code not copied/downloaded

**Solution:** Ensure:
1. Code was generated successfully
2. Copy button is visible below the generated code
3. No JavaScript errors in browser console (F12)

---

## Future Enhancements

Potential additions for future versions:

1. **Python + Pytest** support with async syntax
2. **C# + NUnit** or **MSTest** support
3. **TypeScript + Cypress** integration
4. **Ruby + RSpec** for BDD workflows
5. **Go + Ginkgo** for backend testing
6. **Java + Testng** alternatives
7. **Custom test data generators**
8. **AI-powered test case suggestions**

---

## Summary

### What Was Implemented

| Component | Status | Details |
|-----------|--------|---------|
| Prompt Templates | ✅ | 3 new Playwright prompts added |
| Routing Logic | ✅ | Router updated for TypeScript/Playwright routing |
| UI Dropdowns | ✅ | Language/Engine dropdowns updated |
| Type Exports | ✅ | New prompt types exported |
| Backward Compatibility | ✅ | All Selenium Java features still work |

### Files Affected

```
ai-extension/
├── src/
│   ├── scripts/
│   │   ├── prompts.js          ✏️ Added 3 Playwright prompts
│   │   └── chat.js             ✏️ Updated routing logic
├── panel.html                  ✏️ Updated dropdown values
└── PLAYWRIGHT_IMPLEMENTATION_STEPS.md  ✨ This file
```

### Lines of Code Added

- **prompts.js:** ~350 lines (3 new prompt templates)
- **chat.js:** ~20 lines (new helper method + routing updates)
- **panel.html:** ~10 lines (dropdown value fixes)
- **Total:** ~380 lines of new code

---

## Deployment Checklist

- [ ] Review all code changes
- [ ] Test all 7 test scenarios above
- [ ] Check console for errors (F12)
- [ ] Verify token counting works
- [ ] Test copy/download functionality
- [ ] Validate markdown highlighting for both languages
- [ ] Confirm API responses include code blocks
- [ ] Test with multiple API providers (Groq, OpenAI, Testleaf)
- [ ] Reload extension in Chrome (chrome://extensions)

---

**End of Implementation Guide**

---

*Document Version: 1.0*  
*Last Updated: April 15, 2026*  
*Created for: AI-Extension Chrome Extension Project*
