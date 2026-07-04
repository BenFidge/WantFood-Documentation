# Cross-portal test case template

> **For doc authors, not testers.** This is the standard shape every cross-portal "change here → check there" test case in `09-cross-portal-impact-uat.md` should follow. Copy this block, replace the placeholders, and delete this introductory note from the resulting test case.

The pattern is **Before → Change → Propagate → Verify After**. A non-technical tester should be able to follow each step without knowing how the platform is built.

---

## TC-XP-NNN: <Short, action-oriented title>

**Goal**: One sentence explaining what cross-portal behaviour this test proves.

**Personas / portals you need open at once**:

- ☐ System Admin (`admin.uat@wantfood.com`)
- ☐ Vendor Admin (`vendor.uat@wantfood.com`)
- ☐ Customer Front-end (`customer.uat@wantfood.com`) — incognito window recommended
- ☐ Driver Portal (`driver.uat@wantfood.com`) — only if relevant

> **Tip**: Use **three browser windows** (not three tabs) so you can see the admin change and the customer reaction side-by-side.

---

### Before — capture the starting state

1. As **<persona>**, open **<page name and URL>**.
2. **Take a screenshot** of the area you're about to affect.
3. Note in your bug tracker: *"Before: <one-line description of what you see, e.g., 4 vendors visible, second card is Pizza Roma."*

### Change — perform the admin action

1. Switch to **<admin portal>** as **<persona>**.
2. Go to **<admin page>** → **<section>** → **<exact button/menu>**.
3. Make the change: **<exact field values to enter>**.
4. Save / Publish / Approve as appropriate. Confirm the success toast.

### Propagate — give the system a chance to catch up

> Most admin changes appear on the front-end within **30 seconds**. If they don't, work through the propagation checklist below before logging a bug.

**Propagation checklist** (in order):

1. **Wait 30 seconds**, then hard-refresh the customer front-end (Ctrl+F5 / Cmd+Shift+R).
2. If still not visible, run **System Admin → Platform Tools → Rebuild Caches** and wait 30s.
3. For search/discovery-affecting changes only: run **System Admin → Platform Tools → Reindex Vendors** and wait 30s.
4. If still not visible after step 3, **log as a bug** (severity Medium) with the propagation steps you tried.

### Verify After — every front-end surface affected

Tick every box that applies to this change. **Do not skip any** — surprises usually live in the surface you didn't think to check.

- ☐ **Home page** (`/`): <what should now appear / disappear>
- ☐ **Search results** (`/search?q=<term>`): <expected change>
- ☐ **Cuisine page** (`/cuisine/<slug>`): <expected change>
- ☐ **Vendor page** (`/vendor/<id>`): <expected change>
- ☐ **Basket** (if customer had items): <expected change — clear? warn? unaffected?>
- ☐ **Checkout** (if customer is mid-checkout): <expected change>
- ☐ **Order confirmation / tracking** (for in-flight orders): <expected change>
- ☐ **Customer email**: <subject line, sent within X minutes>
- ☐ **Driver portal active deliveries** (if relevant): <expected change>
- ☐ **Vendor admin live order kanban** (if relevant): <expected change>

> Tick every box, even the ones where the expected outcome is **"no change"** — silent regressions on unrelated surfaces are exactly what cross-portal testing is meant to catch.

### Pass / Fail / Bug-anyway

- ✅ **Pass** if every applicable box was as expected.
- ❌ **Fail** if any box failed and the test can't proceed.
- 🐛 **Bug-anyway** if every box passed but you spotted something odd (slow propagation, confusing toast wording, missing screen-reader announcement, etc.). Log it as **Low** severity and keep going.

### Edge cases to also try

- **<edge case 1>** — describe in one line.
- **<edge case 2>**
- **<edge case 3>**

### Common pitfalls

- *"I changed it but the customer can't see it"* → 9 times out of 10 the customer browser still has the old page cached. Hard-refresh first, then work the propagation checklist above.
- *"The change shows but the email didn't arrive"* → check the **Junk / Spam** folder. Emails come from `noreply@wantfood.com` (example).
- *"The basket got cleared / a price changed mid-checkout"* → that's expected behaviour for many admin changes and **is what we are testing for**. Capture a screenshot of the customer-facing warning if there is one; flag if there isn't.

---

### Authoring checklist (delete from final TC)

- [ ] Did you list **every** front-end surface affected, not just the obvious one?
- [ ] Did you include the propagation checklist (or link to it)?
- [ ] Did you write the change in **exact click-by-click language** a non-technical tester can follow?
- [ ] Did you include a screenshot prompt in the **Before** step?
- [ ] Did you list at least two edge cases?
- [ ] Did you cross-reference the per-portal TC in `02-…` / `03-…` so testers running the admin-only doc are reminded to come here?
