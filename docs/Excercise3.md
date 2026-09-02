# Lab 3 -  Flow Based SBR Configuration and Troubleshooting

Please use the following credentials to connect to Control Hub and configure Webex Contact Center:

| <!-- -->         | <!-- -->         |
| ---------------- | ---------------- |
| `Control Hub URL`            | <a href="https://admin.webex.com" target="_blank">https://admin.webex.com</a> |
| `Username`       | labuser**ID**@wx1.wbx.ai _(where **ID** is your assigned pod number; this ID will be provided by your proctor)_ |
| `Password`       | webexONE1! |

## Objective 

The objective of this lab is to introduce the core concepts of Skill-Based Routing (SBR) in Webex Contact Center and demonstrate how flow variables can dynamically dictate call routing to ensure contacts reach the most qualified resource.

This exercise is divided into three hands-on sections:

- **Section 1: Skill Profile Management**: Learn how to define skills (proficiency and boolean) and map them to agents using Skill Profiles.

- **Section 2: Dynamic Flow Configuration**: Explore advanced Flow Builder techniques to map call variables to skill requirements, driving targeted queue routing based on IVR selections.

- **Section 3: Diagnostics & Troubleshooting**: Identify common skill-mapping and variable misconfigurations, isolate root causes using Flow Debugger execution traces and Analyzer CSR reports, and apply corrective routing logic.

## Section 1 : Review Skills, Profiles, and Agent Entitlements 

**Verify Pre-created Skills:**

- Log into [Webex Control Hub](https://admin.webex.com) with the provided credentials.
  
-  From the left navigation sidebar, go to **Services** and select **Contact Center**.

- Navigate to **Customer Experience** > **Skill Management**.
  
- On the **Skills** tab, locate the following pre-created skills:
	* `WebexOne__Spanish_Fluency`
	* `WebexOne_VIP_Support`

- Review each skill's parameters against the baseline configuration table below:

| Skill Name | Expected Skill Type |
| --- | --- |
| `WebexOne__Spanish_Fluency` | Proficiency |
| `WebexOne_VIP_Support` | Boolean |

**Verify the Skill Profile:**

A **Skill Profile** acts as a container grouping multiple skills together so you can assign them to agents at once.

- In **Contact Center**, go to **User Management** > **Skill Profile**.

- Locate `WebexOne_Spanish_Specialist` in the **Skill Profiles** list.

- Select its row to open the configuration details panel.

- Under the **Active Skills** section, confirm that the settings match the table below:

| Skill Name | Skill Type | Skilled Value | Description |
| --- | --- | --- | --- |
| `WebexOne__Spanish_Fluency` | PROFICIENCY | 5 | Medium-high Spanish speaking capability |
| `WebexOne_VIP_Support` | BOOLEAN | True | Entitled to receive VIP-routed calls |

**Assign the Skill Profile to a User:**

- From the left navigation sidebar, select **User Management** > **Contact Center Users**.

- In the search bar, enter your lab user ID (`labuser<num>@wx1.wbx.ai`) and press **Enter**.

- Select the user row to open the user details side panel.

- Go to the **Agent Settings** section and select `WebexOne_Spanish_Specialist` from the **Skill profile** drop-down menu.

- Select **Save** to apply your changes.

**Validate the Assignment:**

- Re-open the user details panel for `labuser<num>@wx1.wbx.ai`.

- Under **Skill profile Settings**, confirm that the **Skill Profile** field explicitly displays `WebexOne_Spanish_Specialist`.

## Section 2 : Implement Skill-Based Routing (SBR) in Flow 

**Duplicate and Open the Flow Template**

* **Step 1:** An incoming call enters via the Entry Point and triggers the **Menu** node.
  
* **Step 2:** The customer hears an IVR greeting and is prompted to make a selection:
	* **Option 0:** Triggers a **Set Variable** node that assigns flow variable `Webexone_SPanish_FV = 5` and `Webexone_VIPCustomer_FV = True` to the call context.
	* **Option 1:** Triggers a **Set Variable** node that assigns flow variable `Webexone_SPanish_FV = 3` and `Webexone_VIPCustomer_FV = False` to the call context.

* **Step 3:** The call routes to the **Queue** node, where an appropriately skilled agent is selected based on the skill requirement conditions evaluated at the queue level.

**Identify Required Flow Updates**

- To test Skill-Based Routing (SBR) with this flow, you will make two specific updates:

	- **Update 1**: **Queue Assignment:** Assign the designated Contact Center queue to the **Queue** node.
	- **Update 2**: **Skill Configuration:** Define the required skill conditions in the **Skill Requirement** section of the **Queue** node.

**Update 1 : Create the Skill-Based Queue**

- For **Queue Assignement** lets create a skill based Queue and assign the contact center agent to the Queue, to configure 

- In the **Contact Center** navigation pane, select **Queues** under the **Customer Experience** section.

- Click the **Create Queue** button.

- Configure the general queue settings using the table below:

| Field Name | Value / Setting | Description |
| --- | --- | --- |
| **Queue Name** | `WebexOne_SBR_Queue_[name]`
| **Contact Direction** | **Inbound Queue** | 
| **Channel Type** | **Telephony** | 

- Scroll to **Contact Routing Settings**, enable **Skill-Based Routing**, and configure the following:
	* **Skill Assignment Type:** Select **Assign skills in flows**.
	* **Agent Assignment:** Select **Teams**.
	* **Routing Pattern:** Select **Longest Available**.

- Under **Call Distribution**, create a new group and add your designated team: `WebexOne_Team_[num]`.

- Scroll to **Advanced Settings** and verify the following parameters:
	* **Service Level Threshold:** `30` seconds
	* **Maximum Time in Queue:** `30` seconds
	* **Default Music in Queue:** `defaultmusic_on_hold.wav`


- Click **Save** to finalize and create the queue.

**Update 2: Update the Flow with Queue and Skill Requirements**

- Return to **Flows** section  and click your newly created flow row **Copy_webexOne_Skill_Flow_Template_<id>** to open it in **Flow Designer**.

- Click **Edit**  to modify the flow.

- Click the **Queue Contact** node on the canvas to open its properties panel.

- Under **Queue Settings**, select **Static Queue** and choose your newly created queue: `WebexOne_SBR_Queue_[name]`.

- Under the **Skill Requirements** section, click **Add Skill** and configure the two static skill requirements:

**Skill 1 Configuration**
	* **Skill Name:** Static > `WebexOne__Spanish_Fluency`
	* **Condition:** Static > `>=`
	* **Value:** Static > `5`

**Skill 2 Configuration**
	* **Skill Name:** Static > `WebexOne_VIP_Support`
	* **Value:** Static > `True`

**Validate and Publish the Flow**

- Click **Validate** in the bottom right menu bar and ensure no error messages appear.

- Click **Publish** to make the flow active for routing.

- To route incoming calls to your newly defined skill-based agents, the published flow must be mapped to an inbound entry point channel, to configure 

- In **Control Hub**, navigate to **Services** > **Contact Center**, then select **Channels** under the **Customer Experience** section.

- In the **Channels** list, search for the channel created in **Lab 1**.

- Select the channel to open its configuration settings.

- Under **Entry Point Settings**, navigate to the **Routing Flow** dropdown menu and select the published flow configured in the previous exercise (`webexOne_Skill_Flow_<Name>`).

- Click **Save** to apply the changes.

## Section 3 : Validating Dynamic SBR and Flow Logic

**Log in to Agent Desktop and Test Initial Call Flow**

- Open a browser tab and log in to the Webex Contact Center Agent Desktop:
	- URL: [https://desktop.wxcc-us1.cisco.com/](https://desktop.wxcc-us1.cisco.com/)
	- Username: Refer to your lab badge or contact your lab proctor.
	- Password: Refer to your lab badge or contact your lab proctor.

- On the login options screen, select Desktop as the telephony option, set the Team to WebexOne_Team_[num], and click Log In.

- Ensure the agent state is set to Available in the top-right corner of the Agent Desktop.

- From your mobile phone, dial the Dialed Number (DN) assigned to your entry point.

- When prompted by the IVR menu, select Option 1 to route to the logged-in agent.

**Analyze Unexpected Routing Behavior via Flow Debugger**

- Notice that the call is immediately offered to the agent. This behavior is incorrect based on our design.

- Expected Logic: 
	- Selecting Option 1 assigns variables tagging the call as a Non-VIP customer. The logged-in agent's profile requires VIP Support entitlement to handle incoming calls. Therefore, the call should not route to this agent.

- To isolate why the call routed incorrectly lets inspect the execution trace in Flow Debugger.

- In Flow Builder, click Debug from the bottom menu bar.

- Select the Interaction ID corresponding to your test call from the interaction list.

- Locate and expand the QueueContact node activity (this is where skill evaluation occurs).

- Under Activity Inputs > Skills Logs, inspect the JSON trace. The payload highlights the skills enforced on the call:
		- "skillName":"WebexOne__Spanish_Fluency","condition":"gte";"type":"proficiency","value":"5"
		- "skillName":"WebexOne_VIP_Support","condition":"eq";"type":"boolean","value":"True"

{"requirements":[{"weight":1,"radioSkillName":"Static","radioCondition":"Static","skillName":"WebexOne__Spanish_Fluency","condition":"gte","radioValue":"Static","skill":"80468aea-74b7-4afb-a2d3-9d9c4e4f1f92","type":"proficiency","value":"5"},{"radioSkillName":"Static","radioCondition":"Static","skillName":"WebexOne_VIP_Support","condition":"eq","radioValue":"Static","skill":"e6a7297f-254b-483c-bb91-c5e2da8dda63","type":"boolean","value":"True"}],"relaxationToggle":false,"relaxations":[],"removeSkillsOnBlindTransfer":false}

- Root Cause of this is although the Set Variable node set custom flow variables (Webexone_SPanish_FV = 3 and Webexone_VIPCustomer_FV = False), the QueueContact node is configured with Static skill values i.e. Spanish >= 5 and VIP = True). 

- As a result, the queue forced a static VIP requirement of True (level 5) and spanish 5, which matches the logged-in agent's profile.

**Update Queue Node to Dynamic Skill Assignment**

- To enforce the dynamic parameters set during IVR selection, convert the static skill requirements in the QueueContact node to dynamic variables.

- In Flow Builder, switch back to Design mode and toggle Edit mode to ON.

- Select the QueueContact node on the canvas to open its properties panel.

- Expand the Skill Requirements section.

- Update Skill 1 to evaluate dynamically:
	- **Skill Name**: Static > WebexOne__Spanish_Fluency
	- **Condition**: Static > >=
	- **Value**: Change from Static to Dynamic, then select the flow variable **Webexone_SPanish_FV**.

- Update Skill 2 to evaluate dynamically:
	- **Skill Name**: Static > WebexOne_VIP_Support
	- **Value**: Change from Static to Dynamic, then select the flow variable **Webexone_VIPCustomer_FV**.

- Click Validate in the bottom-right menu bar and ensure no configuration errors are returned.

- Click Publish to deploy the updated flow.

**Retest**

- Ensure your agent is set to Available on the Agent Desktop.

- **Test Call 1** : Option 1 - Non-VIP Flow

- Dial your assigned Dialed Number and press Option 1.

- **Result**: The call evaluates VIP = False. Because the agent requires VIP = True, the call will not route to the agent and will drop as expected.

- **Test Call 2** : Option 0 - VIP Flow

- Dial your assigned Dialed Number again and press Option 0.

- **Result**: The call evaluates Spanish >= 5 and VIP = True. The call will successfully route and present to the logged-in agent!

**Congratulations!!!** on completing this lab! 

You have successfully mastered the end-to-end setup of Skill-Based Routing in Webex Contact Center, from configuring skill profiles and dynamic flow variables to diagnosing real-world routing issues and tracking call journeys in Analyzer reports.
