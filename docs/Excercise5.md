# Lab 5 -  Personalized AI Routing 

Please use the following credentials to connect to Control Hub and configure Webex Contact Center:

| <!-- -->         | <!-- -->         |
| ---------------- | ---------------- |
| `Control Hub URL`            | <a href="https://admin.webex.com" target="_blank">https://admin.webex.com</a> |
| `Username`       | labuser**ID**@wx1.wbx.ai _(where **ID** is your assigned pod number; this ID will be provided by your proctor)_ |
| `Password`       | webexONE1! |

## Objective 

Traditional contact center routing relies on static, rigid skill assignments that often lead to misrouted contacts, higher average handle times (AHT), and administrative overhead. In this lab module, we will learn how to configure and leverage Webex Contact Center Personalized AI Routing. 

By completing this module, prticipants will be able to:

- **Understand the Core Value**: Learn how multi-dimensional AI dynamic matchmaking optimizes routing based on operational metrics like Handle Time.

- **Configure Global & Custom KPIs**: Review AI Routing at the organization level and Perform bulk Optimization Checks and initiate Evaluation Mode (Shadow Mode).

- **Master the Concept of Shadow Mode**: Understand how AI models predict handle-time reductions without altering live call flows.

- **Analyze Predictive Performance**: Navigate Webex Analyzer to review AI handle-time prediction models against baseline queue historical performance.ce.

Note on Lab Org Data: Because this lab environment does not feature active live call traffic, your configured queue will validate the setup and configuration flow. For reporting and analytics validation, an existing pre-populated queue (AI_Routing_Reference_Queue) has been provided to explore performance dashboards.


## Section 1 : Review Org-Level Enablement Setting 

- Log in to Control Hub as an Administrator.

- Navigate to Services > Contact Center.

- Under Desktop Experience, select AI Features.

- Open the AI Routing configuration page.

- Ensure that the Organization-level AI Routing switch is set to ON.

- Enabling this feature globally makes it available across the org, but does NOT automatically change routing logic.

### Section 2: Queue Optimization & Evaluation Mode Setup

- Select the Queues tab on the AI Routing page.

- Select the queue created in the previous Skill-Based Routing lab: WebexOne_SBR_Queue_[name]

- Click Run Optimization Check.

- In the KPI selection section, choose the stock metric Handle Time.

- Click Next, then click Run Optimization Check.

- Observe the validation results: The system evaluates whether the queue has sufficient agent density, interaction volume, and potential optimization gain for your chosen KPI.

- In our lab setup, the check will indicate low potential because the queue lacks sufficient agent density and call volume.

- Regardless of the low potential result, click Evaluation Mode.

- Confirm the prompt to start shadow execution.

- Now, In Evaluation Mode, incoming calls to this flow continue to follow standard routing logic. Meanwhile, the AI engine operates in the background to generate handle-time optimization predictions.

### Section 3: Reviewing a Completed Evaluation Queue

- Navigate back to the Queues tab under AI Features.

- In the search/filter bar, enter the queue name: WebexOne_SkillQueueFlow_Anuj.

- Click on WebexOne_SkillQueueFlow_Anuj to open its detailed AI Routing configuration pane.

- We should see the setting **Apply AI Routing** is enabled

- However, take a look at the screenshot below to see the steps required to apply AI Routing after Evaluation Mode completes

<<< Screen shot >>> 


- Review the queue status section before enabling the toggle switch to apply AI Routing:
	- KPI Selected: Handle Time
	- Potential: This queue has high potential for AI Routing.
	- Analyzer Link: Analyzer report which highlgihts how AI routing is behaving.

- The toggle  Blue indicates that Live AI Routing is now active for this queue, and real-time contacts will now be routed dynamically using the trained AI mode.

### Section 4: Reporting & Analytics Concepts

- In Webex Analyzer, Evaluation Mode generates a comparative dashboard that maps actual handle-time metrics against AI-predicted performance. Lets review the report for the Queue **WebexOne_SBR_Queue_[name]** Where you have enabled the evualation Mode. 

- Navigate to Services > Contact Center > Desktop Experience > AI Features.

- Select the Queues tab and locate the queue you configured **WebexOne_SBR_Queue_[name]**.

- Click on the queue to open its AI Routing details pane.

- Locate the Potential status section and click the link: View Analyzer report.

- This will automatically open Webex Analyzer in a new browser tab with the report filtered for your queue.

- Analyzer Link: Analyzer report which highlgihts the 5-day predictive handle-time reduction results before promoting the queue. 
