# Lab 4 - Blind Transfer Skill Requirements and Lifecycle Reporting 


Please use the following credentials to connect to Control Hub and configure Webex Contact Center:

| <!-- -->         | <!-- -->         |
| ---------------- | ---------------- |
| `Control Hub URL`            | <a href="https://admin.webex.com" target="_blank">https://admin.webex.com</a> |
| `Username`       | labuser**ID**@wx1.wbx.ai _(where **ID** is your assigned pod number; this ID will be provided by your proctor)_ |
| `Password`       | webexONE1! |


!!! info
	This task showcases available WebRTC call data and statistics using built-in browser tools and simulates missing audio during the call.

## Objective 

In this lab, you will explore how skill requirements attached to an inbound call behave when an agent performs a blind transfer directly to a skill-based queue.

**Current Call Flow Scenario:**
A customer calls in and selects **Option 0**. The call is routed to a logged-in agent matching the required skills. The agent then attempts a blind transfer to a pre-defined queue from the Agent Desktop. However, instead of connecting to an available agent in that queue, the call fails.

**Task:**
1. **Troubleshoot the Failure:** Identify why the transferred call drops or fails to route.
2. **Apply Corrective Logic:** Troubleshoot the issue Update the contact flow to handle transferred skill attributes properly.
3. **Verify Call Completion:** Perform a successful blind transfer to an available agent.
4. **Build a Journey Report:** Create a custom Analyzer report that maps the complete end-to-end call lifecycle across Labs 2 and 3, displaying the exact skills associated with each call leg.

## Section 1: Review the Transfer Queue Configuration

- Before attempting the transfer lets inspect the pre-configured target queue to verify its routing settings and assigned team.

- In the **Contact Center** navigation pane, select **Queues** under the **Customer Experience** section.

- In the search bar, search for the queue named **WebexOne_BlindTransfer_SGtoFlowQueue**.

- Click the queue row or edit icon to inspect the **Contact Routing Settings**, ensuring the following options are configured:
	* **Skill-Based Routing:** Toggle enabled (*Use skills-based routing for this queue*).
	* **Skill Assignment Type:** Select **Assign skills in flows**.
	* **Agent Assignment:** Select **Teams**.
	* **Routing Pattern:** Select **Longest Available**.

- Scroll to **Call Distribution**, click the **Edit (pencil)** icon under *Actions*, search for the team `WebexOne_Team_31`, and ensure it is added to the queue.

**Note:** The team `WebexOne_Team_31` is assigned to the user `labuser31@wx1.wbx.ai`, who is currently logged in and set to **Available** on the proctor's workstation. 

- Your goal in this exercise is to successfully transfer a call to the proctor.

## Section 2: Initiate Call and Experience the Transfer Failure

- Log into your **Agent Desktop** and ensure your state is set to **Available**.

- From your mobile device, dial your assigned entry point Dialed Number (DN) and press **Option 0** to route the call to your agent session.

- Accept the incoming call on your Agent Desktop.

- On the call control card, verify that the active flow variables and skill requirements are displayed:
	* `Language` : **5**
	* `VIP` : **True**

- On the Agent Desktop action bar, click **Transfer**, select **Queue**, and search for `WebexOne_BlindTransfer_SGtoFlowQueue`.

- Select the queue name and click **Transfer** to execute a blind transfer.

- Observe the transfer behavior

- The call drops immediately instead of connecting to the available proctor agent **`labuser31@wx1.wbx.ai`**.

## Section 3: Troubleshooting and Correcting the Blind Transfer Failure

To troubleshoot a blind transfer failure, inspect the **Flow Debugger** interaction trace alongside the **Analyzer Customer Session Record (CSR)** report to identify the root cause.

**Step 1: Retrieve the Interaction ID from Flow Debugger**

- In **Flow Builder**, click **Debug** in the bottom action bar.

- Locate the failed call in the interaction list and note down its **Interaction ID**.

**Step 2: Open and Configure the Analyzer CSR Report**

- In **Control Hub**, navigate to **Services** > **Contact Center** and select **Overview**.

- Under the **Quick Links** section on the right, select **Analyzer**.

- Click **Visualization**, then double-click your personal user folder: `WebexOne_Report_User[num]`.

- Open the pre-configured report named **CSR Report on SBR** (a pre-built copy of the stock Customer Session Record report).
> **Note:** This report is  pre-populated with several Row Segments to simplify setup:
	> * `EntryPointName`
	> * `ContactSessionID`
	> * `FirstQueueName`
	> * `Required Skills`
	> * `Matched Skills`
	> * `First Agent Name`
	> * `Queue Name`
	> * `Final Agent Name`

- Click **Edit** to modify the report layout.

- Ensure the report time range filter is set to **Today**.

- Confirm that all required fields listed above are present under **Row Segments**.

- Save the visualization and click **Preview**.

- Search for the **Interaction ID** you retrieved in Step 1.

- Review the call record row:
	* `FirstQueueName`: The initial queue where the call landed.
	* `Required Skills`: Skills required for initial call presentation to the agent.
	* `Matched Skills`: The agent skills matched during routing.
	* `First Agent Name`: Name of the agent who answered the initial call.
	* `Transferred Queue Name`: The queue to which the agent transferred the call.
	* `Last Agent Name`: The final agent who received the transferred call.

- **Observation:** The `Transferred Queue Name` displays the correct queue, but `Last Agent Name` still displays the original agent, indicating the transfer was unsuccessful.

## Section 4: Add Termination Fields to Isolate Root Cause

To determine why the transfer failed and which component terminated the call, add specific termination fields to the report:

- Return to the **Visualization** edit page for your report.

- Under **Row Segments**, search for **Terminated By** in the *New Segment* section.

- Drag and drop **Terminated By** to the bottom of the **Row Segments** list.

- Repeat the same process for **Termination Type** and **Abandoned Type**.

- Save the report and click **Preview**.

- Search for your **Interaction ID** again and analyze the updated fields:
	* **Terminated By (`System`):** The call was ended programmatically by the Webex Contact Center platform, rather than by the customer or agent.
	* **Termination Type (`abandoned`):** The system classified the call leg as dropped before connecting to an agent following the transfer request.
	* **Abandoned Type (`Queue`):** The call failed while waiting in the destination queue (`WebexOne_BlindTransfer_SGtoFlowQueue`).

- The field results confirm that the blind transfer reached the destination queue, but no matching agent was found. 

- The system held the call in queue until the timeout threshold was reached, resulting in a system-initiated queue abandonment.

- The underlying cause is a **Skill Carryover Conflict**. By default, when an agent performs a blind transfer, the system carries over the initial call's required skills (`Spanish >= 5`, `VIP = True`). Because the receiving agent in `WebexOne_Team_31` does not hold those exact matching attributes, no eligible agent is found.

## Section 5: Identify Root Cause via Flow Debugger

- Return to **Flow Debugger** and inspect the **QueueContact** activity log for this call.

- Locate the **Skills Logs** JSON trace to confirm that skill removal is disabled:

```json
{
  "requirements": [
    {
      "weight": 1,
      "radioSkillName": "Static",
      "radioCondition": "Static",
      "skillName": "WebexOne__Spanish_Fluency",
      "condition": "gte",
      "radioValue": "Dynamic",
      "skill": "80468aea-74b7-4afb-a2d3-9d9c4e4f1f92",
      "type": "proficiency",
      "value": "5"
    },
    {
      "radioSkillName": "Static",
      "radioCondition": "Static",
      "skillName": "WebexOne_VIP_Support",
      "condition": "eq",
      "radioValue": "Dynamic",
      "skill": "e6a7297f-254b-483c-bb91-c5e2da8dda63",
      "type": "boolean",
      "value": "true"
    }
  ],
  "relaxationToggle": false,
  "relaxations": [],
  "removeSkillsOnBlindTransfer": false
}

```

- Notice that `"removeSkillsOnBlindTransfer"` is set to `false`.

- Now, Update the Flow and Enable Skill Removal

- Open your flow in **Flow Designer** and click **Edit** (or toggle **Edit** to **ON**).

- Click the **QueueContact** node on the canvas to open its properties panel.

- Under **Skill Removal**, toggle **Remove skills on blind transfer** to **Enabled** (`True`).

- Click **Validate**, then click **Publish** to deploy the updated flow.

## Section 6: Retest and Validate

- Dial the entry point number, press **Option 0** to reach your Agent Desktop, and accept the call.

- From the Agent Desktop, perform a blind transfer to `WebexOne_BlindTransfer_SGtoFlowQueue`.

- Verify that the call successfully routes and rings the proctor's phone (`labuser31@wx1.wbx.ai`).

- Re-inspect the **QueueContact** trace in **Flow Debugger**; confirm that `"removeSkillsOnBlindTransfer"` is now set to `true`.

- Refresh your Analyzer report and search for the new **Interaction ID**. Verify that:
	* `Transferred Queue Name` displays the target queue.
	* `Last Agent Name` updates to show the proctor agent who received the transfer.

**Congratulations!!** You have successfully completed this lab.

