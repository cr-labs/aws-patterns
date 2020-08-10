# Names

## Rules for all names
- Unless the pattern explicitly states otherwise, use lowercase characters a-z, digits 0-9, `-` and `_` only in names.
- '-' (hyphen) is a delimiter between fields of a compound name composed of two or more fields.
    _example:_
    ````
    prod-auth (a security group name)
    ````

## AWS accounts
If using ef-open, environments can share accounts or have their own.
The reference implementation places 'prod' in one AWS account, and 'staging' and 'proto' in another.
AWS allows accounts to have have aliases. We use these aliases in some global resource names
| environment | account shortname | Example |
| :---------- | :------ | :------ |
| prod and prod-related global | `<AWS_PROD>`  | `mycompany` |
| staging, proto, and non-prod global | `<AWS_NON-PROD>` | `mycompany-np`  |


## Service Short Names
Every service and fixture has a *short name* that identifies it in name strings throughout the infrastructure (AWS, config files, tools).
- Replace `<service>` in any name pattern with this short name.
    _example:_
    ````
    full service name: Authentication Service
    short name: auth
    ````

## Environment names
Environment names comprise a segment of resource names and paths to configuration and other environment-specific data. Generally the short name of the environment is used.

| Environment | Description | Notes |
| :--- | :--- | :--- |
| prod | production environment | Exists in a separate AWS account from all other environments |
| staging | staging environment | Last stop before production. Configured similarly to production, but with smaller/fewer resources (e.g. 1 or 2 small instances rather than N large ones) |
| proto<N> | prototype environment | Ephemeral stacks. At most, ten (0..9) |
| global | not in any environment* | Special case for lambdas, S3 buckets, and not much else. These resources are accessible from, and in the service of, assets in multiple environments (such as shared S3 buckets containing configuration) |


\* literally `<ENV>` in patterns  
\** this was a hard decision: is `global` _in all environments_ or _in no environments?_ The AWS model drives the strongest case for "in no environments, but has partitioned visibility and access to/from all environments."

## Security Group, Role and CloudFormation Stack Names

| Type (see Service Registry) | Security Group name(s) | Role name  | Cloudformation Stack name |
| :--- | :--- | :--- | :-- |
| **aws_ec2** | `<env>-<service>-ec2` | `<env>-<service>` | `<env>-<service>` |
| _examples:_ | prod-auth-ec2 | prod-auth | prod-auth |
|   |   |   |   |
| **http_service** | ec2:`<env>-<service>-ec2`<br />elb:`<env>-<service>-elb` | ec2:`<env>-<service>`<br />elb: N/A | `<env>-<service>` |
| _examples:_ | `prod-auth-ec2`<br />`prod-auth-elb`  | `prod-auth`  | `prod-auth` |
|   |   |   |   |
| **aws_lambda** | `<env>-<lambda_base_name>-lambda` | `<env>-<lambda_service_name<` _(see Lambda Names below)_ |  `<env>-<lambda_service_name>` _(see Lambda Names below)_ |
|   |   |   |   |
| _examples:_ | `prod-i-do-something-lambda`* | `prod-i-do-something`\**<br />`prod-auth-i-do-something`<br />`staging-auth-i-do-something` |
|   |   |   |
| **aws_fixture** | N/A | N/A | `<env>-<service>` |
| _example:_   | n/a | n/a  | `prod-network`<br />`prod-vpc`  |
|   |   |   |   |
| **aws_security_group**   | `<env>-<service>`  | N/A  | `<env>-<service>` |
| _examples:_  | `staging-cloudfront-ingress` | n/a  | `staging-cloudfront-ingress`  |

\* _for better or worse, this pattern allows hyphens in names. But the hyphen is also a delimiter. If you can persuade people to not use hyphenated names, do it._
\** `prod-i-do-something` is a lambda that does something within the prod environment, but not for a specific service.<br />
`prod-auth-i-do-something` and `staging-...` are lambdas that do something as components of the auth service in the prod and staging environments.

## Security Policy names
- Inline policies
  - name should be descriptive of what the policy provides
  - lowercase_with_underscore_word_separators
  - reminders when working with ef-open:
    - inline policies for services are created by `sync-roles-and-security-groups` based on the service registry, using policy templates from the `/policy_templates` directory
  - policies should be defined within the same template that creates them
  - example security policy names:
    - global_buckets_ro (allows global env to read access to all buckets)
    - instance introspection (allows EC2 instance to access data about itself)
- Managed policies
  - Do not use managed policies. All policies are inlined on roles. *
\* _This is due for a re-think. AWS has added to IAM since these guidelines were written._

## EC2 Subnet, VPC, and VPN names
Pattern: `aws_resource_name ::= [subnet|vpc|vpn]-<env>`
Examples:
- `subnet-staging`
- `vpc-prod`

## EC2 Instance names, and names for AWS resources not defined here
Special considerations for EC2 instances that run single services:
- Every instance serves a particular function, known as its "role"
- The name of the role is the name of the service
- Every instance is a member of a Security Group that controls its network access, and an EC2 role that controls its AWS resource access
Pattern:
- `aws_resource_name ::= <type>-<env>-<service>[-<seq>]`
  - `<service>` is the short name for the service, like "auth"
  - `<type>` is the common aws name for the service, like "ec2, s3, elb, kms, ..."
  - Optional `<seq>` counts from 0; no guaranteed to be ordered or sequential; avoid when possible; use when a resource definite may have more than one otherwise identically-named instance
  - Examples:
    - `elb-prod-auth-01`, `ec2-prod-auth-03`

## AWS S3 Bucket Names and Well-Known Paths
### References:
- S3 User Guide (at AWS)
- S3 bucket names are globally unique; there is no way to reserve namespace

### General rules for S3 buckets
- Use lowercase characters only in bucket names
- Use lowercase characters only in key paths, unless keys are generated by code.
- If keys are generated by code, generate names from [a-z0-9]
- Avoid case-sensitive names because the OS/X filesystem is case-respecting but not case-sensitive and case-sensitivity will cost a bundle eventually
- S3 buckets exist outside account boundaries. Prefix all buckets with `<COMPANY>` to simulate a namespace
- Collision risk: the patterns below rely on a set of buckets being available. If a bucket name is already taken by someone else... (punt at the moment; there aren't many cases like that here)

### Well-known S3 buckets
Note that every environment in (prod, staging, proto) has a full set of buckets

| Bucket suffix | What's in the bucket |
| :------------ | :------------------- |
| `-configs` | Application/service configuration data. Configuration buckets are shared; services' configuration files and parameters for environment localization are on paths in the config bucket. In the reference implementation, configs are managed in git and copied to S3 by CI when there is a commit. Configs are read by the configuration portion of the startup script at instance initialization.
| `-credentials` | Encrypted credentials used by apps to sign on to RDS, SES, and other services that require a credential. When used with ef-open, credentials are written by the credential-management tool. Credentials are read by the configuration portion of the startup script at instance initialization. |
| `-dist` | Build artifacts are dropped here. Written by CI, read by instances at startup to load application code |
| `-s3logs` | S3 bucket activity logs for the buckets above  |

### Per-environment S3 buckets
| Environment | Secret credential buckets | Secret credential bucket logs | Owned by account\* |
| :---------- | :------------------------ | :---------------------------- | :--------------- |
| proto0..proto<N> | `<AWS_NON-PROD>-proto<N>-credentials` | `<AWS_NON-PROD>-proto<N>-s3logs` | `<AWS_NON-PROD>`
| staging | `<AWS_NON-PROD>-staging-credentials` | `<AWS_NON-PROD>-staging-s3logs` | `<AWS_NON-PROD>`
| prod | `<AWS_PROD>-prod-credentials` | `<AWS_PROD>-prod-s3logs` | `<AWS_PROD>`
| global: nonprod |`<AWS_NON-PROD>-global-credentials` | `<AWS_NON-PROD>-global-s3logs` | `<AWS_NON-PROD>`
| global: prod |`<AWS_PROD>-global-credentials` | `<AWS_PROD>-global-s3logs` | `<AWS_PROD>`

\* _See separate discussion of AWS account factoring._

### Shared S3 buckets
In some cases, we define buckets that hold common stuff: build artifacts (since builds are shared across environments), and configuration for all environments (separated by paths, in a config system that can overlay a base config with environment-specific values). This is the reference pattern for ef-open, but it also fits the pattern used in puppet's hiera, and other config systems.

Security pattern is also usable in other models: The shared buckets (or shared config storage, whatever that is) are owned by the prod account which can issue credentials to revise the contents of the bucket or config records. Environment- and service-partitioned read-only access to paths within the bucket or shared storage is granted to other accounts/environments.
| Contents | Bucket name and path in the bucket | Owned by account |
| :------- | :--------------------------------- | :--------------- |
| Build artifacts - applications | `<AWS_PROD>-global-dist/<service>/<app>/<artifact>.gz` | `<AWS_PROD>`  |
| Build artifacts - global lambdas | `<AWS_PROD>-global-dist/lambdas/<lambda_base_name>/<artifact>.gz` | `<AWS_PROD>` |
| Instance and lambda config  | `<AWS_PROD>-global-configs/<service>/...`<br /> `<AWS_PROD>-global-configs/all/...`  | `<AWS_PROD>` |

### Backup S3 buckets
For some S3 buckets, we may deploy a backup bucket in another region that holds a copy of everything in the primary bucket.

- `-credentials` buckets and buckets in the `proto` environment are not backed up. See details in _System Security_.
- Backup bucket name is the primary bucket name with the primary bucket's region and AZ appended, and `-` replaced by `_`
Pattern: `<primary_bucket_name>-<primary_region_id>`
Examples:
- `mycompany-staging-images-us_west_2`

## S3 Configuration key paths
Configuration data are usually managed by both infrastructure teams (who manage the tooling that configures servers and services) and service owners (who decide what is configurable about their services).

The reference implementation places a static configuration on instances when they are launched. Configuration objects are processed during system initialization to build config data structures that are then used by apps when they start up.

Bucket:
- `<AWS_PROD>-globals-configs`
Pattern:
- `configuration_template_path ::= /<service>/templates/<filename.ext>`
- `configuration_parameter_path ::= /<service>/parameters/<filename.ext>.parameters.json`
Examples:
- `/auth/templates/nginx.conf`
- `/auth/parameters/nginx.conf.parameters.json`

## S3 paths for build artifacts
Bucket:
- `<AWS_PROD>-global-dist`
Pattern:
- `artifact_path ::= /<service>/[app]/<artifact>`
Example:
- `/auth/auth-app.zip`

## S3 paths for non-configuration/non-credential data
- Key paths should be defined by the service owner in whatever manner is appropriate for the service's needs
- Document key paths patterns in the service runbook
- S3 was once longer sensitive to _get_ or _put_ rates that could create hot keys, requiring key prefixes to be evenly sharded. Since July 2018, that is no longer the case. See [AWS Guidance](https://aws.amazon.com/about-aws/whats-new/2018/07/amazon-s3-announces-increased-request-rate-performance/)


## S3 paths for S3 activity logs
- First, determine your logging policy to define when logging is required, and what the log handing and retention practices will be.
- All log buckets are owned by `<AWS_PROD>`
- When logging is required, use these bucket and path patterns to configure the logging target.
Bucket name pattern:
- `log_bucket_name ::= <AWS_PROD>-<environment>-s3logs`
Path pattern:
- `[service]-<description>/`
Examples:
- `<AWS_PROD>-staging-s3logs/auth-authserver`
- `<AWS_PROD>prod-s3logs/credentials`


## S3 paths for credentials
- Credential objects are stored in a shared credential bucket in the ef-open reference implementation
- Use cases for credentials include database keys, Github read-only deploy tokens, and other secrets needed by services
- Credentials should only be stored in 'credentials' buckets (name pattern above) which are always encrypted and access-restricted
Pattern:
- `credential_key_path ::= /<resource>/<type>[/<type_qualifier>]`
- `<resource>` is the common short name of any AWS resource, or of a service. Examples of valid values:
  - cloudfront: the cloudfront service (providing a private key for edge authentication)
  - `<service>`: the shortname of any service (AWS or your service) that requires authentication
  - `<database>`: the common name of a database (can be arbitrarily chosen, but do so consistently)

- `<type>` is the description of the actual object stored at this key. Valid values for `<type>`:
  - `rss_private_key`
  - `rsa_public_key`
  - `smtp_username` (e.g. to authenticate to SES)
  - `smtp_password` (e.g. to authenticate to SES
  - `password`
  - `username`
  - `blob` (a data structure understood by the app, for example JSON with username and password together in one object)
- `type_qualifier>` distinguishes multiple distinct keys of the same type with unique purposes. For example, a database may have both read-only and read+update username:password pairs. If a `type_qualifier` is not needed, omit it.
  - Examples:
    - `/cloudfront/rsa_private_key` : the Cloudfront private key
    - `/user_rds/username/readonly`
    - `/user_rds/password/readonly`
    - `/user_rds/username/update`
    - `/auth_rds/json/update`

## AWS RDS DNS Names
Pattern for RDS instances owned by one service:
- `rds_dns_name ::= rds-<env>-<service>-[<description>]`
  - Optional `<description>` distinguishes DBs when a service owns more than one RDS
Pattern for RDS instances shared by multiple services with no clear owner
- `rds_dns_name ::= rds-<env>-<description>`
Implementation and example
- RDS instances come up with arbitrary host names
  - `rds-prod-auth.42er53s3ft3.us-west-2.rds.amazonaws.com.`
- To make the RDS findable by apps at a predictable name, we map host CNAME following the patterns above, to the RDS's real name
  - `rds-prod-auth` (DNS naming is discussed separately)

## Lambda and lambda-related names
|     | Pattern and how it's used | Example name |
| :-- | :------------------------ | :----------- |
| lambda_base_name | `lambda_base_name ::= <description>` | `i-do-something` |
|   |  What you'd call it if deployment details didn't matter. This name is the basis of all other names.<br/>Valid characters: [A_Za-z0-9-] | |
| | | |
| lambda_function_name   | `<env>-[<service>-]<lambda_base_name>`  | `global-i-do-something`<br/>`prod-i-do-something`<br/>`staging-auth-i-do-something`   |
| | | |
| lambda_artifact_name | `lambda_artifact_name ::= <lambda_base_name>.zip` | `i-do-something.zip` |
|   | Packaged lambda with dependencies, as built and written directly in S3. |   |

## CloudFormation names
Stack name
- Pattern: `stack_name ::= <env>-<fixture_or_service_name>`
- Examples for fixtures: `prod-s3`, `proto1-network`, `prod-i-do-something`
- Examples for services: `prod-auth`, `proto3-mailblaster`

Template filename
- Pattern: `template_filename ::= <fixture_or_service_name>.json`
- Examples: `network.json`, `auth.json`

Parameter-file filename
- Pattern: `parameter_filename ::= <fixture_or_service_name>.parameters.<env>.json`
- Examples: `network.parameters.proto0.json`, `s3.parameters.staging.json`

Style for references inside CloudFormation Templates
- CamelCase
- Examples: `Vpc`, `SubnetA`, `SubnetB`, `Dns`, `BillingCron`
