# OS Intro
  - Kernel Mode (priviledged) vs User Mode
  - *Code run in Kernel mode have complete, unrestricted access to the underlying hardware*
  - **Kernel Space**: memory area reserved to kernel processes
## Typical OS architecture

```mermaid
flowchart TB
    subgraph USER["User space"]
        direction LR
        A1["Application 1"]
        A2["Application 2"]
        AN["Application n"]
    end

    subgraph KERNEL["Kernel space"]
        direction TB
        SCI["System Call Interface"]
        K["Kernel"]
        DD["Device drivers"]
    end

    subgraph HW["Hardware"]
        H["Physical hardware"]
    end

    A1 --> SCI
    A2 --> SCI
    AN --> SCI

    SCI --> K
    K --> DD
    DD --> H

    style USER fill:#ffffff,stroke:#777
    style KERNEL fill:#ffffff,stroke:#777
    style HW fill:#ffffff,stroke:#777

    style A1 fill:#f8f8f8,stroke:#000
    style A2 fill:#f8f8f8,stroke:#000
    style AN fill:#f8f8f8,stroke:#000
    style SCI fill:#f8f8f8,stroke:#000
    style K fill:#f8f8f8,stroke:#000
    style DD fill:#f8f8f8,stroke:#000
    style H fill:#f8f8f8,stroke:#000
```
