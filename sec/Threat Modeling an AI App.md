 Threat Modeling an AI App

0:12/10:06

## **Introduction**

Identifying a security gap in a data flow diagram is one thing. Knowing exactly where to insert a control to close it is what separates a reactive security review from a Secure-by-Design architecture. In this reading, you will explore the process of reviewing an AI application data flow diagram, identifying an unprotected Large Language Model (LLM) ingestion vector, and mandating the insertion of an Azure AI Content Safety inspection layer at the correct point in the request path.

Before you can identify what is missing, you need to understand what the current architecture shows and, critically, what it does not show.

## **Steps**

### **1. Open the Application Data Flow Diagram**

Begin with the data flow diagram (DFD) provided by the development team for the internal Copilot application. This diagram should show every component in the request path, from the user interface to the LLM endpoint, including any intermediate services such as an API gateway, an orchestration layer, or a Retrieval-Augmented Generation (RAG) pipeline.

![Data flow diagram showing User Interface connecting to an API Gateway, Orchestration Layer, and Azure OpenAI.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_70e146a1d5f54310ab1699b269b64184_image-5-.png?expiry=1787172453653&hmac=aJGkhlOWHtp8_oqUS6V3SmK2YjeGN0sRBjQrkWRKHHM)

A cloud architecture diagram created within a digital whiteboard tool titled "Enterprise Architecture." Five distinct component cards map out a sequential data flow using directional arrows. From left to right, a User Interface box connects to an API Gateway, which routes data to a central Orchestration Layer. This Orchestration Layer receives an upward-pointing connection from an External Document Store (RAG Ingestion) card below it. Finally, the Orchestration Layer connects to the terminal card on the right, labeled Azure OpenAI LLM Endpoint, completing the Retrieval-Augmented Generation workflow layout.

At this stage, you are not looking for code vulnerabilities. You are mapping trust boundaries. Ask yourself: At which points does data cross from one trust zone into another?

### **2. Annotate the Trust Boundaries on the Diagram**

Mark each point where data moves between components of different trust levels. In this architecture, the key trust boundaries are:

- The boundary between the **user interface and the API gateway**, where external, untrusted user input enters the system.
    
- The boundary between the **orchestration layer and the external document store**, where third-party or user-uploaded content enters the processing pipeline.
    
- The boundary between the **orchestration layer and the Azure OpenAI LLM endpoint**, where the assembled prompt is submitted for model invocation.
    

![Data flow diagram mapping three distinct red dashed trust boundaries across an RAG architecture workflow.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_277e023d18a341a780c563e980d1169f_image-6-.png?expiry=1787172453653&hmac=xULWOMjHMNwPm8WoAJcz1wVrq38T7aKaFkT7kNQoVtI)

Dashed lines cross the directional data arrows to mark three explicit threat modeling perimeters. Trust Boundary 1 (External User Input) is positioned between the User Interface and the API Gateway. Trust Boundary 2 (External Document Ingestion) cuts across the incoming path from the External Document Store to the Orchestration Layer. Finally, Trust Boundary 3 (LLM Invocation Point) separates the Orchestration Layer from the terminal Azure OpenAI LLM Endpoint, identifying critical validation points in the system.

These annotated boundaries are your focal points for threat modeling. Any data crossing these lines without inspection is a potential attack vector.

### **3. Trace the Prompt Assembly Path**

Follow the data flow from the user input through to the point of LLM invocation. In this architecture, the orchestration layer assembles the final prompt by combining the user's message with retrieved documents from the external document store. That assembled prompt is then sent directly to the Azure OpenAI endpoint.

![Data flow diagram highlighting the Orchestration Layer, combining user input and retrieved documents.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_ec5c6cea83ea4d6f99eeb0f922a4f448_image-7-.png?expiry=1787172453653&hmac=rVOJDHpmL7fILuXLMRS8Ov1vX9E_PxKZ7YEkCRncB_4)

The enterprise architecture diagram with an emphasis on the central processing step. The directional arrows connecting the components have been changed to bright orange. The Orchestration Layer component card features a distinct orange border highlight, which visually singles it out from the rest of the workflow. Directly below its purple node icon, a new orange subtitle has been added: "Combining user input + retrieved documents." This modification clearly illustrates the exact point in the Retrieval-Augmented Generation (RAG) system where data merging occurs before being sent to the final Azure OpenAI LLM Endpoint.

Note what is absent. There is no inspection or validation component between the orchestration layer and the LLM endpoint. The assembled prompt, which contains both user-supplied text and externally retrieved document content, is fed to the model without any screening.

### **4. Document the Vulnerability Finding**

Record this gap formally as a threat modeling finding. Your finding should capture three elements: the attack vector, the OWASP classification, and the risk.

- **Attack Vector:** Unvalidated user input and unscreened document content are assembled into a prompt and submitted directly to the LLM.
    
- **OWASP Classification:** LLM01 – Prompt Injection (both direct and indirect vectors are exposed).
    
- **Risk:** An attacker can embed malicious instructions in user input or in a document ingested via the RAG pipeline, causing the model to override its system instructions, exfiltrate data, or generate harmful outputs.
    

![Threat modeling worksheet matrix evaluating indirect prompt injection into a RAG pipeline as a High risk.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_4a62d254533241dcaef377d44b81fcde_GenAI_Module_LC4_Reading_img01.png?expiry=1787172453653&hmac=IxrGcHiw--RXwRZO0vNE6Gc2Usq33yqQ6m1iMHQfST8)

A table titled "Threat Modeling Worksheet – Prompt Injection into RAG Pipeline." The table has a dark blue header and three columns: Attack Vector, OWASP Classification, and Risk Rating. The first column describes an indirect prompt injection via external document ingestion with an entry point at the external document store. The second column classifies this under LLM01:2025–Prompt Injection, explaining that hidden instructions in external content cause the model to deviate from intended behavior. The final column marks the risk rating as High, citing high exploitation likelihood and direct impact on LLM behavior.

### **5. Identify the Correct Insertion Point**

The control must be placed before LLM invocation. This is a non-negotiable architectural requirement. Placing the inspection layer after the LLM has already processed a malicious prompt provides no protection. The two valid insertion points in this architecture are:

- At the API gateway, to screen raw user input before it reaches the orchestration layer.
    
- Within the orchestration layer, immediately before the assembled prompt is dispatched to the Azure OpenAI endpoint, to screen both user input and retrieved document content together.
    

For maximum coverage of both direct and indirect injection vectors, the recommended position is within the orchestration layer, screening the fully assembled prompt. This ensures that malicious content embedded in retrieved documents is also caught.

### **6. Update the Data Flow Diagram With the Mandated Control**

Insert the **Azure AI Content Safety Prompt Shields** API call as a required step in the orchestration layer, immediately before the LLM invocation call. The updated flow should make clear that no prompt reaches the model without first passing through the Prompt Shields evaluation.

![An architecture diagram showing Azure AI Content Safety filtering user prompts before the Azure OpenAI endpoint.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_be13820614664ccf86f8d48ceabd587b_image-9-.png?expiry=1787172453653&hmac=d6EWN4-9onSrjeMZIK45GWrgzEfjO9kGdGkpBpQK8EU)

A new card labeled Azure AI Content Safety – Prompt Shields with a blue shield icon is placed immediately after the Orchestration Layer. Two split paths emerge from this safety layer: an upper black arrow with a green checkmark leads to the standard Azure OpenAI LLM Endpoint, indicating safe traffic. Conversely, a red arrow indicates a path downward to a new red-bordered block marked with a white "X" icon, labeled "Injection Detected – Request Blocked," illustrating how prompt injections are caught and halted.

### **7. Add the Document Ingestion Inspection Point**

Because this architecture uses a RAG pipeline, you must also add a Prompt Shields document attack detection call at the point where external documents are ingested into the orchestration layer. This screens for indirect injection payloads embedded in retrieved content before they are combined with the user prompt.

![Architecture diagram showing Prompt Shields inspecting user prompts and RAG documents before LLM access.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_392bb9dce2014aa6aef08a02418ec80c_image-10-.png?expiry=1787172453653&hmac=XGYY2P5-CVqI7rHqggyLaek8BYhFzcBZEOfKCAbm9vs)

A workflow diagram explains a secure RAG architecture using Azure AI Content Safety Prompt Shields. User requests pass through an API Gateway and an orchestration layer before reaching the Azure OpenAI LLM endpoint. A second inspection point scans external documents from a RAG ingestion store for indirect prompt injection attacks before combining them with user prompts. Red warning paths show blocked requests when injection or document attacks are detected, while green arrows indicate approved flows through the system.

## **Key Considerations**

- **Prompt Shields must be synchronous, not asynchronous.** The inspection call must be completed and return a result before the prompt is forwarded. An asynchronous or fire-and-forget implementation does not protect because the prompt would reach the model before the screening result is available.
    
- **Log all blocked requests.** Every request that Prompt Shields flags and blocks should be logged with sufficient detail for incident investigation. This includes the raw input, the detection category, and the timestamp. These logs are your evidence trail for both security operations and compliance purposes.
    
- **Do not rely on Prompt Shields as the only control.** Prompt Shields is a mandatory layer, not a complete solution. Your architecture should also include output validation before LLM responses are passed to downstream systems, addressing the Improper Output Handling (LLM05:2025) risk identified in the OWASP Top 10 for LLM Applications.
    

## **Conclusion**

You have walked through the process of reviewing an AI application data flow diagram, annotating trust boundaries, identifying an unprotected LLM ingestion vector, and mandating the insertion of Azure AI Content Safety Prompt Shields at the correct point in the request path. The updated architecture now screens both direct user input and indirectly ingested document content before any prompt reaches the model, closing the two primary Prompt Injection attack vectors identified in the threat model.