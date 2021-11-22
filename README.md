## CLUSTER LINKING:
Cluster Linking replicates topics from one Kafka or Confluent cluster to another, providing the following capabilities:
Global Replication
Hybrid cloud
high availability and disaster recovery
Cluster migration
Aggregation
Data sharing

Cluster Linking as an inter-cluster replication layer into Confluent Server, allows you to connect clusters together and replicate topics asynchronously without the need for Connect. 

Creates exact mirrors of topics, including offsets, to enable migration, failover, and rationalizing about your system without offset translation or custom tooling.
Requires Confluent Server on the destination cluster
Requires an inter-broker protocol (IBP) of 2.4 or higher on the source cluster, and an IBP of 2.7 or higher on the destination cluster. 
Requires password.encoder.secret to be set on the Confluent Platform destination cluster’s brokers.
The source cluster can be Kafka or Confluent Server or Confluent Cloud; the destination cluster must be Confluent Server, which is bundled with Confluent Enterprise.

# Configurations 
To enable Cluster Linking, add the above configuration to the broker configuration on the destination cluster 
confluent.cluster.link.enable: “true”
Other configurations available:
cluster.link.paused: default false
cluster.link.retry.timeout.ms: default 5 min
availability.check.ms : default 1 min
consumer.offset.sync.enable: default false
topic.config.sync.ms: default 5000ms
There are few other configs available, we can check them out in confluent docs.
Required Configurations for Control Center
Cluster Linking requires embedded v3 Confluent REST Proxy to communicate with Confluent Control Center and properly display mirror topics on the Control Center UI

'#Kafka REST endpoint URL
confluent.controlcenter.streams.cprest.url=“http://localhost:8090"'

Cluster Linking does not require the Metadata Service (MDS) or security to run, but if you want to configure security, you can get started with the following example which shows an MDS client configuration for RBAC.
You can use confluent.metadata.server.listeners (which will enable the Metadata Service) instead of confluent.http.server.listeners to listen for API requests. Use either confluent.metadata.server.listeners or confluent.http.server.listeners, but not both. If a listener uses HTTPS, then appropriate SSL configuration parameters must also be set. 

Following lines are expected in broker server.properties file
# EmbeddedKafkaRest: Kafka Client Configuration
kafka.rest.bootstrap.servers=<host:port>, <host:port>, <host:port>
kafka.rest.client.security.protocol=SASL_PLAINTEXT

# EmbeddedKafkaRest: HTTP Auth Configuration
kafka.rest.kafka.rest.resource.extension.class=io.confluent.kafkarest.security.KafkaRestSecurityResourceExtension
kafka.rest.rest.servlet.initializor.classes=io.confluent.common.security.jetty.initializer.InstallBearerOrBasicSecurityHandler
kafka.rest.public.key.path=<rbac_enabled_public_pem_path>

# EmbeddedKafkaRest: MDS Client configuration
kafka.rest.confluent.metadata.bootstrap.server.urls=<host:port>, <host:port>, <host:port>
kafka.rest.ssl.truststore.location=<truststore_location>
kafka.rest.ssl.truststore.password=<password>
kafka.rest.confluent.metadata.http.auth.credentials.provider=BASIC
kafka.rest.confluent.metadata.basic.auth.user.info=<user:password>
kafka.rest.confluent.metadata.server.urls.max.age.ms=60000
kafka.rest.cilent.confluent.metadata.server.urls.max.age.ms=60000



#Example commands

kafka-cluster-links --bootstrap-server localhost:9092 \
                       --create \
                       --link example-link \
                       --config bootstrap.servers=example-1:9092,example-2:9092,example-3:9092

Example output 

Cluster link 'example-link' creation successfully completed.

To specify the configuration for a secure cluster link in a file named link-config.properties:

bootstrap.servers=example-1:9092,example-2:9092,example-3:9092
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="example-user" password="example-password"
security.protocol=SASL_SSL
ssl.endpoint.identification.algorithm=https

Then, you can create the cluster link example-link with the following command:

kafka-cluster-links --bootstrap-server localhost:9092 --create --link example-link --config-file link-config.properties

Listing Cluster Links
kafka-cluster-links --list --bootstrap-server localhost:9092

Example Output
Link name: 'example-link', link ID: '123-some-link-id', cluster ID: ‘123-some-cluster-id’

Creating a Mirror Topic¶
kafka-topics --bootstrap-server localhost:9092 \
                 --create \
                 --topic example-topic \
                 --link example-link \
                 --mirror-topic example-topic

We can also 
List Mirror Topics¶
Describe a Cluster Link¶
Alter a Cluster Link¶
Delete a Cluster Link¶
Migrate Consumer Groups from Source to Destination Cluster¶

