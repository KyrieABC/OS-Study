# OS Intro
  - Kernel Mode (priviledged) vs User Mode
  - *Code run in Kernel mode have complete, unrestricted access to the underlying hardware*
  - **Kernel Space**: memory area reserved to kernel processes
## Typical OS architecture
flowchart TD
    subgraph UserSpace [User space]
        direction LR
        App1[Application 1] --> App2[Application2] --> AppDots[...] --> AppN[Application n]
    end

    subgraph KernelSpace [Kernel space]
        direction TD
        SCI[System Call Interface]
        Kernel[Kernel]
        Drivers[Device drivers]
        SCI --> Kernel --> Drivers
    end

    subgraph Hardware [Hardware]
        HW[Hardware]
    end

    App1 --> SCI
    App2 --> SCI
    AppN --> SCI
    Drivers --> HW