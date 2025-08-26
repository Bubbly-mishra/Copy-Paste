Current situation (before fix)

You have a Party creation service.

Whenever a party is created → it calls a duplicate check service.

The duplicate check service calls Client Index Service to fetch possible duplicates.

Automations also simulate this flow → so automation ends up calling Client Index too many times, causing overload/errors.

Fix you’ve already applied

For automation, if the party name is in a predefined list (e.g., "Black Rock"),
→ you skip the Client Index call and just return a mock duplicate party response.

This reduces load on Client Index ✅.

Now the issue

After duplicates are found:

A blocking task is created (e.g., approve/reject duplicate party).

Approving/rejecting requires referencing another party (e.g., "Reject because Black Rock already exists").

Automation still needs duplicates to simulate this approval/rejection step.

Now you’re debating two approaches:

Option 1: Continue using Duplicate Service + Mock Response

In automation steps, when calling the duplicate service with "Black Rock",
→ Duplicate Service again skips Client Index call,
→ returns mock duplicate data,
→ automation proceeds with approve/reject task using this mock data.

✅ Pros

Consistency: Automation flow is the same as real flow (calls duplicate service → gets duplicates → creates blocking task → approve/reject).

Less maintenance: You don’t have to handle a separate “dummy party” concept inside automation.

Future-proof: If duplicate service logic changes, automation still reflects the real system behavior.

❌ Cons

Extra overhead: Even though Client Index isn’t called, Duplicate Service is still invoked multiple times.

More moving parts: Mocking duplicate responses adds complexity if test cases scale.

Option 2: Create a Dummy Party Inside Automation (Skip Duplicate Service)

Instead of calling duplicate service at this step, automation directly creates a dummy party (like "AutomationDummyParty") and uses it to simulate reject/approve logic.

This bypasses duplicate service completely in automation flows.

✅ Pros

Lightweight: No duplicate service calls at all during automation.

Faster: Eliminates even mock duplicate lookups.

No risk of accidental Client Index calls (since duplicate service is fully bypassed).

❌ Cons

Inconsistent with real flow: Automation behaves differently than production (since real system always calls duplicate service).

Harder to maintain: If duplicate service changes behavior (e.g., new fields, logic), automation won’t capture those changes.

Higher test fragility: Test data may drift apart from real system responses.

My Recommendation

It depends on what you value more:

If automation must stay close to production behavior → Option 1 (use Duplicate Service with mock response) is better. It keeps the test realistic and easier to maintain in the long run.

If performance & simplicity matter more for automation (and you’re okay with a less realistic flow) → Option 2 (dummy party) is leaner and avoids duplicate service entirely.
