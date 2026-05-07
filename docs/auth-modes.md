# Auth Modes

MockHunter supports four auth modes in v0.1.0. Pick the one that matches your target page.

## `public`

Use for pages that don't require login (marketing pages, public dashboards, demos).

**Setup:** Nothing. Just provide the URL.

```
URL: https://example.com/pricing
Auth: public
```

## `localhost`

Use when auditing a local dev server. Same as `public` but documents intent and (in v0.2+) may add localhost-specific heuristics.

**Setup:**

```
URL: http://localhost:5173
Auth: localhost
```

**Tip:** Make sure your dev server is running before invoking the skill.

## `form`

Use for any web app with email/password login.

**Setup — provide:**
- Login URL (where the form lives)
- Email field selector (e.g., `input[name="email"]` or `input[type="email"]`)
- Password field selector (e.g., `input[name="password"]` or `input[type="password"]`)
- Submit button selector (e.g., `button[type="submit"]` or `button:has-text("Log in")`)
- Email value (use a test account, not your real one)
- Password value

**Example:**

```
URL: https://staging.myapp.com/dashboard
Auth: form
Login URL: https://staging.myapp.com/login
Email field: input[name="email"]
Password field: input[name="password"]
Submit: button[type="submit"]
Email: tester@example.com
Password: TestOnly123!
```

**Tips for form login:**
- Use a dedicated test account, not your personal account
- Make sure the test account has data so the audit isn't trivially a cold-start scenario
- If your login form has CAPTCHAs, see [edge cases](#edge-cases) below

### Finding selectors

Open your login page in a browser, right-click the email field → Inspect. The simplest selector is usually one of:

- `input[name="email"]`
- `input[type="email"]`
- `input#email`
- `input[placeholder*="email" i]`

If multiple match, pick the most specific. Test in the browser console:

```javascript
document.querySelector('input[name="email"]')
```

If it returns the right element, that's your selector.

## `skip`

Use when you want to audit only what's reachable without logging in. Useful for:
- Confirming a page is gated behind auth (the audit will hit the login wall and stop)
- Auditing the public face of an app (landing page, signup page)
- Smoke-testing the auth gate itself

**Setup:**

```
URL: https://app.example.com/dashboard
Auth: skip
```

The audit will report whatever it sees — likely a login redirect or empty state.

## Edge cases

### CAPTCHA / reCAPTCHA on login

v0.1.0 does not solve CAPTCHAs. **Workaround:** log in manually first in the same browser session that Playwright uses (this requires a persistent Playwright context). Or, if your app has a CAPTCHA-free login route for testing, use that.

### Magic-link login

v0.1.0 does not support magic-link flows. **Workaround:** log in manually first via your email client; Playwright will inherit the session if you're using a persistent context. Or, for testing, switch your app to password login temporarily.

### OAuth (Google, GitHub, etc.)

v0.1.0 does not support OAuth flows. **Workaround:** create a test account that uses email/password instead. Most apps that offer OAuth also offer email/password — use that path for the audit.

### 2FA / TOTP

v0.1.0 does not handle 2FA. **Workaround:** disable 2FA on your test account, or log in manually and let Playwright inherit the session.

### Session cookies expire mid-audit

If the audit takes long enough that your session cookie expires, MockHunter will start seeing 401s and report them as BROKEN. Use a test account with a longer session lifetime, or split the audit into smaller chunks.

### Multi-step login (email → next page → password)

Some apps split email and password across two pages. v0.1.0 supports this if you provide both selectors and tell MockHunter to expect a multi-step flow. If your login is multi-step, mention it in the smart-questions phase and MockHunter will adapt.

### Login page has anti-automation (Cloudflare Turnstile, etc.)

Same workaround as CAPTCHA — log in manually first if you can, or use a test environment without anti-automation.

## Choosing the right mode

| Situation | Mode |
|---|---|
| Public marketing page | `public` |
| Local dev server | `localhost` |
| App with email/password login | `form` |
| Want to audit just the auth gate | `skip` |
| App with OAuth/magic-link/2FA | `form` with workaround (manual pre-login) |
| Don't care about authenticated pages | `skip` |

## Coming in v0.2+

- OAuth flow support (with caveats)
- Magic-link automation via inbox API
- Multi-step login first-class support
- Session re-use across runs
- Cookie-based auth (paste a cookie string, skip the login dance)
