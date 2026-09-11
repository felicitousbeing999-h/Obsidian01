%-------------------------
% Resume in Latex
% Jake Gutierrez style (Guaranteed 1-Page Layout)
%------------------------

\documentclass[letterpaper,9.5pt]{article}

\usepackage{latexsym}
\usepackage[empty]{fullpage}
\usepackage{titlesec}
\usepackage{marvosym}
\usepackage[usenames,dvipsnames]{color}
\usepackage{verbatim}
\usepackage{enumitem}
\usepackage[hidelinks]{hyperref}
\usepackage{fancyhdr}
\usepackage[english]{babel}
\usepackage{tabularx}
\input{glyphtounicode}

\pagestyle{fancy}
\fancyhf{}
\fancyfoot{}
\renewcommand{\headrulewidth}{0pt}
\renewcommand{\footrulewidth}{0pt}

% Tightened margins for single-page fit
\addtolength{\oddsidemargin}{-0.6in}
\addtolength{\evensidemargin}{-0.6in}
\addtolength{\textwidth}{1.2in}
\addtolength{\topmargin}{-.6in}
\addtolength{\textheight}{1.2in}

\urlstyle{same}
\raggedbottom
\raggedright
\setlength{\tabcolsep}{0in}

% Sections formatting
\titleformat{\section}{
  \vspace{-10pt}\scshape\raggedright\small\bfseries
}{}{0em}{}[\color{black}\titlerule \vspace{-3pt}]

\pdfgentounicode=1

%-------------------------
% Custom commands
\newcommand{\resumeItem}[1]{
  \item\small{
    {#1 \vspace{-2pt}}
  }
}

\newcommand{\resumeSubheading}[4]{
  \vspace{-3pt}\item
    \begin{tabular*}{0.98\textwidth}[t]{@{}p{0.82\textwidth}@{\extracolsep{\fill}}r}
      \textbf{#1} & #2 \\
      \textit{\small#3} & \textit{\small #4} \\
    \end{tabular*}\vspace{-5pt}
}

\newcommand{\resumeProjectHeading}[2]{
    \vspace{-3pt}\item
    \begin{tabular*}{0.98\textwidth}[t]{@{}p{0.88\textwidth}@{\extracolsep{\fill}}r}
      \small#1 & #2 \\
    \end{tabular*}\vspace{-5pt}
}

\renewcommand\labelitemii{$\vcenter{\hbox{\tiny$\bullet$}}$}

\newcommand{\resumeSubHeadingListStart}{\begin{itemize}[leftmargin=0.12in, label={}]}
\newcommand{\resumeSubHeadingListEnd}{\end{itemize}\vspace{-6pt}}
\newcommand{\resumeItemListStart}{\begin{itemize}[leftmargin=0.12in]}
\newcommand{\resumeItemListEnd}{\end{itemize}\vspace{-4pt}}

%-------------------------------------------
%%%%%%  RESUME STARTS HERE  %%%%%%%%%%%%%%%%%%%%%%%%%%%%

\begin{document}

%----------HEADING----------
\begin{center}
    \textbf{\Large \scshape Hardik Arora} \\ \vspace{1pt}
    \small Google Cloud Engineer \;|\; Platform Engineering \;|\; Kubernetes \;|\; DevSecOps \\ \vspace{1pt}
    \href{mailto:arorahardik0811@gmail.com}{arorahardik0811@gmail.com} $|$ +91 6397957661 $|$ Uttar Pradesh, India \\
    \href{https://github.com/barbaria888}{GitHub} $|$ 
    \href{https://linkedin.com/in/hardik0811arora}{LinkedIn} $|$ 
    \href{https://hardik-arora.hashnode.dev}{Blog} $|$ 
    \href{https://www.youtube.com/@hardikarora999}{YouTube}
\end{center}
\vspace{-12pt}

%-----------SUMMARY-----------
\section{Professional Summary}
\small{Curious, pragmatic Infrastructure Engineer with a strong foundation in GCP, container orchestration, and DevSecOps. Dedicated to simplifying developer workflows, implementing clean GitOps practices, and building secure, cost-conscious platforms. Keen to roll up my sleeves on day-to-day cluster reliability, internal tooling, and pragmatic AI operational aids alongside high-trust teams. Engages actively in open source and community learning, reaching \textbf{6,700+ cloud professionals} (including 600+ Google professsionals).}
%-----------EXPERIENCE-----------
\section{Experience}
  \resumeSubHeadingListStart
    \resumeSubheading
      {DevOps Engineering Intern}{Present}
      {InterviewCafe}{Noida, India}
      \resumeItemListStart
        \resumeItem{Orchestrated CI/CD pipelines for an EdTech platform, automating builds, artifact management, and continuous delivery to improve release reliability.}
        \resumeItem{Embedded Shift-Left security (Trivy + OWASP ZAP) into the CI/CD pipeline, automatically rejecting vulnerable artifacts and blocking \textbf{95\% of critical vulnerabilities} before staging.}
      \resumeItemListEnd
  \resumeSubHeadingListEnd

%-----------PROJECTS-----------
\section{Projects}
    \resumeSubHeadingListStart
      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/FrugalZeus}{FrugalZeus: Internal Developer Platform Reference}} $|$ \emph{Kubernetes, Argo CD, OpenCost, OpenTelemetry}}{2026}
          \resumeItemListStart
            \resumeItem{Architected an Internal Developer Platform (IDP) featuring an end-to-end GitOps delivery engine, multi-tenant guardrails (NetworkPolicies, ResourceQuotas), and real-time FinOps cost allocation via OpenCost.for bare metal(k3s,kubeadm,kind) and cloud hosted  kubernetes distributions like GKE,AKS,EKS.} 
          \resumeItemListEnd

      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/GKE-Managed-CloudServiceMesh-BookInfo}{Managed Cloud Service Mesh on GKE}} $|$ \emph{GKE, Cloud Service Mesh, Envoy, mTLS, Gateway API}}{2026}
          \resumeItemListStart
            \resumeItem{Deployed a production-grade Google Cloud Service Mesh on GKE using Envoy sidecars to enforce strict mutual TLS (mTLS), zero-trust pod communication, and advanced traffic routing policies.}
          \resumeItemListEnd

      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/FleetOpsGKE}{FleetOpsGKE Enterprise Platform}} $|$ \emph{GKE Fleets, Anthos Service Mesh, Config Sync, Policy Controller}}{2026}
          \resumeItemListStart
            \resumeItem{Built a multi-cluster GKE platform using GKE Fleets, Anthos Service Mesh, Config Sync, and Policy Controller. Centralized workload management, service connectivity, policy enforcement, and GitOps configuration across clusters.}
          \resumeItemListEnd

      
      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/LogOps-Magic-on-GKE}{LogOps Magic \& Distributed Tracing}} $|$ \emph{Cloud Logging, OpenTelemetry, Jaeger, BigQuery}}{2026}
          \resumeItemListStart
            \resumeItem{Implemented production observability pipelines on GKE with Cloud Logging, Log Analytics, BigQuery, OpenTelemetry, and Jaeger. Delivered centralized logging and distributed tracing.}
          \resumeItemListEnd

      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/Secure-GKE-Private-Clusters}{GKE Zero-Trust Network Architecture}} $|$ \emph{Private GKE, VPC, Master Authorized Networks}}{2026}
          \resumeItemListStart
            \resumeItem{Designed private GKE clusters with Private Nodes, Master Authorized Networks, custom VPC, and workload isolation to enforce Zero Trust principles.}
          \resumeItemListEnd

      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/KubeOps-AI}{KubeOps-AI Troubleshooting Platform}} $|$ \emph{Prometheus Alertmanager, RunWhen CodeBundles}}{2026}
          \resumeItemListStart
            \resumeItem{Created an autonomous Kubernetes troubleshooting system that ingests Prometheus Alertmanager events via webhooks. Integrated RunWhen CodeBundles after design discussions with RunWhen CEO.}
          \resumeItemListEnd

      \resumeProjectHeading
          {\textbf{\href{https://github.com/barbaria888/SupplyChain-Guardian-AI-Github_Action}{Supply Chain Guardian AI}} $|$ \emph{Trivy, KinD, GitHub Actions, LLMs}}{2026}
          \resumeItemListStart
            \resumeItem{Built an Agentic DevSecOps GitHub Action combining Trivy, ephemeral KinD clusters, and OpenAI-compatible LLMs for automated vulnerability detection and remediation recommendations.}
          \resumeItemListEnd
    \resumeSubHeadingListEnd

%-----------OPEN SOURCE & COMMUNITY-----------
\section{Open Source \& Community Impact}
  \resumeSubHeadingListStart
    \resumeItem{\textbf{DORA Community Speaker}: Invited presenter at DORA Community Days on local LLM inference, AI platform engineering workflows, and closed-loop remediation strategies.}
    \resumeItem{\textbf{Industry Collaboration}: Invited by former Google Kubernetes leadership to evaluate production-grade Agent Skills for the RunWhen platform. Built a community of \textbf{6,700+ cloud professionals} (815+ from Google).}
    \resumeItem{\textbf{Open Source}: Contributed AWS Security Hub integration (Macie, Inspector, GuardDuty, IAM Access Analyzer) to Arvo-AI via webhooks and Celery. Fixed visibility issues in InfraScan merged into release v1.0.5.}
  \resumeSubHeadingListEnd

%-----------CERTIFICATIONS-----------
\section{Certifications}
 \begin{itemize}[leftmargin=0.12in, label={}]
    \small{\item{
     \textbf{Google Cloud}: 
     \href{https://www.coursera.org/account/accomplishments/specialization/L0GXBNTINRYQ}{PCA Specialisation} $|$ 
     \href{https://www.coursera.org/account/accomplishments/specialization/178LVY9H9IYA}{Cloud DevOps Specialisation} $|$ 
     \href{https://www.coursera.org/account/accomplishments/specialization/6FN6EZ5EUXMY}{ACE Specialisation} $|$ 
     \href{https://www.coursera.org/account/accomplishments/specialization/AOCKJUG0OU9E}{Architecting with GKE} $|$ 
     \href{https://www.coursera.org/account/accomplishments/specialization/N3J5JW2UOUKH}{Cloud AI Infrastructure} \\[1pt]

     \textbf{Terraform \& AI Systems}: 
     \href{https://www.coursera.org/account/accomplishments/verify/XX9TCG1VNGQG}{Terraform for GCP} $|$ 
     \href{https://www.skills.google/public_profiles/e084697c-2e8b-45ca-a26c-766951118d6a/badges/24558271}{Model Armor: Securing AI} $|$ 
     \href{https://www.skills.google/public_profiles/e084697c-2e8b-45ca-a26c-766951118d6a/badges/24529053}{Build \& Deploy Agents in Prod} $|$ 
     \href{https://www.skills.google/public_profiles/e084697c-2e8b-45ca-a26c-766951118d6a/badges/23200993}{Agent Fundamentals} \\[1pt]

     \textbf{Security \& Observability}: 
     \href{https://www.coursera.org/account/accomplishments/verify/GBDYMXXDHV7H}{Labs for Security Engineers} $|$ 
     \href{https://www.virtualbadge.io/certificate-validator?credential=9f3e161a-d0f1-46c2-be41-ecb438c62792}{Vulnerability Management} $|$ 
     \href{https://www.coursera.org/account/accomplishments/verify/ATYZY7JOHAUP}{Logging \& Monitoring (GCP)} $|$ 
     \href{https://www.coursera.org/account/accomplishments/verify/NDC7GAL81HKZ}{Observability for DevOps (IBM)} \\[1pt]

     \textbf{Platform \& CI/CD}: 
     \href{https://www.virtualbadge.io/certificate-validator?credential=432d08fc-3c6f-4be6-abff-a202d7d1d8eb}{Intro to Platform Engineering} $|$ 
     \href{https://www.coursera.org/account/accomplishments/verify/NKJAD1MZW2CN}{CI/CD (IBM)} $|$ 
     \href{https://www.credential.net/caf90f89-5cdf-4de6-ae1e-8afc6ba23105#acc.057zAQwr}{CD \& GitOps (Argo CD)} $|$ 
     \href{https://www.coursera.org/account/accomplishments/verify/CGZO7L33T1AY}{Docker, Kubernetes \& OpenShift}
    }}
 \end{itemize}

%-----------TECHNICAL SKILLS-----------
\section{Technical Skills}
 \begin{itemize}[leftmargin=0.12in, label={}]
    \small{\item{
     \textbf{Cloud Platforms}: Google Cloud Platform, GKE (Autopilot, Fleets), Kubernetes, Compute Engine, Cloud Run, Cloud Storage, VPC, Networking \\
     \textbf{IaC \& CI/CD}: Terraform, Argo CD, Cloud Build, Cloud Deploy, Artifact Registry, Skaffold, GitHub Actions, Bash \\
     \textbf{Security \& Service Mesh}: Cloud Service Mesh, Envoy, mTLS, IAM, Workload Identity, Pod Security Admission, Trivy, OWASP ZAP \\
     \textbf{Observability \& FinOps}: OpenCost, OpenTelemetry, Cloud Logging, Log Analytics, BigQuery, Cloud SQL, Jaeger, Grafana
    }}
 \end{itemize}

%-----------EDUCATION-----------
\section{Education}
  \resumeSubHeadingListStart
    \resumeSubheading
      {GLA University}{Uttar Pradesh, India}
      {Bachelor of Technology (B.Tech.), Computer Science}{}
  \resumeSubHeadingListEnd

\end{document}