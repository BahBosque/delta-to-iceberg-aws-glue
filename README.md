# Delta to Iceberg Migration Tool

A Java-based tool for migrating Delta Lake tables to Apache Iceberg format using AWS Glue Catalog for metadata management and S3 for storage.

## Overview

This tool helps you migrate existing Delta Lake tables stored in S3 to Apache Iceberg format while maintaining all data and metadata through AWS Glue Catalog. It's designed to handle large-scale migrations with batch processing capabilities using CSV configuration files.

## Features

- **Batch Migration**: Process multiple tables defined in CSV files
- **AWS Glue Integration**: Automatic table registration in Glue Catalog
- **S3 Storage**: Seamless migration between S3 locations
- **Data Validation**: Count verification before and after migration
- **Atomic Operations**: Safe migration with temporary tables
- **Logging**: Comprehensive logging for monitoring and debugging

## Prerequisites

- Java 11+
- Apache Spark 3.4+
- AWS CLI configured with appropriate permissions
- Access to AWS S3 and Glue services

### Required AWS Permissions

Your AWS credentials need permissions for:
- S3: Read/Write/Delete on source and target buckets
- Glue: Create/Read/Update/Delete tables and databases
- CloudWatch: Logs (optional, for monitoring)

## Quick Start

### 1. Clone and Build

```bash
git clone <repository-url>
cd migracao_delta_iceberg
mvn clean package
```

### 2. Set Environment Variables

```bash
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_REGION=us-east-1
export AWS_ACCOUNT_ID=123456789012
```

### 3. Prepare Your Migration File

Create a CSV file with your table mappings (see `sample_models.csv`):

```csv
table_name,partition_column
customer_data,created_date
order_history,order_date
product_catalog,last_updated
```

### 4. Run Migration

```bash
java -cp target/migracao_delta_iceberg-1.0-SNAPSHOT.jar com.example.app.DeltaToIcebergMigration
```

## Configuration

### CSV File Format

The migration tool reads table configurations from CSV files with the following format:

| Column | Description | Example |
|--------|-------------|---------|
| `table_name` | Name of the Delta table to migrate | `customer_data` |
| `partition_column` | Column used for partitioning (optional) | `created_date` |

### Spark Configuration

The tool automatically configures Spark with optimized settings for Delta/Iceberg operations:

- Memory allocation: 10GB driver memory
- Catalog integration: Both Delta and Iceberg catalogs
- S3 optimization: Vectorized reading disabled for compatibility
- Date handling: Corrected rebase mode for date/timestamp columns

## Usage Examples

### Basic Migration

```bash
# Migrate all tables defined in models.csv
java -cp target/migracao_delta_iceberg-1.0-SNAPSHOT.jar com.example.app.DeltaToIcebergMigration
```

### List Existing Tables

Use the included utility to inspect your Glue Catalog tables:

```bash
java -cp target/migracao_delta_iceberg-1.0-SNAPSHOT.jar com.example.app.ListGlueTablesDetailed
```

This utility helps you:
- Verify successful migrations
- Check table metadata
- Confirm Iceberg table format
- Inspect storage locations and parameters

## Architecture

### Migration Process

1. **Read Configuration**: Load table definitions from CSV
2. **Delta Reading**: Read existing Delta table from S3
3. **Temporary Creation**: Create temporary Iceberg table
4. **Data Transfer**: Copy data to Iceberg format
5. **Original Cleanup**: Remove original Delta files
6. **Final Creation**: Create final Iceberg table
7. **Validation**: Verify record counts match

### Directory Structure

```
migracao_delta_iceberg/
├── src/main/java/com/example/app/
│   ├── DeltaToIcebergMigration.java    # Main migration logic
│   ├── ListGlueTablesDetailed.java     # Table inspection utility
│   └── util/
│       └── Logger.java                 # Custom logging utility
├── src/main/resources/
│   └── META-INF/services/
│       └── org.apache.iceberg.catalog.CatalogPlugin
├── sample_models.csv                   # Example configuration
├── pom.xml                            # Maven dependencies
└── README.md                          # This file
```

## Advanced Configuration

### Custom S3 Paths

The tool uses environment variables to construct S3 paths:

```bash
# Source bucket pattern
s3://data-lake-snapshot-{AWS_ACCOUNT_ID}-{AWS_REGION}

# Target bucket (same as source by default)
s3://data-lake-snapshot-{AWS_ACCOUNT_ID}-{AWS_REGION}
```

### Glue Database Settings

Default configuration:
- **Catalog**: `iceberg`
- **Database**: `data_lake_snapshot`

Modify these in the source code if needed for your environment.

### Memory Tuning

For large tables, adjust Spark memory settings in the code:

```java
.config("spark.driver.memory", "20g")           // Increase for large schemas
.config("spark.executor.memory", "8g")          // Add if using cluster mode
.config("spark.sql.adaptive.enabled", "true")   // Enable adaptive query execution
```

## Monitoring and Troubleshooting

### Logging

The tool provides detailed logging for each migration step:

- **INFO**: Progress updates and success messages
- **WARN**: Non-critical issues and warnings
- **ERROR**: Migration failures with stack traces
- **DEBUG**: Detailed operation logs (enable as needed)

### Common Issues

1. **Permission Errors**: Verify AWS credentials and policies
2. **Memory Issues**: Increase driver memory for large tables
3. **Schema Conflicts**: Check for incompatible data types
4. **Network Timeouts**: Configure appropriate timeout values

### Validation

After migration, verify your tables:

```bash
# Check table exists in Glue
aws glue get-table --database-name data_lake_snapshot --name your_table_name

# Verify Iceberg format
java -cp target/migracao_delta_iceberg-1.0-SNAPSHOT.jar com.example.app.ListGlueTablesDetailed
```

## Performance Tips

1. **Batch Size**: Process tables in smaller batches for better error handling
2. **Partitioning**: Ensure proper partition column selection for query performance
3. **Resource Allocation**: Monitor memory usage and adjust as needed
4. **Network**: Use same AWS region for all resources

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is open source and available under the MIT License.

## Support

For issues and questions:
1. Check existing issues in the GitHub repository
2. Create a new issue with detailed information
3. Include logs and configuration details

---

**Note**: This tool modifies S3 data by deleting original Delta files. Always backup your data before running migrations in production environments.