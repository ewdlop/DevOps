# DevOps

## @ByteByteGo
[How Big Tech Ships Code]<https://www.youtube.com/shorts/551lh10g_go>

## Azure DevOps Repos 

## It is a version control system provided by Microsoft as part of its **Azure DevOps Services** platform. It enables teams to manage their code repositories, collaborate effectively, and implement best practices in software development. Here's an overview of its key features and how it can be used effectively:

---

## **Features of Azure DevOps Repos**
### **1. Version Control**
- **Supports Git and TFVC (Team Foundation Version Control):**
  - Git is a distributed version control system.
  - TFVC is a centralized version control system (legacy option).

### **2. Repository Management**
- Unlimited private repositories.
- Easy integration with other Azure DevOps tools like Boards, Pipelines, and Artifacts.
- Branching and merging strategies supported.

### **3. Code Collaboration**
- Pull Requests (PRs): Collaborate on code changes with inline commenting and discussion.
- Code Reviews: Ensure code quality before merging to main branches.
- Commit history and comparison tools.

### **4. Security and Compliance**
- Fine-grained access control at the repo, branch, or folder level.
- Integration with Azure Active Directory for identity and access management.
- Compliance with industry standards like ISO, SOC, and GDPR.

### **5. Built-in CI/CD Integration**
- Seamless integration with Azure Pipelines for automated builds, tests, and deployments.
- Supports integration with GitHub Actions and Jenkins.

### **6. Integration with Development Tools**
- Integrates well with Visual Studio, VS Code, and JetBrains tools.
- Command-line Git support for Linux, macOS, and Windows.

---

## **Use Cases**
### **1. Managing Source Code**
- Store and manage application source code securely in private repositories.
- Use branching strategies to organize work, e.g., GitFlow or trunk-based development.

### **2. Collaborating Across Teams**
- Use pull requests for collaborative code reviews.
- Enable team discussions on commits or PRs for knowledge sharing and peer feedback.

### **3. Automating Workflows**
- Trigger builds and deployments automatically with Azure Pipelines when code is pushed or PRs are merged.

### **4. Governance and Compliance**
- Use branch policies to enforce quality gates like requiring reviewers, linked work items, or build validation.

---

## **Best Practices**
1. **Branching Strategy:**
   - Adopt a branching model that suits your workflow (e.g., GitFlow, feature branching).
   - Protect main branches by requiring pull request reviews.

2. **Code Reviews:**
   - Enforce mandatory code reviews for PRs to ensure code quality.
   - Use automated tools to check for syntax errors and code standards.

3. **Integration:**
   - Set up CI/CD pipelines to ensure automated testing and deployment.
   - Leverage other Azure DevOps services like Boards for tracking work items and linking them to code changes.

4. **Security:**
   - Regularly audit repository permissions.
   - Use secrets management tools for sensitive data in pipelines.

---

## **Getting Started**
1. **Setup Azure DevOps Organization:**
   - Sign up for Azure DevOps Services or use an existing Azure subscription.
   - Create a new project and enable Repos.

2. **Clone Repository:**
   - Clone the repository to your local machine using `git clone`.

3. **Work with Code:**
   - Add, commit, and push changes using Git commands or integrated tools like Visual Studio.

4. **Collaborate:**
   - Create and review PRs.
   - Use branch policies to enforce code quality.

---

### **Resources:**
- [Azure DevOps Repos Documentation](https://learn.microsoft.com/en-us/azure/devops/repos/)
- [Azure DevOps Pricing](https://azure.microsoft.com/en-us/pricing/details/devops/)
- [Best Practices for Git in Azure Repos](https://learn.microsoft.com/en-us/azure/devops/repos/git/git-best-practices)


### Work Item Query Language

**WIQL (Work Item Query Language)** is a query language specific to Azure DevOps for filtering and retrieving work items. It is SQL-like and allows advanced querying of work items using the Azure DevOps API or query editor.

Here’s a comprehensive guide with **examples** to help you write WIQL queries.

---

### **1. Basic WIQL Syntax**
```wiql
SELECT [Fields]
FROM WorkItems
WHERE [Conditions]
ORDER BY [Fields] [ASC/DESC]
```

---

### **2. Examples of WIQL Queries**
#### **2.1. Retrieve All Active Work Items**
```wiql
SELECT [System.Id], [System.Title], [System.State]
FROM WorkItems
WHERE [System.State] = 'Active'
```

#### **2.2. User Stories Assigned to Me**
```wiql
SELECT [System.Id], [System.Title], [System.AssignedTo]
FROM WorkItems
WHERE [System.WorkItemType] = 'User Story'
  AND [System.AssignedTo] = @Me
```

#### **2.3. Bugs Created in the Last 7 Days**
```wiql
SELECT [System.Id], [System.Title], [System.CreatedDate]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [System.CreatedDate] >= @Today - 7
```

---

### **3. Advanced WIQL Queries**
#### **3.1. High-Priority Work Items**
```wiql
SELECT [System.Id], [System.Title], [Microsoft.VSTS.Common.Priority]
FROM WorkItems
WHERE [Microsoft.VSTS.Common.Priority] <= 2
```

#### **3.2. Work Items in a Specific Iteration**
```wiql
SELECT [System.Id], [System.Title], [System.IterationPath]
FROM WorkItems
WHERE [System.IterationPath] = 'ProjectName\Iteration1'
```

#### **3.3. Work Items Without Assigned Users**
```wiql
SELECT [System.Id], [System.Title], [System.AssignedTo]
FROM WorkItems
WHERE [System.AssignedTo] = ''
```

---

### **4. Linked Work Items**
#### **4.1. Find Child Work Items of a Specific Parent**
```wiql
SELECT [System.Id], [System.Title]
FROM WorkItemLinks
WHERE [Source].[System.Id] = 12345
  AND [System.Links.LinkType] = 'System.LinkTypes.Hierarchy-Forward'
MODE (Recursive)
```

#### **4.2. Work Items Blocking Others**
```wiql
SELECT [System.Id], [System.Title]
FROM WorkItemLinks
WHERE [Source].[System.State] = 'Active'
  AND [System.Links.LinkType] = 'System.LinkTypes.Dependency-Forward'
MODE (Recursive)
```

---

### **5. Queries with Tags**
#### **5.1. Work Items Containing a Specific Tag**
```wiql
SELECT [System.Id], [System.Title], [System.Tags]
FROM WorkItems
WHERE [System.Tags] CONTAINS 'Urgent'
```

#### **5.2. Work Items with Multiple Tags**
```wiql
SELECT [System.Id], [System.Title], [System.Tags]
FROM WorkItems
WHERE [System.Tags] CONTAINS 'Backend'
  AND [System.Tags] CONTAINS 'API'
```

---

### **6. Queries Using Date Ranges**
#### **6.1. Recently Updated Work Items**
```wiql
SELECT [System.Id], [System.Title], [System.ChangedDate]
FROM WorkItems
WHERE [System.ChangedDate] >= @Today - 7
```

#### **6.2. Work Items with a Target Date**
```wiql
SELECT [System.Id], [System.Title], [Microsoft.VSTS.Scheduling.TargetDate]
FROM WorkItems
WHERE [Microsoft.VSTS.Scheduling.TargetDate] <= @Today + 14
```

---

### **7. Sorting Queries**
#### **7.1. Sort by Priority**
```wiql
SELECT [System.Id], [System.Title], [Microsoft.VSTS.Common.Priority]
FROM WorkItems
WHERE [System.State] = 'Active'
ORDER BY [Microsoft.VSTS.Common.Priority] ASC
```

#### **7.2. Sort by Creation Date**
```wiql
SELECT [System.Id], [System.Title], [System.CreatedDate]
FROM WorkItems
ORDER BY [System.CreatedDate] DESC
```

---

### **8. Combining Multiple Conditions**
#### **8.1. Active User Stories Assigned to Me**
```wiql
SELECT [System.Id], [System.Title]
FROM WorkItems
WHERE [System.WorkItemType] = 'User Story'
  AND [System.State] = 'Active'
  AND [System.AssignedTo] = @Me
```

#### **8.2. Bugs in a Specific Area Path**
```wiql
SELECT [System.Id], [System.Title], [System.AreaPath]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [System.AreaPath] UNDER 'ProjectName\Area1'
```

---

### **9. Recursive Queries**
#### **9.1. Retrieve Hierarchical Work Items**
```wiql
SELECT [System.Id], [System.Title]
FROM WorkItemLinks
WHERE [Source].[System.TeamProject] = 'ProjectName'
  AND [System.Links.LinkType] = 'System.LinkTypes.Hierarchy-Forward'
MODE (Recursive)
```

---

### **10. Special WIQL Features**
#### **10.1. Use Macros**
- `@Me`: Logged-in user.
- `@Today`: Current date.
- `@CurrentIteration`: Current sprint.
- `@StartOfWeek`: Start of the current week.
- `@EndOfWeek`: End of the current week.

#### **10.2. Query Fields**
- `System.Id`
- `System.Title`
- `System.State`
- `System.CreatedDate`
- `System.AssignedTo`
- `Microsoft.VSTS.Common.Priority`
- `System.Tags`

---

### **Tips for Using WIQL**
1. **Write and Test in Azure DevOps**: Use the **Query Editor** in Azure DevOps to build and validate your WIQL queries.
2. **Combine WorkItemLinks and WorkItems**: For hierarchical data, use `WorkItemLinks` combined with `MODE (Recursive)`.
3. **Use Dynamic Macros**: Use macros like `@CurrentIteration` for queries that adjust dynamically.
4. **API Integration**: WIQL queries can be executed programmatically through the Azure DevOps REST API.

WIQL is extremely versatile for managing and querying Azure DevOps work items. Let me know if you'd like to dive deeper into any specific aspect!

Would you like assistance setting up a repository or integrating it with other tools?
