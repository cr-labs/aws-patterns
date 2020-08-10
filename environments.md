# Environments

This defines some common terminology about "environments". The purpose of each
environment, apart from "prod", is somewhat subject to local preference and
custom. This document doesn't get into that.

These patterns assume "staging" is a test step before "production" but nothing here or in ef-open tooling requires it be used that way.

## Local developer environment ("localdev")
This is the developer environment running on a local personal computer, probably in a VM. "Localdev" could also be a virtual host in the cloud.

## Hosted environments
Every hosted environment contains a full stack of resources (fixtures and services) in AWS, in a VPC that is exclusively for that environment, separate from all other environments, except:
- Data stores containing scrubbed test data, which could be moved in and out of proto or staging VPCs from time to time
- S3, which by design is not contained within any VPC

### Production ("prod")
- Belongs to a different AWS account than all other environments  (called "<AWS_PROD>" in other docs)
- Runs the current release of all services plus supporting services within the environment, like CI
- Has auto-scaling and performance enhancement
- Mostly hands-off for humans; ideally all hands-off

### Staging ("staging")
- Runs the current release candidate of all services, which are promoted individually to production when they pass here <-- _your concerns with version compatibility are warranted_
- Structured much like prod, but fewer instances in each service
- Persistent
- No autoscaling
- Integration testing and validation happen here <-- _up for debate_
- The stack is generally understood to be up at all times, though it may be broken due to issues with services being tested

### Prototype ("proto0 .. proto9")
Prototype environments are ephemeral. They are stood up when needed for testing and experimentation, then often torn down, though they may exist in a partially-built state for an indeterminate period of time.
