# Cloud SQL DataGrip Driver

Builds the Google Cloud SQL Java Connector PostgreSQL fat JAR for use with DataGrip/IntelliJ.

Google no longer publishes prebuilt fat JARs for the Java Connector (since v1.14.0), while DataGrip's custom JDBC driver setup benefits from the `jar-with-dependencies` artifact. This repository builds that artifact from the official Google source code in GitHub Actions.

## Build

1. Open **Actions** → **Build Cloud SQL PostgreSQL fat JAR**.
2. Choose **Run workflow**.
3. Enter the upstream Cloud SQL Java Connector version, for example `1.30.0`.
4. Download the generated `postgres-socket-factory-<version>` artifact from the workflow run.
5. Unzip it and add the `postgres-socket-factory-*-jar-with-dependencies.jar` file to DataGrip under the PostgreSQL driver's **Driver Files / Additional Files**.

The workflow checks out the corresponding tag directly from `GoogleCloudPlatform/cloud-sql-jdbc-socket-factory` and runs the upstream-documented command:

```bash
mvn -P jar-with-dependencies clean package -DskipTests
```

## DataGrip setup

Use the normal PostgreSQL JDBC driver (`org.postgresql.Driver`) plus the generated Cloud SQL connector JAR.

Example JDBC URL:

```text
jdbc:postgresql:///app?cloudSqlInstance=PROJECT:REGION:INSTANCE&socketFactory=com.google.cloud.sql.postgres.SocketFactory&enableIamAuth=true&sslmode=disable
```

For local user credentials, configure Google Application Default Credentials (ADC):

```bash
gcloud auth application-default login
```

For IAM database authentication, the PostgreSQL password field may still need a non-empty placeholder because of JDBC driver validation; the Cloud SQL connector supplies the IAM authentication token.

## Source

This repository does not modify the Cloud SQL Java Connector source tree. The build workflow fetches a requested upstream release tag and uploads the resulting build artifact for convenience.
