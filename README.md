# F5 XC as AI Gateway
This repo/deployment demonstrates:
- Running **LiteLLM Proxy** as a containerized app on **Virtual Kubernetes (vk8s)**
- Routing/load-balancing across multiple LLM backends and models
- Publishing and securing the service using **F5 Distributed Cloud (XC)** for global load balancing, WAF, API security, and DDoS protection
- A simple **Sentence paraphraser AI app** consuming the gateway

# The APP
```mermaid
flowchart TB
    User["👤 End User / Client App"]

    subgraph F5["F5 Distributed Cloud (XC)"]
        direction TB
        LB["Global Load Balancer"]
        WAF["WAF / API Security"]
        BOT["Bot Defense / Rate Limiting"]
        LB --> WAF --> BOT
    end

    subgraph K8S["vK8s Cluster (p-kuligowski namespace)"]
        direction TB
        SVC_FE["Service: frontend"]
        FE["Deployment: frontend-deployment\n(Web UI / API consumer)"]
        SVC_LLM["Service: litellm (ClusterIP:4000)"]
        LLM["Deployment: litellm\n(LiteLLM Proxy Gateway)"]
        CM["ConfigMap: litellm-config\n(config.yaml - model routing)"]
        SEC["Secret: litellm-secret\n(LITELLM_MASTER_KEY)"]

        SVC_FE --> FE
        FE -->|HTTP| SVC_LLM
        SVC_LLM --> LLM
        CM -.mounted.-> LLM
        SEC -.env var.-> LLM
    end

    subgraph Providers["LLM Providers"]
        direction TB
        OpenAI["OpenAI"]
        Anthropic["Anthropic (Claude)"]
        Azure["Azure OpenAI"]
        Bedrock["AWS Bedrock"]
        Other["...other providers"]
    end

    User --> LB
    BOT --> SVC_FE
    LLM --> OpenAI
    LLM --> Anthropic
    LLM --> Azure
    LLM --> Bedrock
    LLM --> Other
```
