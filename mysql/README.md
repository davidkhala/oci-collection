# Oracle MySQL Database Service (OCI MDS)

MDS public endpoint
- https://einstein.oracle.com/apex/f?p=300000:Q:117303034888538::::QUESTION:direct-connection-from-on-premise-to-mysql-database-service-2781
- For security reasons, it is not possible to assign a public IP directly to the MySQL endpoint, only private IPs.
- Load Balancing service isn't designed to support MySQL protocol. 
- As an alternative, you can install **[MySQL Router](https://gist.github.com/alastori/005ebce5d05897419026e58b9ab0701b)** in a Compute VM to redirect the traffic to MySQL endpoints.

## Provision
### Before provision
0. Prepare VCN/Subnets, User Group, Policies
1. Choose one option among [ Standalone | High Availability | HeatWave ]
2. Configure Hardware
  - MySQL.HeatWave.VM.Standard.E3 is (16 core, 512GB) the only option for now
  - Configure Data Storage size, default is 1024 GB
## Limit
- general query log is not available
- MySQL Enterprise Audit is not on roadmap
