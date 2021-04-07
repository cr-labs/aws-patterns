# Directory structure

This is how I organized things for ef-open using Cloudformation. Some slight variant
of this pattern would also work for Terraform.

```
/cloudformation

FIXTURES:
  /cloudformation/fixtures/templates/ --- fixture template files
    - filename: <fixture>.json
    - example: /cloudformation/fixtures/templates/s3.json

  /cloudformation/fixtures/parameters/ --- fixture parameter files
    - filename: <fixture>.parameters.<env>.json
    - example: /cloudformation/fixtures/parameters/s3.parameters.prod.json

SERVICES:
  /cloudformation/services//templates -- service template files
    - filename: <service>.json
    - example: /cloudformation/services/templates/account-maintenance.json
  /cloudformation/services/parameters -- service parameter files
    - filename: <service>.parameters.<env>.json
    - example: /cloudformation/services/parameters/account-maintenance.parameters.prod.json

CONFIGS: (we rolled our own config system)
  /configs
    /all -- Config files that are placed on every instance
      /templates -- the config templates
      /parameters -- corresponding .parameters.json files
    /<service_name> -- service-specific config files and templates
      /templates -- the config files
      /parameters -- corresponding .parameters.json files
  - configs were copied by Jenkins to s3://<mycompany>-global-configs/

LAMBDAS
  /lambdas
  - Lambda source code. Source files end with an extension indicating language (.py or .js)
  - Build artifacts copied by Jenkins to s3://<mycompany>-global-dist/lambdas/

PACKER
  /packer
  - packer configurations

SERVICE REGISTRY
  /service_registry.json -- the service registry

TOOLS
  /tools
  - software for use with this content including wrappers for aws cloudformation
