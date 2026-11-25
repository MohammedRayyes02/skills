# Examples: How This Skill Changes Decisions

Each example shows the tempting default, what a senior engineer does instead, and why.

## Contents
1. Reuse before creating
2. Abstraction that earns its place
3. Scope discipline
4. Failing test
5. Object-level authorization
6. N+1 query
7. Frontend states and semantics
8. Asking vs assuming
9. Honest reporting

---

## 1. Reuse before creating

**Task:** format a price in the order summary.

**Bad:** write a new `formatPrice()` inside the component.
**Good:** search first (`grep -ri "currency\|formatMoney\|Intl.NumberFormat" src/`), find `lib/money.ts: formatMoney`, import it.
**Why:** a second formatter drifts from the first (rounding, locale) and creates two places to fix.

## 2. Abstraction that earns its place

**Task:** add CSV export for orders; users export reports too, later.

**Bad:**
```python
class ExporterFactory:
    def create(self, kind): ...
class AbstractExporter(ABC): ...
class OrderExporter(AbstractExporter): ...
```
Factory, abstract base, and one implementation, for a need that exists once.

**Good:**
```python
def orders_to_csv(orders):
    ...
```
When the second export arrives and the shared part is visible, extract the shared writer then.
**Why:** an abstraction guessed from one example usually fits the wrong axis of change.

## 3. Scope discipline

**Task:** fix the date shown on the invoice page (off by one day for some users).

**Bad:** fix the bug, also rename variables, reformat the file, upgrade the date library, convert the class to hooks.
**Good:** fix the timezone handling in the one function, add a regression test, and write: "Noticed `formatDate` is duplicated in `ReportRow`; not changed, want me to unify it?"
**Why:** the diff stays reviewable and revertable; the cleanup stays the user's decision.

## 4. Failing test

**Situation:** after the change, `test_total_includes_tax` fails.

**Bad:** change the expected value to match the output, or mark the test skipped.
**Good:** work out which is wrong. If the test encodes the real rule, fix the code. If the rule legitimately changed per the requirement, update the test and say so explicitly in the report.
**Why:** a test edited to pass proves nothing, and silent edits hide behavior changes.

## 5. Object-level authorization

**Task:** add `GET /documents/{id}`.

**Bad:**
```js
const doc = await Document.findById(req.params.id);
res.json(doc);
```
Any logged-in user can read any document by guessing ids.

**Good:**
```js
const doc = await Document.findOne({ _id: req.params.id, ownerId: req.user.id });
if (!doc) return res.status(404).json({ error: "Not found" });
res.json(doc);
```
Or use the project's policy/guard mechanism if one exists.
**Why:** insecure direct object reference is one of the most common real vulnerabilities; ownership must be part of the lookup.

## 6. N+1 query

**Task:** list projects with their owner names.

**Bad:**
```php
foreach (Project::all() as $project) {
    echo $project->owner->name;   // one extra query per row
}
```
**Good:**
```php
$projects = Project::with('owner:id,name')->paginate(25);
```
**Why:** 1 + N queries become 2, and the unbounded `all()` becomes a page.

## 7. Frontend states and semantics

**Task:** show a list of tasks fetched from an API.

**Bad:** render `tasks.map(...)` only; blank screen while loading, on error, and when empty. Rows are `<div onClick>`.

**Good:** loading skeleton, error message with retry, empty state with "Create your first task", rows as real links/buttons with labels, keyboard operable.
**Why:** users spend most of their time in the non-happy states; semantic elements give keyboard and screen-reader support for free.

## 8. Asking vs assuming

**Case A (ask):** "Add role-based access." Which roles, which resources, and who assigns them are unknown and change the data model. Ask one focused question before building.

**Case B (assume):** "Add a `createdAt` sort to the list." Descending is the obvious default and trivially changed. Implement it, state "sorted newest first", move on.
**Why:** ask when a wrong guess is expensive; assume when it is cheap to undo.

## 9. Honest reporting

**Bad:** "Implemented the feature and all tests pass." (tests were never run)

**Good:**
> Implemented password reset in `AuthController` and `PasswordResetService`. Token expiry is 60 min, matching the existing email-verification setting.
> Verified: ran `php artisan test --filter=PasswordReset` (6 passed) and `./vendor/bin/pint --test` (clean). Not verified: actual email delivery (no mail server in this environment).
> Risk: no rate limit on the reset endpoint yet; recommend adding one.

**Why:** the reader can trust what was checked and knows exactly what remains.
