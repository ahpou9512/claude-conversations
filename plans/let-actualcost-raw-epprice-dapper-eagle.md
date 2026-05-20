# Fix actualCost currency for zero-price courses

## Context
Zero-price courses always display "0 USD" regardless of the selected country/currency, because the zero-branch had "0 USD" hardcoded. Non-zero price courses correctly reflect the country currency using `${raw.epcurrency}`. The fix is to use `raw.epcurrency` in the zero-branch as well, so changing the country also updates the currency for free courses (e.g. "0 MYR" for Malaysia).

## Root Cause
**File:** `ui.apps/src/main/content/jcr_root/apps/dell/components/custom-components/coveosearch/clientlib-coveosearch/js/coveosearch.js`  
**Line 778 (comment showing old hardcoded version):**
```javascript
//let actualCost = raw.epprice == 0 ? "0 USD" : `${raw.epprice} ${raw.epcurrency}`;
```
The previously deployed code hardcoded `"0 USD"` in the zero-price branch instead of using `${raw.epcurrency}`.

## Changes Required

### 1. Remove debug console.log (line 779)
```javascript
// DELETE this line:
if (raw.epprice == 0) console.log('[DEBUG] epprice=0, epcurrency=', raw.epcurrency, 'raw=', raw);
```

### 2. Keep the fixed actualCost line (already done at line 780)
```javascript
let actualCost = raw.epprice == 0 ? `0 ${raw.epcurrency}` : `${raw.epprice} ${raw.epcurrency}`;
```
This ensures zero-price courses use `raw.epcurrency` dynamically, matching the behaviour of non-zero price courses.

## Verification
- Select **Malaysia** as country → zero-price course should show `0 MYR / X Training Credits`
- Select **USA** as country → zero-price course should show `0 USD / X Training Credits`
- Non-zero price course in any country should remain unaffected
