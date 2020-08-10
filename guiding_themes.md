# Guiding themes

## Hosted service > custom-built service
- We can't run it as well as "they" can (they're running at huge scale, we aren't)
- ... for most things
- AWS in particular has the ability to failover (RDS, Elasticache, DNS, ...) that we can't do better "on top of" AWS (in ordinary cases)
- However, AWS services may lag current open source functionality, or our requirements may be so specialized that the hosted service is obviously not the best choice. When that is the case, we roll our own, knowing there will be an ongoing cost for maintenance and operations.

## Library beats Service, aka Static beats Dynamic
- Reduce dependencies around instance config and operation
  - loading from a URL or puppet master creates an unnecessary real-time dependency for instance bring-up
  - Prebuilt AMIs (for conventional hosts) or Docker images, assure everything is ready to go before we launch
- "If AWS is running, we're running"
  - S3 can be the sole real-time requirement for instance launch dependencies
    - Configuration at instance bring-up: load config files from S3, place, use
    - Deploys: Load application code from S3, unzip, execute
- Tools should read state from AWS and instances, and interact with real-time state. Tools should not store a private record of the state of any system whose true state can be read in realtime at time of use

## Generic > bespoke
- A generic application server that launches service code beats bespoke service hosting/config in lower complexity to design/deploy, and less operational complexity and training due to commonalities across all services
- Pull, not push; instances bring themselves up
  - Shared base image (AMI or Docker image)
  - Pull environment-specific config (paths and credentials for env-specific resources)
  - Pull application code for microservices based on a version registry ("what version of service S should be running in environment E?)

## Simple > Complex in a virtual environment
- Beware of artificial optimizations
  - No service packing: one service per instance for bare instances; one service per pod (plus directly-supporting sidecars) for Docker/Kube
- "Service" = distinct IP address and/or port, entirely separable w/o telling anyone
- Use AWS HA features when available
  - RDS and Elasticache, secondary in a different AZ, auto-failover
    - DNS-based failover requires app awareness for smooth failover; consider using Aurora instead of RDS (possibly higher cost); no great solution for Elasticache - a cache proxy like mcrouter can help

## Distinguish fixtures from services
- "Fixtures" are more-or-less permanent objects. They don't cost money (or have nominal cost) if not being used, and are "always there"
  - Examples: VPC, Subnets, Security Groups, Roles, Global and Shared S3 buckets, DNS, CloudFront
- "Services" are everything else - microservices and their _private_ resources
  - Examples: application servers for one service, the servic's private mySQL on RDS or Redis on Elasticache, ...
