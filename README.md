# ServiceNow Incident Management
## Client Scripts and UI Policies

> Naan Mudhalvan Mini Project: **Implement Client Script & UI Policy (Incident)**

## Project Overview

ServiceNow is a cloud-based platform widely used for IT Service Management (ITSM).
Incident Management is one of its core processes, and every disruption is captured
as an Incident record carrying fields such as Impact, Urgency, Priority,
Assignment Group, Assigned To and State.

This project improves data accuracy and consistency while creating and updating
Incident records. It uses **UI Policies** and **Client Scripts** on the Incident
table to control fields dynamically, automatically populate values, validate
required information before saving, and prevent incorrect updates. The controls
respond to the **Impact** field and affect Assignment Group, Assigned To, Urgency
and State.

## Problem Statement

In a default ServiceNow instance, users can leave important fields empty or change
them in ways that do not follow the business process. For High Impact incidents in
particular:

- High Impact incidents are saved without an Assignment Group or an Assigned To user.
- Urgency is set inconsistently, so the calculated Priority does not reflect the real severity.
- State is changed directly from the Incident list, bypassing the checks available on the form.

## Objectives

- Enforce mandatory fields automatically when Impact is High.
- Keep Urgency consistent with Impact by setting it automatically and locking it.
- Validate the record before it is saved, and show clear messages to the user.
- Restrict risky direct edits of State from the list view while keeping form updates available.
- Return the form to its normal behaviour when Impact is no longer High.

## Technologies Used

| Technology | Use in this project |
|---|---|
| ServiceNow platform | Hosts the Incident table, form and list |
| UI Policy and UI Policy Actions | No-code control of field behaviour (mandatory, read-only) |
| Client Scripts (JavaScript) | onChange, onSubmit and onCellEdit logic using the `g_form` API |

## ServiceNow Module

**Incident Management**: the Incident table, Incident form and Incident list.
Configuration is done under `System UI -> UI Policies` and `System UI -> Client Scripts`
and requires administrative or configuration access.

## Features Implemented

1. Makes Assignment Group mandatory when Impact is High.
2. Makes Urgency read-only when Impact is High.
3. Automatically sets Urgency to High when Impact is changed to High.
4. Prevents saving a High Impact Incident when Assigned To is empty.
5. Prevents users from changing State directly from the Incident list.
6. Allows State changes through the Incident form.
7. Reverses UI Policy restrictions when Impact is changed from High to another value.

## Client Scripts

All scripts are on the Incident table and are active. Source: [`ServiceNow/Client_Scripts/`](ServiceNow/Client_Scripts/)

| Script | Type | Field | Behaviour |
|---|---|---|---|
| [Auto set urgency for high impact](ServiceNow/Client_Scripts/onChange_Auto_Set_Urgency_For_High_Impact.js) | onChange | Impact | Sets Urgency to High and shows an info message when Impact becomes High |
| [Prevent save if Assigned To missing](ServiceNow/Client_Scripts/onSubmit_Prevent_Save_If_Assigned_To_Missing.js) | onSubmit | n/a | Blocks the save and shows an error on Assigned To when Impact is High and Assigned To is empty |
| [Prevent state change via list edit](ServiceNow/Client_Scripts/onCellEdit_Prevent_State_Change_Via_List_Edit.js) | onCellEdit | State | Shows an alert and cancels the edit when State is edited in the list view |

## UI Policies

Source: [`ServiceNow/UI_Policies/High_Impact_Control.md`](ServiceNow/UI_Policies/High_Impact_Control.md)

**High Impact Control** (Incident table, Active, Reverse if false = true).
Condition: Impact is 1 - High.

| UI Policy Action | Setting |
|---|---|
| Assignment group | Mandatory = true |
| Urgency | Read-only = true (Visible unchanged) |

## System Workflow

**When Impact = High**
- The UI Policy makes Assignment group mandatory and Urgency read-only.
- The onChange script sets Urgency to High and shows an info message.
- The onSubmit script blocks saving if Assigned To is empty.

**When Impact changes away from High**
- "Reverse if false" reverts the UI Policy changes (Assignment group is no longer mandatory; Urgency is editable again).
- The onSubmit script no longer blocks the save.

**In the Incident list**
- The onCellEdit script blocks direct edits of State; State must be changed from the Incident form.

```
User creates/updates incident
        -> Incident form and list (ServiceNow UI)
        -> UI Policies and Client Scripts (field control, automation, validation)
        -> Incident table (stored by the ServiceNow platform)
```

## Implementation

1. Create the **High Impact Control** UI Policy on Incident with condition *Impact is 1 - High* and *Reverse if false*.
2. Add the UI Policy Action making **Assignment group** mandatory.
3. Add the UI Policy Action making **Urgency** read-only.
4. Create the **onChange** Client Script on the Impact field.
5. Create the **onSubmit** Client Script for save validation.
6. Create the **onCellEdit** Client Script on the State field.
7. Test the configuration (see Testing).

Detailed steps and notes: [`ServiceNow/Configuration/Configuration_Notes.md`](ServiceNow/Configuration/Configuration_Notes.md)
and [`Documentation/Implementation_Documentation.pdf`](Documentation/Implementation_Documentation.pdf).

## Testing

Five test cases cover mandatory enforcement, successful save, the reverse condition,
list edit blocking and form-based State update. See [`Testing/Test_Cases.md`](Testing/Test_Cases.md)
and [`Documentation/Testing_Documentation.pdf`](Documentation/Testing_Documentation.pdf).

## Screenshots

### UI Policy
| | |
|---|---|
| ![UI Policy creation](Screenshots/01_UI_Policy_Creation_High_Impact_Control.jpg) | ![Condition and reverse if false](Screenshots/02_UI_Policy_Condition_Impact_High_Reverse_If_False.jpg) |
| UI Policy creation | Condition (Impact is 1 - High) and Reverse if false |
| ![Assignment group action](Screenshots/03_UI_Policy_Action_Assignment_Group_Mandatory.jpg) | ![Urgency action](Screenshots/04_UI_Policy_Action_Urgency_ReadOnly.png) |
| Assignment group set to Mandatory | Urgency set to Read-only |

### Client Scripts
| | |
|---|---|
| ![onChange script](Screenshots/05_Client_Script_onChange_Configuration.jpeg) | ![onCellEdit script](Screenshots/06_Client_Script_onCellEdit_Creation.jpeg) |
| onChange Client Script configuration | onCellEdit Client Script creation |

### Behaviour and Testing
| | |
|---|---|
| ![Urgency auto-set](Screenshots/07_Incident_Form_Urgency_AutoSet_ReadOnly.png) | ![Mandatory enforcement](Screenshots/08_Test_Mandatory_Enforcement_New_Incident.jpg) |
| Urgency auto-set to High and read-only | Mandatory enforcement on a new Incident |
| ![onSubmit validation](Screenshots/09_Test_OnSubmit_Validation_Assigned_To_Error.jpeg) | ![Reverse condition](Screenshots/10_Test_Reverse_Condition_Impact_Medium.jpeg) |
| onSubmit validation error on Assigned To | Reverse condition (Impact set to Medium) |
| ![List edit blocking](Screenshots/11_Test_List_Edit_Blocking_State_Alert.jpeg) | ![Incident list](Screenshots/12_Incident_List_View.png) |
| State list edit blocked by alert | Incident list view |
| ![Incident form State](Screenshots/13_Incident_Form_State_In_Progress.png) | |
| Incident form showing State | |

## Team Members

| Team Member | Contribution |
|---|---|
| Rithika R | ServiceNow Configuration |
| Roshan BR | ServiceNow Configuration |
| Rishika R | Client Script Configuration |
| Jeevasree SK | Client Script Configuration |
| Sheika Fathima K | Client Script Configuration |

Programme: BE CSE (AIML), IIIrd Year, 5th Semester.

## Team Contributions

Contributions are listed in the table above and in [`Team/Team_Members.md`](Team/Team_Members.md).
The project documents record contributions at this level only (ServiceNow Configuration
or Client Script Configuration); no further breakdown is recorded.

## Demo Video

The complete live ServiceNow project demonstration is available here:

[Watch Team Demo Video](PASTE_DEMO_LINK_HERE)

## Project Documentation

| Document | Description |
|---|---|
| [Project_Overview.pdf](Documentation/Project_Overview.pdf) | Introduction, problem statement, objectives, features and architecture |
| [Implementation_Documentation.pdf](Documentation/Implementation_Documentation.pdf) | Setup, configuration steps, functional documentation, known issues, future enhancements |
| [Testing_Documentation.pdf](Documentation/Testing_Documentation.pdf) | Test cases with expected results and screenshots |
| [Configuration_Notes.md](ServiceNow/Configuration/Configuration_Notes.md) | Configuration summary, how the components work together, implementation notes |

### Known Issues and Future Enhancements (from the project documentation)

- Requires access to a configured ServiceNow instance.
- Client Scripts are configured specifically for the Incident table.
- The State restriction applies to direct list editing; State changes through the form are allowed.
- The controls are client-side, so additional server-side validation may be required for integrations or other ways of creating records.

Possible future work listed by the team: server-side validation using Business Rules,
automatic Incident assignment, SLA integration, email notifications for High Impact
Incidents, automatic Priority calculation, dashboards and reports, Category/Subcategory
validation, and automated routing to Assignment Groups.
