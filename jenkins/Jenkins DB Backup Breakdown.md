![Pasted image 20260910232717.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260910232717.png)![Pasted image 20260910232912.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260910232912.png)![Pasted image 20260910233027.png](https://github.com/felicitousbeing999-h/Obsidian01/blob/main/Pasted%20image%2020260910233027.png) 

## Visual Execution Path

Code snippet

```mermaid
sequenceDiagram
    autonumber
    actor Boss as Cron Timer (*/10)
    participant J as Jenkins Box
    participant App as stapp01 (App Server)
    participant Stor as ststor01 (Storage Server)

    Boss->>J: Wake up! Run job
    Note over J,App: Step 1: Tell App to dump its own DB
    J->>App: SSH command: "Run mysqldump, save to /tmp"
    App-->>App: Generates db_YYYY-MM-DD.sql locally
    
    Note over J,App: Step 2: Grab the file
    J->>App: SCP command: "Give me that /tmp file"
    App-->>J: File lands in Jenkins /tmp

    Note over J,Stor: Step 3: Yeet file to safe storage
    J->>Stor: SCP command: "Take this file into /home/natasha/db_backups"
    Stor-->>J: File saved!
    
    Note over J: Job Success (Exit 0)
```

## What Actually Happened

Code snippet

```mermaid
flowchart TD
    subgraph S1 ["1. The Trigger"]
        T["Timer goes tick: */10 * * * *"] --> JK["Jenkins wakes up"]
    end

    subgraph S2 ["2. The Extraction"]
        JK -- "SSH: Make dump" --> AP["stapp01<br/>(MySQL DB: kodekloud_db01)"]
        AP -- "Creates file" --> DUMP1["stapp01: /tmp/db_DATE.sql"]
    end

    subgraph S3 ["3. The Relay"]
        JK -- "SCP pull" --> DUMP1
        DUMP1 --> DUMP2["Jenkins: /tmp/db_DATE.sql"]
    end

    subgraph S4 ["4. The Final Drop"]
        DUMP2 -- "SCP push" --> ST["ststor01<br/>(/home/natasha/db_backups/)"]
    end
```

## Server Role Card

- **Jenkins:** The middleman runner. Has no database itself. Calls everyone via SSH.
    
      
    
- **`stapp01`:** The worker box holding the live MySQL database. Created the raw SQL dump on demand.
    
      
    
- **`ststor01`:** The storage vault. Did zero work, just received the final `.sql` file.