Azure DevOps queries are a powerful way to filter, search, and analyze work items based on criteria you define. Below are **examples of Azure DevOps queries**:

---

### **1. Basic Queries**
#### **1.1. Find Active Bugs**
```plaintext
Work Item Type = Bug AND State = Active
```

#### **1.2. User Stories Assigned to a Specific User**
```plaintext
Work Item Type = User Story AND Assigned To = @Me
```
`@Me` dynamically sets the query to the logged-in user.

#### **1.3. Tasks in Progress**
```plaintext
Work Item Type = Task AND State = In Progress
```

#### **1.4. Recently Updated Work Items**
```plaintext
Changed Date >= @StartOfDay
```

---

### **2. Advanced Queries**
#### **2.1. Work Items Without Tags**
```plaintext
Tags = ""
```

#### **2.2. High-Priority Items**
```plaintext
Priority <= 2
```
This assumes priority is ranked, where `1` is the highest priority.

#### **2.3. Work Items Linked to a Specific Feature**
```plaintext
Work Item Type = User Story AND Parent[Feature] = "Feature Name"
```

#### **2.4. Find Work Items in a Specific Area**
```plaintext
Area Path UNDER "ProjectName\AreaName"
```

---

### **3. Sprint-Related Queries**
#### **3.1. Work Items in the Current Sprint**
```plaintext
Iteration Path = @CurrentIteration
```

#### **3.2. Sprint Backlog**
```plaintext
Iteration Path = @CurrentIteration AND (Work Item Type = Task OR Work Item Type = Bug)
```

#### **3.3. Unassigned Work in Sprint**
```plaintext
Iteration Path = @CurrentIteration AND Assigned To = ""
```

#### **3.4. Completed Work in the Last Sprint**
```plaintext
Iteration Path = @PreviousIteration AND State = Done
```

---

### **4. Custom Fields**
If your organization uses custom fields, you can query them:
```plaintext
[Custom Field Name] = "Custom Value"
```
For example:
```plaintext
[Customer Impact] = "High"
```

---

### **5. Queries for Linked Work Items**
#### **5.1. Find Child Work Items for a Parent**
```plaintext
Work Item Type = Task AND Parent = [Parent Work Item ID]
```

#### **5.2. Work Items with No Parent**
```plaintext
Work Item Type IN (Bug, User Story, Task) AND Parent = ""
```

#### **5.3. Work Items Blocking Others**
```plaintext
State = Active AND [Blocked By] = @Me
```

---

### **6. Queries for Dates**
#### **6.1. Created in the Last 7 Days**
```plaintext
Created Date >= @Today - 7
```

#### **6.2. Due in the Next 14 Days**
```plaintext
Target Date <= @Today + 14
```

#### **6.3. Items Modified This Week**
```plaintext
Changed Date >= @StartOfWeek
```

---

### **7. Queries for Tags**
#### **7.1. Items Tagged as "Urgent"**
```plaintext
Tags CONTAINS "Urgent"
```

#### **7.2. Items Tagged with Multiple Tags**
```plaintext
Tags CONTAINS "Frontend" AND Tags CONTAINS "API"
```

---

### **8. Multi-Team Queries**
#### **8.1. Work Items for a Specific Team**
```plaintext
Team Project = "Team Name"
```

#### **8.2. Work Items Across All Teams**
```plaintext
Team Project UNDER "OrganizationName"
```

---

### **9. Complex Logical Queries**
#### **9.1. Unresolved Bugs with High Priority**
```plaintext
Work Item Type = Bug AND State != Resolved AND Priority = 1
```

#### **9.2. User Stories Assigned to Specific Team Members**
```plaintext
Work Item Type = User Story AND Assigned To IN ("User1", "User2")
```

---

### **10. Saved Query Example**
If you frequently need a query, save it for reuse:
1. Open Azure DevOps.
2. Navigate to **Boards > Queries**.
3. Create a new query and use the desired filters.
4. Save the query for team access or personal use.

---

### **Tips for Optimizing Queries**
- Use **Folders** to organize saved queries.
- Combine conditions with `AND`, `OR`, and `NOT` for more complex logic.
- Use macros like `@Me`, `@Today`, `@CurrentIteration` to make queries dynamic.
- Leverage query sharing for team-wide visibility.

These examples can be tailored further based on your Azure DevOps environment and processes. Let me know if you want specific query modifications!
