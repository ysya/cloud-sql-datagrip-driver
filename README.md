# Cloud SQL DataGrip Driver

Prebuilt Google Cloud SQL Java Connector PostgreSQL fat JARs for use with DataGrip/IntelliJ.

Google no longer publishes prebuilt fat JARs for the Java Connector (since v1.14.0), while DataGrip's custom JDBC driver setup benefits from the `jar-with-dependencies` artifact. This repository builds that artifact from the official Google source code and publishes it as a GitHub Release.

## Automatic releases

The GitHub Actions workflow checks the official `GoogleCloudPlatform/cloud-sql-jdbc-socket-factory` releases once per hour at minute 17.

When a new upstream release is detected, it automatically:

1. Checks out the matching official upstream tag.
2. Builds the fat JAR with:

   ```bash
   mvn -P jar-with-dependencies clean package -DskipTests
   ```

3. Extracts the PostgreSQL `jar-with-dependencies` artifact.
4. Generates a SHA-256 checksum.
5. Publishes both files as a GitHub Release using the same version tag.

GitHub scheduled workflows can occasionally run late, so publication may not happen exactly at minute 17.

You can also run the workflow manually from **Actions** if you want to trigger the check immediately.

## Download

Open this repository's **Releases** page and download:

```text
postgres-socket-factory-<version>-jar-with-dependencies.jar
```

The matching `.sha256` file is published alongside it.

Then add the JAR to DataGrip under the PostgreSQL driver's **Driver Files / Additional Files**.

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

## Supply-chain note

The build job checks out the official Google upstream tag with persisted Git credentials disabled. The job that executes upstream Maven code only has read-only repository permissions. Publishing is done in a separate job with `contents: write` permission after the build artifact has been produced.

## Source

This repository does not modify the Cloud SQL Java Connector source tree. The workflow builds directly from a released upstream tag and links back to the corresponding official release in each generated GitHub Release.
