# Fixtures and services

This architecture has two main kinds of elements: Fixtures and Services.

## Fixtures
Fixtures are shared resources (not owned or used exclusively by one service) that rarely or never change once created. For example, S3 buckets containing service configurations are created once and remain in existence indefinitely, in all environments.

Fixtures are not deleted when an environment is torn down - not even in ephemeral prototype environments.

As with an other architectural resource, define fixtures in Terraform or CloudFormation templates. Store those templates along with their accompanying parameter (environment-specific config) files in the repo.

Fixtures and their templates reside in the repo at:
`<repo>/fixtures/templates/<fixturename>.json` (for Cloudformation)
`<repo>/fixtures/parameters/<fixturename>.parameters.<env>.json` (for Cloudformation)

As with a Service, a Fixture may stand up a resource differently depending on the environment it's in. For example, a network fixture may create one subjet in one availability zone for prototype environments, but two subnets in two availability zones for staging and production environments.

Fixtures include:
- Network structure: VPC + Subnets inside the VPC
  - This will rarely if ever be revised once stood up, because replacing a VPC or subnet implies replacing or re-connecting the entire environment
- Security Groups (one per service, name is `<env>-<service`)
  - Updated when a new service appears, or a service adds/loses connections, or is retired
- Roles (one per service, name is `<env>-<service`)
  - Updated when a new service appears, a service adds/loses API privileges, or is retired
- Security Group rules connecting services to each other or to shared resources
  - Define ingress and egress rules together, so that cross-service and service-to-shared-resource and service-to-fixture rules are plainly visible. Do not define service access to private resources here (such as a service to its private RDS instance). Place those ingress/egress rules in the template for the service
- Fixture lambdas
  - Lambdas that are used for multiple services. Some may be inside an environment (VPC). Some may be outside all environments (not in any VPC)
- Shared S3 buckets
  - Rarely if ever revised once stood up, due to dependencies from all over
  - Shared S3 buckets I used with ef-open included: credentials, configuration (we rolled our own service config kit), lambdas (deployable artifacts), and logging for S3 buckets
- Other shared resources that aren't owned by a single service
  - Logging services (like ELK)
  - Metrics systems
  - Widely shared memcache or Redis caches
  - Lambdas that servce the entire environment or that provide resources to multiple services or production tools

## Services
Services are... everything else. The modular units that implement the functionality of the product.

Services and their templates reside in the repo at:
`<repo>/services/templates/<servicename>.json` (for Cloudformation)
`<repo>/services/parameters/<servicename>.parameters.<env>.json` (for Cloudformation)
