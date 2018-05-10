# Deploy an Azure Databricks Workspace with a Custom Virtual Network Address Range

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-databricks-workspace-with-custom-vnet-address%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-databricks-workspace-with-custom-vnet-address%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

This template allows you to create a Azure Databricks workspace with a custom virtual network address range.
For more information, see the <a href="https://docs.microsoft.com/en-us/azure/azure-databricks/">Azure Databricks Documentation</a>.


# Testing Connectivity 

After peering the VPCs, you can test network connectivity from a cluster to the metastore VPC by running the following command inside a notebook:

```
%sh
nc -vz ${DNS name or private IP} ${port}
```

## Global init script

https://docs.databricks.com/user-guide/clusters/init-scripts.html#global-init-script

## External Hive Metastore

https://docs.databricks.com/user-guide/advanced/external-hive-metastore.html

## External Hive Metastore Config - MySql

```
# JDBC connect string for a JDBC metastore
javax.jdo.option.ConnectionURL jdbc:mysql://${mysql-host}:${mysql-port}/${metastore-db}

# Username to use against metastore database
javax.jdo.option.ConnectionUserName ${mysql-username}

# Password to use against metastore database
javax.jdo.option.ConnectionPassword ${mysql-password}

# Driver class name for a JDBC metastore (Runtime 3.4 and later)
javax.jdo.option.ConnectionDriverName org.mariadb.jdbc.Driver

# Driver class name for a JDBC metastore (prior to Runtime 3.4)
# javax.jdo.option.ConnectionDriverName com.mysql.jdbc.Driver
```

```
# Hive specific configuration options.
# spark.hadoop prefix is added to make sure these Hive specific options will propagate to the metastore client.
spark.hadoop.javax.jdo.option.ConnectionURL jdbc:mysql://${mysql-host}:${mysql-port}/${metastore-db}

# Driver class name for a JDBC metastore (Runtime 3.4 and later)
javax.jdo.option.ConnectionDriverName org.mariadb.jdbc.Driver

# Driver class name for a JDBC metastore (prior to Runtime 3.4)
# javax.jdo.option.ConnectionDriverName com.mysql.jdbc.Driver

spark.hadoop.javax.jdo.option.ConnectionUserName ${mysql-username}
spark.hadoop.javax.jdo.option.ConnectionPassword ${mysql-password}

# Spark specific configuration options
spark.sql.hive.metastore.version ${hive-version}
# Skip this one if ${hive-version} is 0.13.x.
spark.sql.hive.metastore.jars ${hive-jar-source}

# If any of your tables or databases use s3 as the file system scheme,
# uncomment the next line to set the s3:// URL scheme to the S3A file system.
# spark.hadoop prefix is added to make sure these file system options
# propagate to the metastore client and Hadoop configuration.
# spark.hadoop.fs.s3.impl com.databricks.s3a.S3AFileSystem

# If you need to use AssumeRole, uncomment the following settings.
# spark.hadoop.fs.s3a.impl com.databricks.s3a.S3AFileSystem
# spark.hadoop.fs.s3n.impl com.databricks.s3a.S3AFileSystem
# spark.hadoop.fs.s3a.credentialsType AssumeRole
# spark.hadoop.fs.s3a.stsAssumeRole.arn ${sts-arn}
```

```
# Hive specific configuration option
# spark.hadoop prefix is added to make sure these Hive specific options will propagate to the metastore client.
spark.hadoop.hive.metastore.uris thrift://${metastore-host}:${metastore-port}

# Spark specific configuration options
spark.sql.hive.metastore.version ${hive-version}
# Skip this one if ${hive-version} is 0.13.x.
spark.sql.hive.metastore.jars ${hive-jar-source}

# If any of your tables or databases use s3 as the file system scheme,
# uncomment the next line to set the s3:// URL scheme to the S3A file system.
# spark.hadoop prefix is added to make sure these file system options
# propagate to the metastore client and Hadoop configuration.
# spark.hadoop.fs.s3.impl com.databricks.s3a.S3AFileSystem

# If you need to use AssumeRole, uncomment the following settings.
# spark.hadoop.fs.s3a.impl com.databricks.s3a.S3AFileSystem
# spark.hadoop.fs.s3n.impl com.databricks.s3a.S3AFileSystem
# spark.hadoop.fs.s3a.credentialsType AssumeRole
# spark.hadoop.fs.s3a.stsAssumeRole.arn ${sts-arn}
```

External Metastore Setup - Notebook

```
%scala

dbutils.fs.put(
    "/databricks/init/${cluster-name}/external-metastore.sh",
    """#!/bin/sh
      |# Loads environment variables to determine the correct JDBC driver to use.
      |source /etc/environment
      |# Quoting the label (i.e. EOF) with single quotes to disable variable interpolation.
      |cat << 'EOF' > /databricks/driver/conf/00-custom-spark.conf
      |[driver] {
      |    # Hive specific configuration options for metastores in the local mode.
      |    # spark.hadoop prefix is added to make sure these Hive specific options will propagate to the metastore client.
      |    "spark.hadoop.javax.jdo.option.ConnectionURL" = "jdbc:mysql://${mysql-host}:${mysql-port}/${metastore-db}"
      |    "spark.hadoop.javax.jdo.option.ConnectionUserName" = "${mysql-username}"
      |    "spark.hadoop.javax.jdo.option.ConnectionPassword" = "${mysql-password}"
      |
      |    # Spark specific configuration options
      |    "spark.sql.hive.metastore.version" = "${hive-version}"
      |    # Skip this one if ${hive-version} is 0.13.x.
      |    "spark.sql.hive.metastore.jars" = "${hive-jar-source}"
      |
      |    # If any of your table or database use s3 as the file system scheme,
      |    # uncomment the next line to set the s3:// URL scheme to S3A file system.
      |    # spark.hadoop prefix is added to make sure these file system options will
      |    # propagate to the metastore client and Hadoop configuration.
      |    # "spark.hadoop.fs.s3.impl" = "com.databricks.s3a.S3AFileSystem"
      |
      |    # If you need to use AssumeRole, uncomment the following settings.
      |    # "spark.hadoop.fs.s3a.impl" = "com.databricks.s3a.S3AFileSystem"
      |    # "spark.hadoop.fs.s3n.impl" = "com.databricks.s3a.S3AFileSystem"
      |    # "spark.hadoop.fs.s3a.credentialsType" = "AssumeRole"
      |    # "spark.hadoop.fs.s3a.stsAssumeRole.arn" = "${sts-arn}"
      |EOF
      |
      |case "$DATABRICKS_RUNTIME_VERSION" in
      |  "")
      |     DRIVER="com.mysql.jdbc.Driver"
      |     ;;
      |  *)
      |     DRIVER="org.mariadb.jdbc.Driver"
      |     ;;
      |esac
      |# Add the JDBC driver separately since must use variable expansion to choose the correct
      |# driver version.
      |cat << EOF >> /databricks/driver/conf/00-custom-spark.conf
      |    "spark.hadoop.javax.jdo.option.ConnectionDriverName" = "$DRIVER"
      |}
      |EOF
      |""".stripMargin,
    overwrite = true
)
```

Setup Remote External Metastore - Notebook 
```
%scala

dbutils.fs.put(
    "/databricks/init/${cluster-name}/external-metastore.sh",
    """#!/bin/sh
      |
      |# Quoting the label (i.e. EOF) with single quotes to disable variable interpolation.
      |cat << 'EOF' > /databricks/driver/conf/00-custom-spark.conf
      |[driver] {
      |    # Hive specific configuration options for metastores in the remote mode.
      |    # spark.hadoop prefix is added to make sure these Hive specific options will propagate to the metastore client.
      |    "spark.hadoop.hive.metastore.uris" = "thrift://${metastore-host}:${metastore-port}"
      |
      |    # Spark specific configuration options
      |    "spark.sql.hive.metastore.version" = "${hive-version}"
      |    # Skip this one if ${hive-version} is 0.13.x.
      |    "spark.sql.hive.metastore.jars" = "${hive-jar-source}"
      |
      |    # If any of your table or database use s3 as the file system scheme,
      |    # uncomment the next line to set the s3:// URL scheme to S3A file system.
      |    # spark.hadoop prefix is added to make sure these file system options will
      |    # propagate to the metastore client and Hadoop configuration.
      |    # "spark.hadoop.fs.s3.impl" = "com.databricks.s3a.S3AFileSystem"
      |
      |    # If you need to use AssumeRole, uncomment the following settings.
      |    # "spark.hadoop.fs.s3a.impl" = "com.databricks.s3a.S3AFileSystem"
      |    # "spark.hadoop.fs.s3n.impl" = "com.databricks.s3a.S3AFileSystem"
      |    # "spark.hadoop.fs.s3a.credentialsType" = "AssumeRole"
      |    # "spark.hadoop.fs.s3a.stsAssumeRole.arn" = "${sts-arn}"
      |}
      |EOF
      |""".stripMargin,
    overwrite = true
)
```
