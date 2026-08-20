# Gate the TechHealth App Pipeline (20 min)
## Overview
In the real world, theory and practice must meet. You've learned why shifting security left is critical and what the core technologies are. Now it's time to put it all together. As a DevSecOps architect, your primary role is to build the automated guardrails that enable teams to move fast and stay secure.
In this lab, you will act as the lead DevSecOps architect. Starting with a vulnerable application repository, you will implement a series of security gates. You will enable scanning for vulnerable dependencies (SCA), integrate static code analysis (SAST) directly into the CI pipeline, and finally, configure an automated branch protection rule to ensure no vulnerable code can be merged into the main branch.
By completing this activity, you will:
●	Enable and configure Dependabot to perform automated Software Composition Analysis (SCA).
●	Integrate a GitHub Actions workflow with CodeQL to perform Static Application Security Testing (SAST) on pull requests.
●	Configure a branch protection rule to enforce SAST scans as a required status check.
●	Verify that a security gate correctly blocks a pull request from being merged.
Your lab files



	
On the VM desktop, open the Files folder (/home/coder/Desktop/Files).
Step 1 — Enable Dependency Scanning (SCA) with Dependabot
The TechHealth API relies on several open-source Python libraries. Your first task is to ensure you have visibility into any known vulnerabilities within these third-party dependencies.

Step 2 — Integrate SAST with CodeQL into the CI Pipeline
Now that you're watching your dependencies, you need to analyze your own custom code. You will edit the existing CI workflow to add a CodeQL scanning job that runs every time a pull request is opened.
Step 3 — Create the Branch Protection Gate
The final and most critical step is to enforce your new security checks. You will now create a branch protection rule that makes the CodeQL scan mandatory for any pull request targeting the main branch.
Success checklist
You have successfully completed this lab if your document includes:
●	Enabled Dependabot for the repository.
●	Edited the ci.yml workflow to include a codeql-analysis job.
●	Created a branch protection rule for the main branch.
●	Confirmed that the CodeQL Analysis status check is required on a new pull request.
●	Verified that a failed CodeQL Analysis check blocks a pull request from being merged.
