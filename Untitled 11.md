# Model the Patient Portal (30 min)

## Overview

This is the final, hands-on lab for this module, where you will bring together everything you have learned. You will move from theory and diagrams to a practical, tool-based analysis. Now, you will use the Microsoft Threat Modeling Tool (TMT) to perform a formal analysis of the TechHealth Patient Portal, generate a report, and derive actionable security requirements.

You will act as the Application Security Architect, taking a draft architecture diagram and using TMT to systematically uncover design flaws and propose concrete mitigation strategies. This is the core workflow of a modern security architect.

## By completing this activity, you will:

●      Create a Data Flow Diagram (DFD) in the Microsoft Threat Modeling Tool based on a reference architecture.

●      Define and apply trust boundaries to a DFD to enable automated threat analysis.

●      Generate and interpret a threat report from TMT based on the STRIDE framework.

●      Propose specific mitigation strategies for high-priority threats identified by the tool.

## Your lab files

On the VM desktop, open the **Files** folder (/home/coder/Desktop/Files).

1. **Access the lab environment:** This lab requires the Microsoft Threat Modeling Tool (TMT). As TMT is a Windows-only application, all learners (especially those on macOS or Linux) must use the provided Windows Virtual Machine where the tool is pre-installed.
2. **Visual Reference:** You have been provided with a file named Data_Flow_Diagram_Draft.vsdx. This Visio diagram contains the draft architecture you are tasked with modeling. You do not need Visio to open it; a simple image of the diagram is provided for you in Activity 1.

## Step 1 — Review the Reference Architecture

Before you open the tool, you must understand the system you are modeling. The team has provided you with a draft diagram of the **Patient Portal Authentication Flow**.

**Step 2 — Build the DFD in the Threat Modeling Tool**

Now, translate the reference diagram into a formal DFD using the TMT.

**Step 3 — Define Boundaries and Generate the Report**

A diagram is just a picture until you add trust boundaries. This step tells TMT which parts of the system are exposed and which are internal, enabling it to find threats.

**Step 4 — Analyze and Mitigate threats**

Your final task is to analyze the report and propose solutions. This turns your analysis into an actionable plan for the development team.

**Success checklist**

You have successfully completed this lab if your document includes:

●      You have a saved TMT file containing a DFD that accurately models the Patient Portal authentication flow.

●      The DFD includes a trust boundary correctly separating the internal application from external interactors.

●      You have viewed the generated Analysis Report within TMT.

●      You have a separate text file that lists three threats identified by the tool, each with a corresponding mitigation strategy.

●      The three required attendee roles listed with a clear and specific justification for each.

●      A clear understanding of the deliverables expected from the workshop.

●      The timeline established and defined in writing.