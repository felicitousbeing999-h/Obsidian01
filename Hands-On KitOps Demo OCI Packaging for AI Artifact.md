

  

This guide documents the complete step-by-step hands-on workflow for packaging, versioning, inspecting, and pushing AI/ML artifacts using **KitOps** and **Jozu Hub**.

  

---

  

## 📋 Overview & Architecture

  

KitOps is an open-source tool (CNCF Sandbox candidate) that allows platform and DevOps engineers to treat AI models, prompt templates, code, and datasets as standard OCI (Open Container Initiative) artifacts.

  

```

+-------------------------------------------------------------------+

|                            KITFILE                                |

|  - Model Weights (GGUF / Bin)     - Datasets (CSV / TXT)          |

|  - Prompts & Templates            - Application / Runner Code     |

+-------------------------------------------------------------------+

                                  |

                                  v

                            [ kit pack ]

                                  |

                                  v

                   +-----------------------------+

                   |    Immutable OCI ModelKit   |

                   |   (SHA-256 Digest Address)  |

                   +-----------------------------+

                                  |

                                  v

                            [ kit push ]

                                  |

                                  v

            +-------------------------------------------+

            |  OCI Registry (Jozu Hub / GHCR / ECR)    |

            +-------------------------------------------+

```

  

---

  

## 🛠️ Step 1: Environment Setup & KitOps Installation

  

To set up KitOps on a Linux cloud VM without Homebrew:

  

```bash

# 1. Fetch the latest release tag from GitHub

LATEST_TAG=$(curl -s https://api.github.com/repos/kitops-ml/kitops/releases/latest | grep '"tag_name":' | sed -E 's/.*"([^"]+)".*/\1/')

  

# 2. Download the x86_64 Linux tarball

curl -L -o kitops-linux.tar.gz "https://github.com/kitops-ml/kitops/releases/download/${LATEST_TAG}/kitops-linux-x86_64.tar.gz"

  

# 3. Extract and move binary to system PATH

tar -xzvf kitops-linux.tar.gz

sudo mv kit /usr/local/bin/

rm kitops-linux.tar.gz

  

# 4. Verify installation

kit version

```

  

---

  

## 🔑 Step 2: Authentication with Registry

  

Log in to Jozu Hub using your registered account email address:

  

```bash

kit login jozu.ml

# Username: arorahardik0811@gmail.com

# Password: @hanumant999

```

  

> **Note:** Jozu Hub requires your full account email address as the CLI username, not the display username.

  

---

  

## 📂 Step 3: Project Structure & Artifact Preparation

  

Create an isolated directory and set up project files (model weights, adapters, code, and datasets):

  

```bash

# Create project workspace

mkdir ~/kit-demo && cd ~/kit-demo

  

# Create required project files

echo "# Llama 3 Fine-Tune Demo" > README.md

echo "dummy lora weights" > lora-adapter.gguf

echo "Sample training data line" > training-data.txt

```

  

---

  

## 📄 Step 4: Define the `Kitfile`

  

Create a `Kitfile` at the root of your project to declare how artifacts map into the OCI ModelKit package:

  

```yaml

manifestVersion: "1.0"

package:

  name: llama3 fine-tuned

  version: 3.0.0

  authors: [Jozu AI]

model:

  name: llama3-8B-instruct-q4_0

  path: jozu.ml/jozu/llama3-8b:8B-instruct-q4_0

  license: Apache 2.0

  description: Llama 3 8B instruct model

  parts:

    - path: ./lora-adapter.gguf

      type: lora-adapter

code:

  - path: ./README.md

datasets:

  - name: fine-tune-data

    path: ./training-data.txt

```

  

---

  

## 📦 Step 5: Pack, Inspect, and Push

  

Execute the core KitOps lifecycle commands to package and publish your ModelKit:

  

```bash

# 1. Pack the project into an OCI ModelKit

kit pack . -t jozu.ml/arorahardik0811/finetune:latest

  

# 2. List local ModelKit packages

kit list

  

# Output verification:

# REPOSITORY                         TAG     MAINTAINER  NAME              SIZE     DIGEST

# jozu.ml/arorahardik0811/finetune   latest  Jozu AI     llama3 fine-tuned 6.0 KiB  sha256:818b7f8e82ce0b423d8ee561a3ca114756e78868b9ef11d53277c304fc3a4897

  

# 3. Push ModelKit to remote OCI registry

kit push jozu.ml/arorahardik0811/finetune:latest

  

# Output verification:

# Pushing jozu.ml/arorahardik0811/finetune:latest

# Pushed sha256:818b7f8e82ce0b423d8ee561a3ca114756e78868b9ef11d53277c304fc3a4897

```

  

---

  

## 🔄 Step 6: Verification & Pull Test

  

To verify that your ModelKit is stored correctly on Jozu Hub, pull and unpack it into a clean test directory:

  

```bash

# Create a fresh directory

mkdir ~/test-pull && cd ~/test-pull

  

# Pull the published ModelKit from Jozu Hub

kit pull jozu.ml/arorahardik0811/finetune:latest

  

# Unpack and verify contents

kit unpack jozu.ml/arorahardik0811/finetune:latest

ls -la

```

  

---

  ![[Pasted image 20260729223041.png]]

## Key Takeaways for DevOps & MLOps

  

* **Immutable Artifacts:** Model weights, datasets, code, and prompts are linked together under a single SHA-256 digest.

* **Standard Infrastructure:** No dedicated ML storage needed; existing OCI container registries (Jozu Hub, GHCR, ECR, Docker Hub) manage versioning and security.

* **Declarative Configuration:** `Kitfile` brings Dockerfile-like simplicity to AI application delivery.