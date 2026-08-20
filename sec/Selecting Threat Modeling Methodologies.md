As the architect, you need to run a series of threat modeling workshops, but which methodology should you choose? The answer depends entirely on the question you are trying to answer.

## **STRIDE: The Security Workhorse**

**STRIDE** is the most common threat modeling methodology, developed by Microsoft. It is a mnemonic that stands for six categories of classic security threats. Its primary goal is to help you identify threats that could compromise the security of your application.

- **S**poofing: Illegitimately pretending to be someone or something else
    
- **T**ampering: Modifying data or code without authorization
    
- **R**epudiation: Claiming you did not perform an action when you did
    
- **I**nformation Disclosure: Exposing information to someone not authorized to see it
    
- **D**enial of Service: Making a system or resource unavailable to legitimate users
    
- **E**levation of Privilege: Gaining capabilities beyond what you are authorized for
    

![Infographic illustrating the STRIDE threat modeling framework with six colored columns defining each type of security risk.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_03dc0307541a4c90968cc2be1a4f2221_Threat_Modeling_SC9_Reading_Image3.png?expiry=1786701541857&hmac=cp15UzDS2vpsmj_WLV_MbBGbg1WiNUykpKVTx1mvy_Y)

This infographic matrix breaks down security categories under the top gray title bar "STRIDE THREAT MODEL (SIMPLIFIED)" over a light grid canvas. The structure is organized into six vertical columns, each corresponding to a letter in the acronym. The first column features a red block with the white letter "S" for "Spoofing," accompanied by a red masquerade mask icon and a bullet point reading "Pretending to be someone else." The second column displays an orange block with the letter "T" for "Tampering," paired with a document and pencil icon and a bullet point for "Modifying data or code." The third column contains a light green block with the letter "R" for "Repudiation," paired with green checkmark and red X circle icons, defining it as "Denying an action occurred." The fourth column shows a blue block with the letter "I" for "Information Disclosure," represented by an open yellow file folder containing documents and an unlocked padlock, noted as "Exposing sensitive data." The fifth column presents a purple block with the letter "D" for "Denial of Service," illustrated by a fractured, split server stack icon, indicating "System unavailable." The final column features a teal block with the letter "E" for "Elevation of Privilege," highlighted by a golden key icon and a descriptive bullet reading "Unauthorized permissions."

**When to Use STRIDE:** Use STRIDE as your default methodology for most software projects. It is excellent for identifying a broad range of common security threats in a system.

For HealthForward's telehealth platform, using STRIDE would help you answer questions like:

- **Spoofing:** Could an attacker pretend to be a real doctor to gain access to patient records?
    
- **Tampering:** Could a patient modify their own prescription details after the doctor has submitted them?
    
- **Information Disclosure:** What if a bug in the video consultation feature allowed one patient to see another patient's lab results?
    

This is the bread-and-butter of application security. STRIDE helps you ensure your application is robust against common attacks.

## **LINDDUN: The Privacy Specialist**

While STRIDE is excellent for security, it does not explicitly address privacy concerns. **LINDDUN** was created specifically to fill this gap. It is a methodology focused entirely on identifying privacy-related threats in software architecture. It, too, is a mnemonic for seven categories of privacy threats.

- **L**inkability: Can attackers link different pieces of personal data together?
    
- **I**dentifiability: Can users be personally identified from the data?
    
- **N**on-repudiation: Can a user plausibly deny an action they took? (Note: In privacy, this is a threat, whereas in security it's a property to enforce).
    
- **D**etectability: Can an attacker determine if a user is in the system?
    
- **D**ata Disclosure: Is personal information being revealed?
    
- **U**nawareness: Are users unaware of what data is being collected and why?
    
- **N**on-compliance: Does the system violate privacy regulations like HIPAA or GDPR?
    

**When to Use LINDDUN:** Use LINDDUN whenever you are handling sensitive personal information, especially **Personally Identifiable Information (PII)** or data covered by privacy laws. For a healthcare application, it is essential.

Applying LINDDUN to HealthForward's platform, you would ask different questions than with STRIDE:

- **Linkability:** Could an insurance company link a patient's "anonymous" browsing on our health blog with their actual medical diagnosis from the telehealth platform?
    
- **Identifiability:** Are we collecting a patient's IP address, location, and device ID when it is not strictly necessary, making it easy to identify them?
    
- **Unawareness:** Is it clear to the patient that their video consultation is being temporarily cached on a server, and have they consented to it?
    

LINDDUN does not primarily focus on availability threats such as Denial of Service. It focuses on privacy harms, including cases where a user's privacy is violated even if the system is functioning as designed from a security perspective.

## **PASTA: The Business Risk Champion**

**PASTA (Process for Attack Simulation and Threat Analysis)** is a much more comprehensive, risk-centric methodology. Unlike STRIDE or LINDDUN, which provide threat categories, PASTA is a seven-stage process designed to align security activities with business objectives. It focuses on identifying threats that pose a real risk to the business and helps justify security investments in terms of business impact.

The seven stages of PASTA involve defining business objectives, analyzing the attack surface, enumerating threats, and modeling those attacks to calculate their impact.

**When to Use PASTA:** Use PASTA when you need to connect technical threats to tangible business risk, especially in complex, high-value systems. It is more time-intensive than STRIDE or LINDDUN but provides a much richer, risk-based analysis. It's the right choice when you need to answer the "So what?" question for business leaders.

For HealthForward, using PASTA would lead to a very different kind of workshop:

- You would start by defining the business objectives, such as "Maintain patient trust by guaranteeing the confidentiality of PHI."
    
- Then, instead of just saying a **Denial of Service** is a threat, PASTA forces you to analyze its impact: "What is the **financial and reputational cost** to HealthForward if the prescription management service is down for four hours during peak times?"
    
- This approach helps you prioritize. A threat that has minimal business impact gets less attention than one that could cause significant financial loss or damage the company's reputation.
    

## **Choosing the Right Methodology**

Your choice of methodology focuses your workshop on what is most important for a given system.

![Flowchart choosing a threat modeling methodology based on goals.](https://d3c33hcgiwev3.cloudfront.net/imageAssetProxy.v1/_63958d1f07584caca0d4fc49e05fd3b1_Threat_Modeling_SC9_Reading_Image4.png?expiry=1786701541857&hmac=rfeZzbxfGRIftXTL09cPrMrpvBfY5aCSLqpD3-p0ngQ)

This decision-tree flowchart guides users in selecting a security framework, topped by the bold headline "CHOOSING YOUR THREAT MODELING METHODOLOGY" on a light gray canvas. The logic sequence originates at a central, blue diamond decision block labeled "GOAL?" which branches out into three distinct downward paths. The leftmost path uses a blue directional line leading to a white rectangular box reading "Identify technical threats," which channels down via an open arrow into a rounded, blue horizontal badge labeled "STRIDE" featuring a blue masquerade mask icon. The middle path uses a green directional line leading to a white rectangle labeled "Identify privacy threats," which flows down into a rounded, green horizontal badge labeled "LINDDUN" displaying a green padlock and profile user icon. The rightmost path follows an orange directional line down to a white rectangle labeled "Connect to business risk," which drops down into a rounded, orange horizontal badge labeled "PASTA" containing a fractured shield and key icon. The entire analytical infographic finishes with a centered footer line stating, "BUILDING RESILIENT SECURITY AND PRIVACY."

|**Methodology**|**Primary Focus**|**Best For**|**Example Question it Answers**|
|---|---|---|---|
|**STRIDE**|**Security:** Protecting the system from classic attacks|General-purpose software, APIs, and network services where core security is the goal|"Could an attacker tamper with this data?"|
|**LINDDUN**|**Privacy:** Protecting the personal data of the system's users|Applications handling PII, PHI, or data subject to GDPR, HIPAA, or other privacy regulations|"Are we collecting more user data than is necessary?"|
|**PASTA**|**Business Risk:** Connecting technical threats to business impact and objectives|Complex, mission-critical systems where security decisions must be justified in terms of risk|"What is the business impact if this asset is compromised?"|

## **Conclusion**

A threat modeling workshop is a guided conversation, and your chosen methodology provides the script. There is no single "best" methodology—only the right one for your goal. By selecting **STRIDE** for general security, **LINDDUN** for privacy-sensitive applications, or **PASTA** for risk-centric analysis, you ensure that your workshop is a focused, efficient, and effective use of everyone's time. As an architect, knowing how to choose and apply the right framework is a critical skill that transforms threat modeling from a theoretical exercise into a practical tool for building safer, more resilient systems.