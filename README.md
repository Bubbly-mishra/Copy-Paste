# Duplicate Party Handling in Automation

## Current Situation
- A **Party creation service** exists.  
- Whenever a party is created → it calls a **Duplicate Check Service**.  
- Duplicate Check Service → calls **Client Index Service** to fetch possible duplicates.  
- Automations replicate this flow → but cause **too many Client Index calls**, leading to overload/errors.  

---

## Fix Already Applied
- For automation, if the party name is in a **predefined list** (e.g., `"Black Rock"`):  
  - **Skip Client Index call**.  
  - Return a **mock duplicate party response**.  
- This reduces load on Client Index ✅.  

---

## New Issue
- After duplicates are found:  
  1. A **blocking task** is created (approve/reject duplicate party).  
  2. Approving/rejecting requires referencing another party (e.g., *Reject because Black Rock already exists*).  
  3. Automation still needs duplicates to simulate this approval/rejection step.  

---

## Approaches

### Option 1: Continue Using Duplicate Service + Mock Response
- In automation, when calling Duplicate Service with `"Black Rock"`:  
  - Skip Client Index call.  
  - Return mock duplicate data.  
  - Automation proceeds with approve/reject task using mock data.  

**Pros ✅**
- Consistent with real flow.  
- Less maintenance (no dummy party concept needed).  
- Future-proof (automation adapts if Duplicate Service logic changes).  

**Cons ❌**
- Still calls Duplicate Service multiple times (though lightweight).  
- Mocking duplicate responses may get complex as test cases scale.  

---

### Option 2: Create a Dummy Party in Automation (Skip Duplicate Service)
- Automation directly creates a **dummy party** (e.g., `"AutomationDummyParty"`) to simulate reject/approve logic.  
- Completely bypasses Duplicate Service in automation steps.  

**Pros ✅**
- Lightweight and faster.  
- No Duplicate Service calls at all.  
- Zero risk of accidental Client Index calls.  

**Cons ❌**
- Flow diverges from production (inconsistent).  
- Harder to maintain if Duplicate Service logic changes.  
- Test data may drift from real responses.  

---

## Recommendation
- If **automation should mirror production** → choose **Option 1 (mock via Duplicate Service)**.  
- If **performance and simplicity** are more important → choose **Option 2 (dummy party)**.  

👉 Suggested: **Option 1**. Automations are meant to validate **real-world flows**, so keeping them consistent avoids false positives.  

---

## Hybrid Approach (Optional)
- **Functional Tests** → Use **Option 1** with mock duplicate service calls.  
- **Load/Stress Tests** → Use **Option 2** with dummy parties to avoid duplicate service calls entirely.  
