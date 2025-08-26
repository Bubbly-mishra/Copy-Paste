# Handling Duplicate Party in Automation (Reject Case)

## Background
In the current system:
- A **Party Creation Service** is used to create new parties.  
- When a new party is created, the system calls the **Duplicate Check Service**.  
- The Duplicate Check Service, in turn, calls the **Client Index Service** to identify potential duplicates.  

### Problem
During **automation runs**, this process leads to:
- Multiple calls being made to the **Client Index Service**.  
- High load on Client Index, which can only handle limited requests efficiently.  
- Errors due to excessive parallel requests during test executions.  

---

## Implemented Fix
To reduce load on Client Index during automation:
- A **predefined list of party names** (e.g., `"Black Rock"`) has been introduced.  
- If an automation uses one of these party names:  
  - The Duplicate Check Service **skips calling Client Index**.  
  - Instead, it returns a **mock duplicate party response**.  

This approach ensures Client Index is protected from unnecessary load during test runs.  

---

## New Challenge: Reject Flow
After a duplicate is detected:
- A **blocking task** is created in the system.  
- To **reject** this task, a reference to an existing duplicate party must be provided.  

This means:  
- Even in automation, we still need a way to supply a duplicate party reference during **rejection**.  
- The question is how best to simulate this behavior without overloading Client Index.  

---

## Options

### **Option 1: Use Duplicate Service with Mock Responses**
In this approach:
- The automation calls the **Duplicate Check Service** with a predefined party name (e.g., `"Black Rock"`).  
- The Duplicate Service recognizes the predefined name, skips Client Index, and returns a **mock duplicate party**.  
- This mock duplicate is then used in the rejection flow.  

**Pros ✅**
- **Realistic**: Closely mirrors the actual production flow.  
- **Maintainable**: Any changes in Duplicate Service logic are automatically reflected in automation.  
- **Future-proof**: Keeps test data aligned with system behavior.  

**Cons ❌**
- Still involves Duplicate Service calls (though lightweight).  
- Requires maintaining mock response data.  

---

### **Option 2: Create a Dummy Party Directly in Automation**
In this approach:
- Automation bypasses the Duplicate Check Service entirely.  
- Instead, it creates a **dummy party** (e.g., `"AutomationDummyParty"`) that acts as a reference for rejection.  

**Pros ✅**
- **Lightweight**: No Duplicate Service calls at all.  
- **Fast**: Reduces overhead during automation runs.  
- **Simple**: No need for mock data setup in Duplicate Service.  

**Cons ❌**
- **Less realistic**: Flow diverges from production behavior.  
- **Maintenance overhead**: If duplicate handling logic changes in production, automation will not reflect it.  
- **Risk of false positives**: Tests may pass in automation but fail in production.  

---

## Recommendation
- For **functional and regression automation**:  
  Use **Option 1**. This ensures automation remains aligned with production workflows and validates the real duplicate handling process.  

- For **load or performance testing**:  
  Option 2 may be considered to reduce overhead and isolate performance testing from Client Index or Duplicate Service dependencies.  

👉 Overall, **Option 1 is the preferred approach** for long-term maintainability and accuracy of automation.  
