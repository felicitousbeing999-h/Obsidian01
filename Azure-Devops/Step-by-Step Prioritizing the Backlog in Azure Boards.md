## **Introduction**

This guide outlines an objective process for transforming recurring incidents into a prioritized automation plan using **Azure Boards**. You will learn to review your backlog, rank tasks using a clear **ROI formula**, and update your sprint plan to maximize value.

## **Phase 1: Preparing Your Backlog in Azure Boards**

Before you can prioritize, you need to ensure your data is set up correctly. This phase walks through configuring your Azure Boards project to capture the necessary information for ROI calculations. This typically requires project administrator privileges.

### **Step 1: Ensure Incidents Are Tracked as Work Items**

Ensure every recurring task or incident is a distinct work item in your backlog. Whether you use standard types like **User Story** or custom types like **"Toil,"** each potential automation must be tracked individually to be prioritized.

![Azure DevOps Boards page showing a backlog for the Automation-Strategy Team with three new user story work items.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_cb05652910a14dc498466a8ca6b3423d_Screenshot-2026-06-05-174750.png?expiry=1789111711909&hmac=DonJOHcr1nngi2rqmC3uuJ8gaYuynTzB2lC9WApjx-4)

The image displays a screenshot of the Azure DevOps web portal interface, focused on the "Backlogs" dashboard view within the "Automation-Strategy" project. On the far left, a vertical navigation sidebar highlights the active "Boards" section and its selected sub-option, "Backlogs," alongside other platform links like Overview, Repos, and Pipelines. The main display area header indicates this is the "Automation-Strategy Team" backlog workspace, offering several interactive action controls such as "+ New Work Item", "View as Board", and "Column Options". Beneath the section tabs for "Backlog" and "Analytics", a data table lists three distinct items filtered by the "Stories" category. Organized into columns for Order, Work Item Type, Title, and State, the backlog populates three sequential items, all designated as a "User Story" and marked with a grey status indicator as "New". The descriptions for these specific user stories read "Clear user session cache", "Manually scale up database", and "Restart web-app-prod-blue server".

### **Step 2: Add Custom Fields for ROI Calculation**

To calculate ROI, you need to add specific data points to your work items. Navigate to your project's process customization settings to add new fields.

1. Go to Organization Settings > Boards > Process.
    
2. If your project uses a built-in process (Agile, Scrum, CMMI), you must first create an inherited process: select the three-dot menu on the base process > Create inherited process > name it (for example, Engineering-Agile).
    
3. Select your inherited process > choose the work item type (for example, User Story) > select Fields > New field to add each custom field.
    
4. After adding fields, go to Organization Settings > Projects > select your project > Change process > switch to your inherited process.
    
5. Add the following custom fields:
    
    - **Manual Toil Hours Month**: A decimal number field to store the total hours spent per month manually resolving the incident.
        
    - **Incident Frequency per Month**: A number field for how many times the incident typically occurs in a month.
        
    - **Automation ROI Score**: A decimal number field that will hold the calculated ROI score. You will manually update this field based on a formula.
        
    - **Estimated Effort Hours**: A number field to estimate the engineering hours required to build the automation.
        

**Important:** Set Automation ROI Score as a **Decimal** field. If it is created as an Integer, values like **2.5** will be rounded to **3**, which breaks the prioritization logic.

![Azure DevOps "New work item" dialog showing a User Story being created with the title "Manually scale up database."](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_7811ccdc073b45d18b88afaa6d493538_Screenshot-2026-06-05-175314.png?expiry=1789111711909&hmac=NzHwF8VLYcx0tCLZE2XngoKod8KoWhjz1SXPOj1aOF8)

The image displays a screenshot of the Azure DevOps web interface inside the Organization Settings, specifically focusing on the process customization for a "User Story" work item type. On the far left, a vertical navigation sidebar under "Organization Settings" includes categories such as "General" with options like Overview, Projects, Users, Billing, Global notifications, Usage, Extensions, and Microsoft Entra, followed by sections for "Security" and "Boards". The main content area contains a breadcrumb trail at the top reading "All processes > Engineering-Agile > User Story", with three tabs below it: "Layout", "States", and "Rules", with the "Layout" tab currently selected. Below a toolbar featuring buttons for "New field", "New group", "New page", and "Get extensions", the workspace outlines the structural layout of a user story form template. The main column on the left shows a section titled "Details" containing a multi-line text field designated for "Acceptance Criteria". The center column outlines several data field attributes such as "Priority" (Integer), "Risk" (Text single line), "Classification", and a "Value area" block containing fields like "Manual Toil Hours Mo..." (Decimal), "Incident Frequency pe..." (Integer), "Automation ROI Score" (Decimal), and "Estimated Effort Hours" (Integer). The rightmost column displays sections for "Development" and "Related Work", both configured with "Links" attributes.

## **Phase 2: Applying an ROI Formula to Rank Items**

With the custom fields in place, you can now systematically evaluate each item in your backlog.

### **Step 3: Quantify Toil and Estimate Effort**

Go through each work item in your automation backlog and fill in the fields you created. This is a team activity.

- For **Manual Toil Hours Month** and **Incident Frequency per Month**, use data from your monitoring or ticketing systems. If you don't have precise data, work with the team to form a credible estimate.
    
- For **Estimated Effort Hours**, your engineering team should provide a rough estimate of how long it would take to design, build, test, and deploy the automation.
    

### **Step 4: Calculate and Input the ROI Score**

Now, apply a simple ROI formula to each work item. A powerful formula for prioritizing toil reduction using your new fields is:

ROI Score = Manual Toil Hours Month / Estimated Effort Hours

This formula prioritizes tasks that are frequent, time-consuming, and relatively cheap to automate. For each work item, calculate this score and enter it into the **Automation ROI Score** field.

**Example:** An incident takes 20 total hours of manual work per month. The team estimates it will take 4 hours of engineering effort to automate.

- **ROI Score** = 20 / 4 = **5.0**
    

Another incident takes 5 hours per month. It will take 2 hours to automate.

- **ROI Score** = 5 / 2 = **2.5**
    

The first item has a higher ROI score and should be prioritized.

**Note:** The ROI Score formula prioritizes high-impact, time-consuming tasks. For tasks with very short break-even periods (under 1 month), you may choose to override the score and pull them into the sprint regardless.

### **Step 5: Configure and Sort Your Backlog View**

To make prioritization easy, add your new fields as columns in the backlog view.

1. In your **Backlog** view, select **Column Options** from the toolbar.
    
2. Add **Automation ROI Score**, **Manual Toil Hours Month**, and **Estimated Effort Hours** as columns. Select **OK**.
    
3. To sort by ROI Score—since standard backlog views do not support column header sorting—select **Create Query** from the backlog toolbar.
    
4. In the query editor, ensure the query type is set to **Flat list of work items**.
    
5. Select **Column Options** → **Sorting** tab → add **Automation ROI Score** → set to **Descending** → select **OK**.
    
6. **Save** and **Run** the query.
    

Work items with the highest ROI scores will now appear at the top of your list. Bookmark this query for use in every sprint planning session. This provides you with an objectively ranked list of priorities based on data rather than intuition.

![Azure DevOps backlog page showing a "User Story" titled "Clear user session cache" with a "New" state.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_670e6738bed1427fb53f86d02daeb7fa_Screenshot-2026-06-05-175711.png?expiry=1789111711909&hmac=xJ5iMBdsiNK0QwIOddox_3sxJVlBHrOWvtTFdB3P6GY)

The image displays a screenshot of the Azure DevOps web interface inside the "Automation-Strategy" project, specifically showing a query view named "Automation ROI Ranking." On the far left, a vertical navigation sidebar highlights the active "Boards" category and its selected sub-option, "Queries," alongside other sidebar sections like Repos, Pipelines, Test Plans, and Project settings. In the central workspace, the query interface is toggled to the "Results" tab, revealing a data table with truncated column headers such as "Automat...", "Manual Toil...", and "Estimat..." across two rows containing numerical metrics. Partially overlaying the right side of the screen is a prominent white modal dialog box titled "Column options." This modal is switched to the "Sorting" tab, where a rule is configured to sort the data by the field "Automation ROI Score." A downward-pointing arrow button next to the field displays a dark tool tip reading "Sort descending," indicating the active sorting direction, while buttons for "+ Add a column", "Cancel", and "OK" anchor the configuration controls at the bottom right.

## **Phase 3: Updating the Sprint Board and Communicating the Strategy**

With a ranked backlog, you can now plan your sprint and communicate your decisions to stakeholders.

### **Step 6: Select Top Items for the Next Sprint**

During your sprint planning meeting, display the sorted backlog. Based on your team's capacity for the upcoming sprint, drag the highest-ROI work items from the backlog into the sprint.

1. Open the **Planning pane** in Azure Boards by selecting **View options** (the icon with three small bars) and toggling **Planning** to **On**.
    
2. From the backlog list, drag your work items into the designated sprint panel (such as **Iteration 1** or **Sprint 1**) located in the Planning pane on the right.
    
3. Persist in this process until you reach your team's planned capacity, which you can monitor via the capacity bar within that same Planning pane.
    

![Azure DevOps process settings for User Stories, showing custom fields like Automation ROI Score and Risk.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_fa8c16275fe04aa797838395f1e298ff_Screenshot-2026-06-05-180453.png?expiry=1789111711909&hmac=xKdVZF_ZiGS5YvlEGKKn1lxjGKN_0RBbtXnnFXP8VvM)

The image displays a screenshot of the Azure DevOps web interface inside the "Automation-Strategy" project, specifically showing the "Backlogs" dashboard for the "Automation-Strategy Team." On the far left, a vertical navigation sidebar displays main sections including Overview, Boards (with the "Backlogs" sub-option highlighted), Repos, Pipelines, Test Plans, and Project settings. In the central workspace, the interface displays a table under the "Backlog" tab containing three rows categorized under the "User Story" work item type. The titles of these stories, ordered from one to three, are "Clear user session cache," "Restart web-app-prod-blue server," and "Manually scale up database." On the right side of the screen, a vertical "Planning" side panel is open, instructing users to drag and drop work items to include them in a sprint. This planning panel outlines the "Automation-Strategy Team backlog" along with a list of target iterations, showing "Iteration 1" (with a "Planned Effort: 0" indicator and miniature representations of the selected stories being dragged or assigned), followed below by "Iteration 2" and "Iteration 3," both marked as having "No work scheduled yet," and a "+ New Sprint" button at the bottom right.

### **Step 7: Present a Data-Driven Strategy**

Using the sorted backlog as a visual aid is a powerful communication tool. In sprint planning or a retrospective meeting, you are no longer just presenting a list of tasks; you are presenting a data-driven strategy.

- **Visual Basis:** Share your screen showing the **Automation ROI Ranking** query created in Step 5. Ensure it is sorted by **Automation ROI Score** in descending order so stakeholders immediately see the highest value at the top.
    
- **Strategic Walkthrough:** Walk stakeholders through the top items your team has selected for the sprint (the items you dragged into the iteration in Step 6).
    
- **Explain the "Why":** Point directly to the data fields you created. For example:"We chose to automate the **server restart process** first because it has an ROI score of **5.0**. It currently generates **20 hours of manual toil** per month but only takes **4 hours** of engineering effort to automate. This move will immediately free up the team to focus on higher-level feature development."
    
- **Objective Buy-in:** This approach replaces subjective debates about what "feels" important with objective evidence, making it significantly easier to get executive buy-in for your automation roadmap.
    

![GitHub settings page for creating a branch protection rule for the "main" branch in the Assessifyx repository.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_3195976688124f14b5fc0088b7a6f004_Screenshot-2026-06-05-180813.png?expiry=1789111711909&hmac=-rE5R4FM62MGF-2ixlQmz-VvjHtNj1FLM1mHg4houIo)

The image displays a screenshot of the Azure DevOps web interface inside the "Automation-Strategy" project, specifically looking at a query view named "Automation ROI Ranking." On the far left, a vertical navigation sidebar highlights the active "Boards" section and its selected sub-option, "Queries," alongside other standard options like Repos, Pipelines, Test Plans, and Project settings. The central workspace shows the query's "Results" tab, displaying a table with two listed rows under truncated column headers like "Automat...", "Manual Toil...", and "Estimat...". Above the table, a status message indicates "2 of 3 work items, 1 selected," and action buttons like "Run query," "+ New," and "Save" are visible. On the right side of the screen, a vertical details panel is open for the selected item, labeled "USER STORY 4." The story is titled "Restart web-app-prod-blue serv" and shows buttons to "Save" and "Follow," with "No one selected" as the assignee and "0 Comments." Its metadata details that the State is "New," the Reason is "New," the Area is "Automation-Strategy," and the Iteration path is set to "Automation-Strategy\Iteration 1," with a note indicating it was updated by Shivaran Sasi 19 minutes ago. The panel finishes with collapsible sections for "Description" and "Acceptance Criteria" at the bottom right.

## **Summary**

By adding custom fields to Azure Boards and using a simple ROI formula, you have transformed your backlog from a basic to-do list into a sophisticated strategic planning tool. This process enables you to:

1. **Objectively rank** automation tasks based on real impact.
    
2. **Clearly communicate** priorities to stakeholders with data.
    
3. **Ensure engineering efforts** are always focused on delivering the greatest possible business value by eliminating the most expensive manual toil first.