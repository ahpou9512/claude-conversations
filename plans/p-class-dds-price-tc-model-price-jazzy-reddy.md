# Plan: Clean Price Display — Strip Floating Point from price

## Context

`model.price` is a Java `double`. When the value is `0.0`, HTL renders it literally as `0.0`.
The requirement is: always display the exact numeric amount, but strip unnecessary decimal points —
`0.0` → `0`, `100.0` → `100`, `99.99` → `99.99`. Currency and training credits are always shown.

The cleanest fix adds one formatting method in Java (testable, readable) and keeps HTL a simple one-liner.

---

## Files to Modify

| File | Change |
|------|--------|
| `core/src/main/java/com/dell/core/models/CourseDetailsModel.java` | Add `getPriceLabel()` method |
| `ui.apps/src/main/content/jcr_root/apps/dell/components/custom-components/course-details-updated/course-details-updated.html` | Replace price `<p>` block with clean HTL |

---

## Java Changes — `CourseDetailsModel.java`

Add one method near the existing `getPrice()` block (~line 351):

```java
public String getPriceLabel() {
    double price = course != null ? course.price : 0;
    long whole = (long) price;
    return price == whole ? String.valueOf(whole) : String.valueOf(price);
}
```

- Strips `.0` for whole-number prices (`0.0` → `"0"`, `100.0` → `"100"`).
- Preserves decimals for fractional prices (`99.99` → `"99.99"`).
- No behavioural change to existing `getPrice()` — safe, additive change.

---

## HTL Changes — `course-details-updated.html` (lines 74–79)

Replace the current block:

```html
<p class="dds__price_tc">
<!--                    ${model.price == 0 ? 'Free' : model.price}-->
<!--                    ${model.price == 0 ? '' : model.currency} -->
<!--                    / ${model.trainingCredit} Training Credits </p>-->
                        ${model.price} ${model.currency} / ${model.trainingCredit} Training Credits
                    </p>
```

With:

```html
<p class="dds__price_tc">${model.priceLabel} ${model.currency} / ${model.trainingCredit} Training Credits</p>
```

- Dead commented-out code is removed.
- `${model.priceLabel}` handles all formatting; currency and training credits are always shown.

---

## Verification

1. **Zero price**: `course.price = 0.0` → renders `0 USD / X Training Credits`.
2. **Whole-number price**: `course.price = 100.0` → renders `100 USD / X Training Credits`.
3. **Fractional price**: `course.price = 99.99` → renders `99.99 USD / X Training Credits`.
4. Build and deploy to AEM local instance; navigate to a course-details page for each case above.
5. Confirm the stale commented-out code is fully removed.
