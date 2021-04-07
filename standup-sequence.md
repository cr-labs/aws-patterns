# Standup sequence

This defines the order of operations to stand up the global fixtures (one set, total, for a system), as well as environment-specific fixtures (one set per environment) environment-specific services.

If not using ef-open (terraform or something else), the commands will vary but the standup sequence should still be run in this order.

<style>
table th:first-of-type {
    width: 30%;
    vertical-align: top;
}
table th:nth-of-type(2) {
    width: 70%;
    vertical-align: top;
}
</style>

### Global fixtures
_once for production, and once for nonprod which share an account_
What  | Command to stand it up |  
|--|--|
| s3 buckets:<br/>_shared s3 buckets are created in, and owned by, the prod account except the credentials bucket (if using one)_ | ef-cf /path/to/s3.json global \<mycompany><br/>ef-cf /path/to/s3-\<mycompanydev>-credentials.json global \<mycompanydev> |
| global lambdas (if any) | ef-cf /path/to/\<TBD> global \<mycompany><br/>ef-cf /path/to/\<TBD> global \<mycompanydev><br/>_TBQH I wrote this pattern but did not deploy any global lambdas. Still, here's a place for them._ |
### Environment-specific fixtures
_create once per environment; don't tear down even for the prototype environments_
What  | Command to stand it up |  
|--|--|
| network fixtures:<li>network<li>vpc<li>subnet(s)<li>gateway(s) | ef-cf /path/to/network.json \<env> |
| vpn<br/>_may require addiional setup if at installation the VPN creates a pre-shared key that ust be manually entered into the VPN endpoint_ | ef-cf /path/to/vpn.json \<env> |
| s3 per-environment buckets  | ef-cf /path/to/s3.json \<env> |
| resources shared by multiple services (e.g. shared RDS, Redis, memcached, SQS) if any | ef-cf /path/to/? \<env>  |
| <li>generate roles and security groups for every \<env>-\<service><li>set policies on roles from /policy_templates directory<li>generate Customer-Managed Keys (CMKs) in KMS for every \<service>  | ef-generate /path/to/service_registry.json \<env> --commit |
| populate fixture security groups _e.g. set the office IP addresses in the CIDR security group_ | ef-cf /path/to/sq.json \<env>  |
| per-environment lambdas | ef-cf /path/to/lambda.json <env> |
| logging  | ef-cf /path/to/elk.json \<env>   |

### Services
_these should be able to stand up in any order though we may have a preferred order_
What  | Command to stand it up |  
|--|--|
| individual services with their private resources | ef-cf /path/to/\<service_short_name>.json \<env>
| service post-standup initialization   | e.g. initialize database (TBD)  |
|   |   |
