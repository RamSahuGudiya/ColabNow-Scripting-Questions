# ServiceNow Scripting — Practice & Interview Questions

A structured collection of **ServiceNow server-side scripting practice questions, interview scenarios, and practical solutions** covering GlideRecord, GlideAggregate, Business Rules, Script Includes, reference qualifiers, attachments, and real-world automation scenarios.

---

## 📚 Table of Contents

- [1. Background Scripting](#1-background-scripting)
  - [1.1 Low-Level Questions](#11-low-level-questions)
  - [1.2 Medium-Level Questions](#12-medium-level-questions)
- [2. Scripting Interview Questions](#2-scripting-interview-questions)
  - [2.1 Attachment Handling](#21-attachment-handling)
  - [2.2 Incident and Problem Synchronization](#22-incident-and-problem-synchronization)
  - [2.3 Users Logged In During Last 90 Days](#23-users-logged-in-during-last-90-days)
  - [2.4 Category → Assignment Group → Incident](#24-category--assignment-group--incident)
  - [2.5 Create Incident from Problem](#25-create-incident-from-problem)
  - [2.6 Download a Record as XML](#26-download-a-record-as-xml)
  - [2.7 Async Business Rule](#27-async-business-rule)
  - [2.8 Dynamic Reference Qualifier](#28-dynamic-reference-qualifier)
  - [2.9 Reference Field Ordering](#29-reference-field-ordering)
  - [2.10 Least Assigned User](#210-least-assigned-user)
- [3. Additional Practice Scenarios](#3-additional-practice-scenarios)

---

# 1. Background Scripting

Background scripting is useful for practicing ServiceNow server-side APIs such as `GlideRecord`, `GlideAggregate`, queries, filtering, grouping, and record manipulation.

---

## 1.1 Low-Level Questions

### Q1. Fetch All Incidents

**Question**

Write a Background Script to fetch all Incident records and print their Incident numbers.

**Expected Output**

```text
INC0000001
INC0000002
INC0000003
```

---

### Q2. Fetch Active Incidents

**Question**

Fetch all active incidents and print:

- Incident Number
- Short Description

---

### Q3. Fetch Priority 1 Incidents

**Question**

Find all incidents where Priority = `1 - Critical` and print their incident numbers.

---

### Q4. Fetch Hardware Incidents

**Question**

Find all incidents where Category = `Hardware` and print:

- Incident Number
- Short Description

---

### Q5. Fetch Incidents Assigned to Abel Tuter

**Question**

Find all incidents assigned to **Abel Tuter** and print:

- Incident Number
- Short Description

---

### Q6. Fetch Incidents for a Specific Caller

**Question**

Find all incidents raised by **Demo User** and print the incident numbers.

---

### Q7. Fetch Unassigned Incidents

**Question**

Find all incidents where **Assignment Group** is empty and print their incident numbers.

---

### Q8. Fetch Incidents with No Assigned User

**Question**

Find all incidents where **Assigned To** is empty and print their incident numbers.

---

### Q9. Fetch One Incident

**Question**

Using a specific Incident `sys_id`, retrieve that incident and print:

- Number
- Short Description
- Priority

---

### Q10. Fetch Latest Incident

**Question**

Find the latest created incident and print:

- Number
- Short Description
- Created Date

---

# 1.2 Medium-Level Questions

These questions introduce multiple conditions, aggregation, date-based queries, and relationships between fields.

---

### Q11. Active Priority 1 Incidents

**Question**

Find incidents where:

- Active = `true`
- Priority = `1`

Print:

- Number
- Short Description
- Assignment Group

---

### Q12. Hardware Incidents Assigned to a Group

**Question**

Find all Hardware incidents assigned to a specific assignment group.

Print:

- Number
- Short Description
- Assigned Group

---

### Q13. Incidents Assigned to a Particular User

**Question**

Find all incidents assigned to **Abel Tuter**.

Print:

- Number
- Priority
- State

---

### Q22. Assignment Group with the Highest Incident Count

**Question**

Find the assignment group that has the highest number of incidents.

**Display**

| Assignment Group | Incident Count |
|---|---:|
| Software | 12 |

---

### Q23. Active Incidents by Assignment Group

**Question**

Using `GlideAggregate`, find the number of active incidents for each assignment group.

Display only groups having at least **3 active incidents**.

---

### Q24. Incidents Open for More Than 5 Days

**Question**

Find all incidents that have been open for more than 5 days.

Print:

- Incident Number
- Created Date
- State
- Short Description

---

### Q25. Incidents Created in the Last 7 Days

**Question**

Find incidents created within the last 7 days.

Print:

- Number
- Created Date
- Caller
- Short Description

---

### Q26. Caller and Assigned To Comparison

**Question**

Find incidents where the caller and assigned user are different people.

Print:

| Incident | Caller | Assigned To |
|---|---|---|
| INC0010001 | Demo User | Abel Tuter |

---

### Q27. Group Populated but Assigned To Empty

**Question**

Find incidents where:

- Assignment Group is populated
- Assigned To is empty
- Active = `true`

Print:

- Incident Number
- Assignment Group

---

### Q28. Priority 1 but No Assignment

**Question**

Find active Priority 1 incidents where either:

- Assignment Group is empty, **or**
- Assigned To is empty

Print:

- Incident
- Priority
- Assignment Group
- Assigned To

---

### Q29. Latest Incident for Each Assignment Group

**Question**

Using `GlideAggregate`, identify the assignment groups and their incident counts, then determine the latest-created incident for each group.

Print:

- Assignment Group
- Latest Incident
- Created Date

---

# 2. Scripting Interview Questions

This section contains practical ServiceNow scripting scenarios that are useful for technical interviews and real project development.

---

## 2.1 Attachment Handling

### Scenario

Copy an attachment from an Incident record to its associated Problem record.

### Solution

```javascript
(function executeRule(current, previous /*null when async*/) {

    var gr = new GlideRecord("problem");

    gr.addQuery('parent', current.getUniqueValue());
    gr.query();

    if (gr.next()) {

        GlideSysAttachment.copy(
            "incident",
            current.getUniqueValue(),
            "problem",
            gr.getUniqueValue()
        );

    }

})(current, previous);
```

### Key Concepts

- `GlideRecord`
- `GlideSysAttachment`
- `sys_id`
- Incident → Problem relationship

---

# 2.2 Incident and Problem Synchronization

## Scenario

When the work note on an Incident is updated, the same work note should also be added to the associated Problem.

### Business Rule

```javascript
(function executeRule(current, previous /*null when async*/) {

    var worknote = current.work_notes.getJournalEntry(1);

    updateWorkNotes(
        current.problem_id,
        worknote
    );

})(current, previous);
```

### Script Include

```javascript
function updateWorkNotes(problemId, worknotes) {

    var gr = new GlideRecord('problem');

    gr.addQuery('sys_id', problemId);
    gr.query();

    if (gr.next()) {

        gr.work_notes = worknotes;
        gr.update();

    }
}
```

### Flow

```text
Incident Work Note Updated
          │
          ▼
    Business Rule
          │
          ▼
 getJournalEntry(1)
          │
          ▼
    Script Include
          │
          ▼
 Find Associated Problem
          │
          ▼
 Update Problem Work Notes
```

---

# 2.3 Users Logged In During the Last 90 Days

### Question

Write a server-side script to retrieve users who logged in during the last 90 days.

### Solution

```javascript
var gr = new GlideRecord('sys_user');

var days = new GlideDateTime();

// Get the current date and time
days.addDaysUTC(-90);

gr.addQuery(
    'last_login_time',
    ">=",
    days
);

gr.query();

while (gr.next()) {

    gs.info(
        gr.name +
        " logged in at: " +
        gr.last_login_time
    );
}
```

### Key Concepts

- `GlideDateTime`
- Date manipulation
- `addDaysUTC()`
- Querying `sys_user`

---

# 2.4 Category → Assignment Group → Incident

## Scenario

Write a ServiceNow server-side script to:

1. Retrieve all incident categories.
2. For each category, print all incident numbers belonging to that category.
3. Within the same category, group incidents by assignment group.
4. For each assignment group, print the group name and incident numbers.

### Expected Structure

```text
Category: Hardware

    Incident: INC0010001
    Incident: INC0010002

    Assignment Group: Hardware

        Incident: INC0010001
        Incident: INC0010002

    Assignment Group: Network

        Incident: INC0010003
```

### Solution

```javascript
var ga = new GlideAggregate('incident');

ga.addNotNullQuery('category');
ga.groupBy('category');
ga.query();

while (ga.next()) {

    var category = ga.getValue('category');

    gs.info("Category: " + category);

    // Incidents under this category
    var gr = new GlideRecord('incident');

    gr.addQuery('category', category);
    gr.query();

    while (gr.next()) {

        gs.info(
            "   Incident: " +
            gr.number
        );
    }

    // Assignment groups under this category
    var ga2 = new GlideAggregate('incident');

    ga2.addQuery('category', category);
    ga2.groupBy('assignment_group');
    ga2.query();

    while (ga2.next()) {

        var group =
            ga2.getDisplayValue('assignment_group');

        gs.info(
            "   Assignment Group: " +
            group
        );

        // Incidents under this assignment group
        var gr2 = new GlideRecord('incident');

        gr2.addQuery('category', category);
        gr2.addQuery(
            'assignment_group',
            ga2.assignment_group
        );

        gr2.query();

        while (gr2.next()) {

            gs.info(
                "      Incident: " +
                gr2.number
            );
        }
    }
}
```

### Logic

```text
Incident Table
      │
      ▼
 Group by Category
      │
      ├── Hardware
      │      │
      │      └── Group by Assignment Group
      │              │
      │              ├── Hardware Team
      │              └── Network Team
      │
      └── Software
             │
             └── Group by Assignment Group
```

---

# 2.5 Create Incident from Problem

## Scenario

If the Priority of a Problem is changed to **Critical (P1)**, create an associated Incident.

The caller of the Incident should be the **currently logged-in user**.

### Solution

```javascript
(function executeRule(current, previous /*null when async*/) {

    var inc = new GlideRecord('incident');

    inc.initialize();

    inc.problem_id = current.sys_id;

    inc.caller_id = gs.getUserID();

    inc.short_description =
        "This is tested to create a inc for problem";

    inc.insert();

    gs.addInfoMessage(
        'This is tested for inc to problem ' +
        gs.getUserID()
    );

})(current, previous);
```

### Key Concepts

- `initialize()`
- `insert()`
- `current.sys_id`
- `gs.getUserID()`
- Reference fields

---

# 2.6 Download a Record as XML

## Scenario

Add a download button to a ServiceNow record form.

When the user clicks the button, the current record should be downloaded as XML.

### Solution

```javascript
function downloadRecord() {

    var recordSysId =
        g_form.getUniqueValue();

    var tableName =
        g_form.getTableName();

    var url =
        '/' +
        tableName +
        '.do?XML=&sys_id=' +
        recordSysId;

    var link =
        document.createElement('a');

    link.href = url;

    link.download =
        tableName +
        '_' +
        recordSysId +
        '.xml';

    link.click();
}
```

### Key Concepts

- `g_form.getUniqueValue()`
- `g_form.getTableName()`
- Client-side JavaScript
- ServiceNow XML record endpoint

---

# 2.7 Async Business Rule

## Scenario

Try to implement the following scenario using an **Async Business Rule**:

> An incident whose caller is VIP and whose priority is P1 is resolved. Create a Problem.

### Solution

```javascript
(function executeRule(current, previous /*null when async*/) {

    gs.info(
        "Async Business Rule is working for incident: " +
        current.number +
        " VIP Caller " +
        current.caller_id.vip
    );

    if (
        current.priority == "1" &&
        current.caller_id.vip
    ) {

        gs.info(
            "Successfully created a problem and async has run as expected."
        );

        var prb =
            new GlideRecord('problem');

        prb.initialize();

        prb.short_description =
            current.short_description;

        prb.description =
            current.description;

        var created_problem =
            prb.insert();

        current.setValue(
            'problem_id',
            created_problem
        );

        current.update();
    }

})(current, previous);
```

### Important Concepts

- Async Business Rule
- VIP caller
- Priority
- Creating related records
- `insert()`
- `current.update()`

---

# 2.8 Dynamic Reference Qualifier

## Scenario

Dynamically filter a reference field in the Incident table, such as `Caller`.

The reference field should display only users who belong to the **same department as the currently logged-in user**.

### Script Include

```javascript
var deptDynamic = Class.create();

deptDynamic.prototype = {

    initialize: function() {},

    dname: function() {

        var gr =
            new GlideRecord("sys_user");

        gr.addQuery(
            'sys_id',
            gs.getUserID()
        );

        gr.query();

        if (gr.next()) {

            return "department=" +
                   gr.getValue("department");
        }
    },

    type: 'deptDynamic'
};
```

### Logic

```text
Current Logged-In User
          │
          ▼
     Get User Record
          │
          ▼
       Get Department
          │
          ▼
 Return Reference Qualifier
          │
          ▼
 Filter Caller Reference Field
```

---

# 2.9 Reference Field Ordering

## Scenario

Display reference field records in alphabetical order by name.

### Reference Qualifier / Attribute

```text
encode_utf8=false,
ref_contributions=user_show_incidents,
ref_ac_order_by=last_name,
ref_qual_elements=last_name
```

### Key Concept

The `ref_ac_order_by` attribute controls the ordering of records displayed in the reference autocomplete.

---

# 2.10 Least Assigned User

## Scenario

Assign the current ticket to the user who has the **least number of assigned incidents**.

### Approach

1. Retrieve incidents.
2. Identify the assigned user.
3. Count incidents for each user.
4. Find the user with the lowest count.
5. Assign the current record to that user.

### Solution

```javascript
var userTicketCount = {};

var leastAssignedUser = null;

var leastCount = 99999;

var inc =
    new GlideRecord('incident');

inc.query();

while (inc.next()) {

    var assignedTo =
        inc.assigned_to.toString();

    if (assignedTo) {

        if (!userTicketCount[assignedTo]) {
            userTicketCount[assignedTo] = 0;
        }

        userTicketCount[assignedTo]++;
    }
}

// Find the user with the least ticket count
for (var userId in userTicketCount) {

    if (
        userTicketCount[userId] <
        leastCount
    ) {

        leastCount =
            userTicketCount[userId];

        leastAssignedUser =
            userId;
    }
}

if (leastAssignedUser) {

    current.assigned_to =
        leastAssignedUser;
}
```

### Logic

```text
Incident Records
      │
      ▼
Find Assigned Users
      │
      ▼
Count Incidents Per User
      │
      ▼
Compare Counts
      │
      ▼
Find Minimum
      │
      ▼
Assign Current Incident
```

---

# 3. Additional Practice Scenarios

## Scenario 1 — Incident Work Notes to Problem

**Requirement**

When an Incident work note changes, update the work note of the associated Problem.

### Business Rule

```javascript
(function executeRule(current, previous /*null when async*/) {

    updateworknotes(
        current.problem_id,
        current.work_notes.getJournalEntry(1)
    );

})(current, previous);
```

### Script Include

```javascript
var updateworknotes = function(
    problem_id,
    worknotes
) {

    var prb =
        new GlideRecord('problem');

    prb.addQuery(
        "sys_id",
        problem_id
    );

    prb.query();

    if (prb.next()) {

        prb.work_notes =
            worknotes;

        prb.update();
    }
}
```

---

# 4. ServiceNow Scripting Concepts Covered

| Area | Concepts |
|---|---|
| GlideRecord | Querying, filtering, inserting, updating |
| GlideAggregate | Counting, grouping, aggregation |
| Business Rules | Synchronous and asynchronous processing |
| Script Includes | Reusable server-side logic |
| Journal Fields | `getJournalEntry()` |
| Attachments | `GlideSysAttachment.copy()` |
| Users | `sys_user`, current user |
| Date Queries | `GlideDateTime` |
| Reference Fields | Reference qualifiers |
| Client Scripting | `g_form` |
| Record Relationships | Incident ↔ Problem |
| XML | Record XML download |
| JavaScript | Objects, loops, functions |
| Assignment Logic | Least-assigned user |

---

# 5. Interview Preparation Roadmap

## 🟢 Beginner

Focus on:

- What is `GlideRecord`?
- `addQuery()`
- `query()`
- `next()`
- `getValue()`
- `getDisplayValue()`
- `getUniqueValue()`
- Basic filtering
- Reference fields

---

## 🟡 Intermediate

Focus on:

- `GlideAggregate`
- `groupBy()`
- `addAggregate()`
- Multiple conditions
- Date queries
- Journal fields
- Related records
- `insert()`
- `update()`
- `deleteRecord()`

---

## 🔴 Advanced

Focus on:

- Business Rules
- Async Business Rules
- Script Includes
- GlideAjax
- Reference Qualifiers
- `GlideSysAttachment`
- Complex record relationships
- Aggregation and grouping
- Performance considerations
- Avoiding unnecessary queries

---

# 6. Recommended Practice Method

For each question, follow this process:

### Step 1 — Understand the Requirement

Identify:

- Table
- Fields
- Conditions
- Expected output

### Step 2 — Decide the API

For example:

```text
Need individual records
        ↓
   GlideRecord

Need count/grouping
        ↓
   GlideAggregate

Need reusable server logic
        ↓
   Script Include

Need automatic execution
        ↓
   Business Rule
```

### Step 3 — Write the Query

Start with the smallest possible query.

### Step 4 — Process the Records

Use:

```javascript
while (gr.next()) {
    // logic
}
```

### Step 5 — Validate the Result

Use:

```javascript
gs.info();
```

to verify the output.

### Step 6 — Think About Performance

Before using a script in production, consider:

- Can the query be more specific?
- Can unnecessary records be avoided?
- Is `GlideAggregate` better than retrieving every record?
- Is the logic running synchronously?
- Could the operation be asynchronous?
- Could repeated queries be avoided?

---

# 7. Quick Reference

## GlideRecord

```javascript
var gr = new GlideRecord('incident');

gr.addQuery('active', true);

gr.query();

while (gr.next()) {
    gs.info(gr.number);
}
```

## GlideAggregate

```javascript
var ga =
    new GlideAggregate('incident');

ga.addAggregate('COUNT');

ga.groupBy('assignment_group');

ga.query();

while (ga.next()) {

    gs.info(
        ga.getDisplayValue('assignment_group') +
        ' : ' +
        ga.getAggregate('COUNT')
    );
}
```

## Current User

```javascript
gs.getUserID();
```

## Current Record Sys ID

```javascript
current.getUniqueValue();
```

## Display Value

```javascript
gr.getDisplayValue('assignment_group');
```

## Field Value

```javascript
gr.getValue('assignment_group');
```

## Insert Record

```javascript
var gr =
    new GlideRecord('incident');

gr.initialize();

gr.short_description =
    'Test Incident';

gr.insert();
```

## Update Record

```javascript
gr.short_description =
    'Updated Description';

gr.update();
```

---

# 8. Final Goal

The purpose of this repository is to build practical ServiceNow scripting skills through progressively harder problems.

The focus is not only on memorizing APIs, but on understanding:

```text
Requirement
     ↓
Identify Table
     ↓
Identify Fields
     ↓
Build Query
     ↓
Retrieve Records
     ↓
Process Data
     ↓
Perform Action
     ↓
Validate Result
     ↓
Optimize
```

> **Practice the requirement first, then write the script.**
>
> The goal is to understand why the script works, not just memorize the syntax.

---

## 🛠 Technologies & APIs

- ServiceNow
- JavaScript
- GlideRecord
- GlideAggregate
- GlideDateTime
- GlideSysAttachment
- Business Rules
- Async Business Rules
- Script Includes
- Reference Qualifiers
- Client-side `g_form`
- ServiceNow Background Scripts
