# SOA checklist

Disclaimer - Trust no one, use your brain! (**Work continuously in progress**)

## Administrative (people, flows, responsibilities)

* Blueprint/template for a new service
* Documentation, standards, guides (how-to, know-how documents)
* Team support
   * Understanding of the whole process by each member
   * Pro-active development and support
   * Accepted responsibilities and duties for each stage of a service
* Plan for service live circle
   * Pre-production development
   * Launching
   * Rollout backward compatible version of a service
   * Hotfixing
   * Rollout backward incompatible version of a service
      * Data migration
      * Switchover
      * Service rollback


##Automated processes

* Continuous development
   * Tests
      * Automated
         * Unit tests
         * Functional tests
         * Code style (lints and sniffers)
         * Code quality monitoring ([Sonar](http://www.sonarqube.org/), [Scrutinizer](https://scrutinizer-ci.com/))
         * Code coverage checks
      * Manual
         * Feature acceptance/Business acceptance
         * A/B tests  
   * Conditions of integration
      * Code style checks
      * Test results
      * Code coverage percentage
   * Conditions of disintegration a feature
      * Error rate after deploy live
      * Helthchecks
   * Storing a new tested snapshots/artefact of a service
      * Artefact storage (Docker registry)
         * Cleanup policy (Delete old tags with timeout)
* Continuous delivery of stable artefacts 
   * [Images builder](https://github.com/paunin/images-builder)
   * Services provisioning([Ansible](https://www.ansible.com/))

##Implementation

* System layers
   * Hardware: Servers and networks
      * Scaling (adding new nodes) should not affect consistency of other layers
      * Degradation (removing nodes)  should not affect consistency of other layers
      * Monitoring
         * Hardware
         * Network
         * Resources and load
      * Alerting policy
   * Cluster: Services management system  ([Kubernrtes](http://kubernetes.io/), alternatives: [OpenShift](https://www.openshift.com),[Apache Mesos/Apache Karaf](http://servicemix.apache.org/))
      * Monitoring
         * Availability of each node in the cluster
         * All services up and running
         * Connectivity between different pods and services
         * Public endpoints accessibility 
      * Alerting policy
      * Restart (full or partial) should bring cluster and systems up without destruction
      * Log aggregation system - collect all logs from all containers
      * Execution environment
         * Meta-project with topology of the system
            * Showroom + Staging 
               * Separate namespace for each showroom
               * Fixed showroom for the staging (last stable pre-release)
            * Production
               * Configuration
                  * Secrets
                  * Configs should be a part of the meta-project 
   * Service: Application and any service
      * Service itself (Docker image)
         * Backward compatibility for a few generations
            * Cleanup policy for deprecated/unused:
               * Logic branches
               * Data structures (RDBMS/NoSql)
         * One container - one process
            * Segregated commands even in one image (management layer can pick any to run)
            * Built in commands
               * Test service/source code (docker compose to setup required test ENV)
            * DEV/DEBUG mode
         * Logging
            * Writing in stdout (without using containers’ file system) will enforce cluster layer to keep all logs
         * Monitoring
            * Application and business checks (New Relic: throughput, metrics)
            * Self health checks (metrics+[Prometeus](http://www.prometeus.net/site/)+[Grafana](https://grafana.org/))
               * Queues content (amount of messages)
               * Db content (custom checks)
               * Cache utilization check
         * Alerting policies ([Prometeus](http://www.prometeus.net/site/), [NewRelic](https://newrelic.com))
         * Self-sufficiency 
            * Interfaces documentation
               * Restful API
                  * [Swagger](http://swagger.io/)
               * Port and service description (README.md files)
            * Service should be able to set itself up
               * Wait for required related services and ports ([dockerize](https://docs.docker.com/compose/startup-order/))
               * Configuring from environment variables ([confd](https://github.com/kelseyhightower/confd))
               * Warming up
                  * Run data migration (needed maintenance service)
                  * Cache fulfilment
      * Replication, balancing and scaling on service level
      * Failover and self-reorganisation in case of:
         * Service crashed
         * Physical node out of cluster
         * Resources problems on specific node 
      * Logs system
         * Service to collect and access logs grabbed from Cluster layer
            * ELK stack/Gray Log/etc
      * Persistent volumes to keep data
         * EBS AWS
         * Ceph
         * NFS
* Common services 
   * Single sign-on service
      * Authentication service ([JWT](https://jwt.io/))
      * Authorization requests from all services
   * Detached processing ([CQRS](http://martinfowler.com/bliki/CQRS.html))
      * Request-Queue-Processor schema
      * Stream data addressing and processing ([Reactor](https://projectreactor.io/))
   * Real Time data requests processing
      * Reliable data provider/API gateway (sync data retrieving)
         * Request-Manager-Service solution
   * Reliable data-bus for events
      * Event-Broker-Subscriber solution ([Apache Camel](http://camel.apache.org/))
         * Http/TCP API endpoint to accept events
         * Event fulfilment (Earn required information for subscribers)
         * Event delivery
         * Event delivery policies
            * Retry
            * Reque
            * Giveup
   * RDBMS: [Postgres cluster](https://github.com/paunin/postgres-docker-cluster)
   * DB backups: [PG backupper](https://github.com/paunin/pg-backupper)
   * Key-value + Queue: [Redis cluster](https://github.com/relaxart/kubernetes-redis-cluster)
   * Messages system: [Rabbit MQ cluster](https://github.com/relaxart/docker-rabbitmq-cluster)
   * Healthcheck system
   * Alerting system
