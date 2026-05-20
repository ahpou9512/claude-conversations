# Fix: Internal User Banner Not Displaying in Audience Wrapper

## The Issue — What Is Going Wrong

### How the audience wrapper works (on publish/live site)

The `dell` audiencewrapper component renders all three wrapper divs (Guest, Partner, Internal) in the HTML but hides them all with the CSS class `dds__d-none`. A JavaScript in `global.js` then reads the current user's audience type from the header element and reveals only the matching wrapper.

```
Page loads
  → globalheader.html renders <header class="dt-emc-header" data-audtype="...">
  → global.js reads data-audtype
  → finds wrapper with matching class (e.g. display-internal-content)
  → removes dds__d-none from that wrapper → banner shows
  → clears innerHTML of all other wrappers
```

In **edit/preview mode on Author**, the `jq-audiencewrapper` and `dds__d-none` classes are never added (by design in the HTL), so ALL wrappers are visible — that's why it looks correct in authoring.

---

### Why Internal users see empty columns on the live site

The `data-audtype` value on the header is set **server-side** by `EdSvcUserInfo.java` (used in `globalheader.html`).

**Bug in `EdSvcUserInfo.java` (line 224–226):**

```java
// Comment says "Not INTERNAL/GUEST" — but code only excludes GUEST
if (StringUtils.isNotBlank(sessionId)
        && StringUtils.isNotBlank(audType)
        && !isGuest) {          // ← isInternal is NOT excluded
    // Makes API call to external auth service...
    audienceType = userJsonObj.optString("AudienceTypes"); // returns e.g. "Customer, ExternalX, US"
}
```

This causes **two failure paths** for Internal users:

| Scenario | What happens | Result on page |
|---|---|---|
| Internal user **has** `userKey` cookie | API is called → returns `"Customer, ExternalX, US"` → `data-audtype` set to that | JS builds `displayClassName = "display-customer, externalx, us-content"` — no wrapper matches → **all 3 wrappers cleared** |
| Internal user **has no** `userKey` cookie | API skipped, falls to `else if (GUEST)` which also skips → `audienceType` stays `null` | `data-audtype` not rendered in HTML → JS reads `null` → `null.toLowerCase()` TypeError crash → **all 3 wrappers stay hidden** |

Guest works because line 287 explicitly sets `audienceType = audType` for Guest.  
Partner works because the API returns `"Partner"` correctly for them.

---

## The Fix — One File, Two Lines

**File:** `core/src/main/java/com/dell/core/use/EdSvcUserInfo.java`

### Change 1 — Exclude Internal users from the API call (line 226)

```java
// Before:
if (StringUtils.isNotBlank(sessionId)
        && StringUtils.isNotBlank(audType)
        && !isGuest) {

// After:
if (StringUtils.isNotBlank(sessionId)
        && StringUtils.isNotBlank(audType)
        && !isGuest
        && !isInternal) {
```

### Change 2 — Add Internal to the fallback (lines 287–289)

```java
// Before:
} else if(StringUtils.equalsIgnoreCase(audType, "GUEST")) {
    audienceType = audType;
}

// After:
} else if(StringUtils.equalsIgnoreCase(audType, "GUEST")
        || StringUtils.equalsIgnoreCase(audType, "INTERNAL")) {
    audienceType = audType;
}
```

With these two changes, Internal users will always get `audienceType = "Internal"` (from the cookie), so `data-audtype="Internal"` is set correctly on the header → `global.js` builds `displayClassName = "display-internal-content"` → the Internal audience wrapper is revealed → banner shows.

---

## Critical Files

| File | Role |
|---|---|
| `core/src/main/java/com/dell/core/use/EdSvcUserInfo.java` | Server-side use class — sets `data-audtype` on the header. **This is the file to fix.** |
| `ui.apps/.../template-components/globalheader/globalheader.html` | Renders the header with `data-audType="${edSvcUser.audienceType}"` |
| `ui.apps/.../clientlib-base/js/global.js` | Contains the audience wrapper JS (audience-wrapper.js module, ~line 21859) — reads `data-audtype` and shows/hides wrappers |
| `ui.apps/.../audiencewrapper/audiencewrapper.html` | Renders the wrapper divs with `display-internal-content` etc. classes |

---

## Verification

1. Build and deploy to DEV
2. Log in as an Internal user (Dell employee with `AudienceTypes=INTERNAL` cookie)
3. Visit the page — the Internal audience wrapper banner should now appear
4. Log in as Guest and Partner — confirm their banners still appear correctly
5. Check browser DevTools Console → no `TypeError` on `null.toLowerCase()`
6. Check `data-audtype` attribute on `.dt-emc-header` → should be `"Internal"` for Internal users
