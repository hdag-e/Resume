graph TD
    classDef edge fill:#f9f,stroke:#333,stroke-width:2px;
    classDef internal fill:#bbf,stroke:#333,stroke-width:2px;
    classDef core fill:#f96,stroke:#333,stroke-width:2px;
    classDef server fill:#bfb,stroke:#333,stroke-width:2px;

    WAN[Edge WAN / External<br>203.0.113.0/24] ::: edge --> RTR_EDGE[RTR-EDGE-01<br>ISR 4331 Edge Router] ::: edge
    
    RTR_EDGE --> RTR_INT1[RTR-INT-01<br>Internal Router] ::: internal
    RTR_EDGE --> RTR_INT2[RTR-INT-02<br>Internal Router] ::: internal
    
    RTR_INT1 --> SW_DIST[SW-DIST-01<br>3560 Distribution Switch] ::: core
    RTR_INT2 --> SW_DIST
    
    SW_DIST --> SW_ACC1[SW-ACC-01<br>Access Switch] ::: internal
    SW_DIST --> SW_ACC2[SW-ACC-02<br>Access Switch] ::: internal
    SW_DIST --> SW_ACC3[SW-ACC-03<br>Access Switch] ::: internal
    
    SW_ACC1 --> USERS[User Subnet<br>10.20.10.0/24<br>PCs 1-4]
    SW_ACC1 --> GUEST[Guest Subnet<br>192.168.50.0/24<br>Laptop]
    
    SW_ACC2 --> ADMIN[Admin Subnet<br>10.20.20.0/24<br>PCs 5-8]
    SW_ACC2 --> MGMT[Management Subnet<br>10.20.99.0/24]
    
    SW_ACC3 --> LOGGING[Logging & Time Subnet<br>10.20.50.0/24<br>SRV-SYSLOG & SRV-NTP] ::: server
    SW_ACC3 --> SERVERS[Server Subnet<br>10.20.30.0/24<br>SRV-APP-01 & 02] ::: server
