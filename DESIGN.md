# Design Document

This document outlines the design choices, architecture, and technical decisions behind the Meeting Scheduling Agent.

***
## 1. Motivation & Problem Statement

Many people work remotely, leading to a high volume of meeting requests. Every time someone wants to set a meeting, they first need to check their schedule for availability. An AI agent that could listen to a user's messages and automatically schedule and update meetings for them would be a powerful tool. Furthermore, if the agent encountered scheduling conflicts, it could ask for the user's preference and resolve the issues accordingly.

***
## 2. Technical Choices

* **Automation Platform: n8n**
    * I chose the n8n platform because it is easy to implement and contains almost all the functions that an AI agent needs.

* **Language Model: Google Gemini**
    * For the LLM, I used Google Gemini, which not only provides a sufficient range of models but also offers free trials.

***
## 3. Core Design Decisions

### Prompt Engineering

For prompt engineering, I first described the main task of the AI agent and provided instructions for each internal step, while also strictly defining the agent's output format. This ensures the data returned by the LLM is structured and reliable.

### Evaluation Strategy

For the workflow output, I didn’t use quantitative evaluation scores. Since I only needed the LLM to understand messages and extract meeting information, I could verify the output by observing the final results, such as **Slack notifications** and **changes to the Google Calendar**.

***
## 4. Task Completion

The agent determines its task is complete when it reaches a defined **terminal state** in the workflow. This is not a single point but rather several possible outcomes:

* **Success ✅**: The task is complete when a meeting is created on Google Calendar and a **final confirmation message is successfully sent to Slack**. This is the "happy path" end state.

* **Conflict & Hand-off ❓**: The task for the main workflow is complete when it detects a scheduling conflict and **successfully sends the interactive message with "Re-write" and "Dismiss" buttons to Slack**. At this point, the main workflow's job is done, and it has handed off the responsibility to the user. The sub-workflow then has its own completion states.

* **No Action Needed 🤷**: The task is complete when the Gemini LLM processes the conversation and **determines that no meeting-related actions are required**. The workflow would end after a conditional logic (IF) node finds there's nothing to do.

* **User Dismissal 🛑**: In the sub-workflow, the task is complete when the user clicks "Dismiss" and the **workflow intentionally ends without taking any action**.

***
## 5. Workflow Architecture & Evolution

At first, the workflow was linear, but this design didn't work when processing multiple meetings from a single conversation. The split-out operation would execute the workflow for each item, but the Google Calendar conflict check could only output a single boolean result (`true` or `false`), which caused the workflow to lose the specific meeting information it was processing.

To solve this, after splitting out and getting separate meeting items, I fed these items into a **loop**. This allowed the workflow to check for conflicts on a per-item basis while also retrieving the correct meeting information for subsequent creation and update steps.

***
## 6. Future Improvements

If given another week, I would plan to add a sub-workflow for loading **long-term memory**.

Currently, the workflow can only process messages from a specific time period and cannot refer to historical conversations. I would use a scheduled trigger to load historical messages about meeting information each day, feeding this data into a **vector store** to serve as the agent's memory.

This would enable the agent to make wiser decisions. For example, if a new request is vague, the agent could use historical context to enrich the new meeting event with a more precise time or a detailed description.
