# Amazon Kinesis to S3 log transfer Terraform module

Terraform module and Lambda for saving JSON log records from Kinesis Data Streams to S3.

![terraform v0.11.x](https://img.shields.io/badge/terraform-v0.11.x-brightgreen.svg)

## Prerequisites
1. Records in Kinesis stream must be valid JSON data. Non-JSON data will be saved with `unknown` prefix.
    1. gzipped JSON, [CloudWatch Logs subscription filters log format](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/SubscriptionFilters.html) are supported.
    2. Logs without either of necessary keys listed below will be saved as `unknown` as well.
2. JSON data must have the following keys (key names are modifiable via variables):
    1. `log_type`: Log type identifier. Log data will be saved by this key: `%log_type%/YYYY-MM/DD/`. 
    2. `log_id`: Any unique identifier. Used to avoid file overwrites on S3. Also is useful to search for a specific log record.
    3. `time`: Any timestamp supported by [dateutil.parser.parse](https://dateutil.readthedocs.io/en/stable/parser.html#dateutil.parser.parse). ISO8601 with milli/microseconds recommended.

## Usage
```HCL
resource "aws_kinesis_stream" "stream" {
  name             = "stream"
  shard_count      = "1"
  retention_period = "24"
}

module "kinesis_to_s3" {
  source  = "baikonur-oss/lambda-kinesis-to-s3/aws"

  lambda_package_url    = "https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-s3/releases/download/v1.0.0/lambda_package.zip"
  name                  = "kinesis_to_s3"

  kinesis_stream_arn = "${aws_kinesis_stream.stream.arn}"
  batch_size = "100"
  log_bucket         = "example-bucket"
  log_path_prefix    = "foo/bar"
}

```

Warning: use same module and package versions!

### Version pinning
Make sure to use `?ref=` version pinning in module source URI.
This will save you from a lot of troubles.

For more information on module version pinning, see [Selecting a Revision](https://www.terraform.io/docs/modules/sources.html#selecting-a-revision) section of Terraform Modules documentation.


<!-- Documentation below is generated by pre-commit, do not overwrite manually -->
<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| handler | Lambda Function handler (entrypoint) | string | `"main.handler"` | no |
| kinesis\_stream\_arn | Source Kinesis Data Streams stream name | string | n/a | yes |
| lambda\_package\_url | Lambda package URL (see Usage in README) | string | n/a | yes |
| log\_bucket | Target S3 bucket to save data to | string | n/a | yes |
| log\_id\_field | Key name for unique log ID | string | `"log_id"` | no |
| log\_path\_prefix | Log file path prefix | string | n/a | yes |
| log\_timestamp\_field | Key name for log timestamp | string | `"time"` | no |
| log\_type\_field | Key name for log type | string | `"log_type"` | no |
| max\_batch\_size | Maximum number of records passed for a single Lambda invocation | string | n/a | yes |
| memory | Lambda Function memory in megabytes | string | `"256"` | no |
| name | Resource name | string | n/a | yes |
| runtime | Lambda Function runtime | string | `"python3.6"` | no |
| starting\_position | Kinesis ShardIterator type (see: https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html ) | string | `"TRIM_HORIZON"` | no |
| tags | Tags for Lambda Function | map | `<map>` | no |
| timeout | Lambda Function timeout in seconds | string | `"60"` | no |
| timezone | tz database timezone name (e.g. Asia/Tokyo) | string | `"UTC"` | no |
| tracing\_mode | X-Ray tracing mode (see: https://docs.aws.amazon.com/lambda/latest/dg/API_TracingConfig.html ) | string | `"PassThrough"` | no |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Contributing

Make sure to have following tools installed:
- [Terraform](https://www.terraform.io/)
- [terraform-docs](https://github.com/segmentio/terraform-docs)
- [pre-commit](https://pre-commit.com/)

### macOS
```bash
brew install pre-commit terraform terraform-docs

# set up pre-commit hooks by running below command in repository root
pre-commit install
```
