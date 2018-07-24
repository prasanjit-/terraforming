# TERRAFORMING

- Generate Terraform Templates of an existing AWS Infrastructure

Terraforming Docker Image is available at servergurus/terraforming
https://hub.docker.com/r/servergurus/terraforming/

- Pull the Docker image:
```
$ docker pull servergurus/terraforming:latest
```

- And then run Terraforming as a Docker container:
```
$ docker run \
    --rm \
    --name terraforming \
    -e AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXXXXXXX \
    -e AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx \
    -e AWS_REGION=xx-yyyy-0 \
    servergurus/terraforming:latest \
    terraforming s3
```

- Usage
```
$ terraforming
```

```
Commands:
  terraforming asg             # AutoScaling Group
  terraforming dbpg            # Database Parameter Group
  terraforming dbsg            # Database Security Group
  terraforming dbsn            # Database Subnet Group
  terraforming ec2             # EC2
  terraforming ecc             # ElastiCache Cluster
  terraforming ecsn            # ElastiCache Subnet Group
  terraforming elb             # ELB
  terraforming iamg            # IAM Group
  terraforming iamgm           # IAM Group Membership
  terraforming iamgp           # IAM Group Policy
  terraforming iamip           # IAM Instance Profile
  terraforming iamp            # IAM Policy
  terraforming iamr            # IAM Role
  terraforming iamrp           # IAM Role Policy
  terraforming iamu            # IAM User
  terraforming iamup           # IAM User Policy
  terraforming nacl            # Network ACL
  terraforming r53r            # Route53 Record
  terraforming r53z            # Route53 Hosted Zone
  terraforming rds             # RDS
  terraforming rt              # Route Table
  terraforming rta             # Route Table Association
  terraforming s3              # S3
  terraforming sg              # Security Group
  terraforming sn              # Subnet
  terraforming vpc             # VPC
  ```
  
- Export tf

```
$ terraforming <resource> [--profile PROFILE]
(e.g. S3 buckets):
```
```
$ terraforming s3
```

```
resource "aws_s3_bucket" "hoge" {
    bucket = "hoge"
    acl    = "private"
}

resource "aws_s3_bucket" "fuga" {
    bucket = "fuga"
    acl    = "private"
}
```

- Export tfstate

```
$ terraforming <resource> --tfstate [--merge TFSTATE_PATH] [--overwrite] [--profile PROFILE]
(e.g. S3 buckets):
```
```$ terraforming s3 --tfstate```

```
{
  "version": 1,
  "serial": 1,
  "modules": {
    "path": [
      "root"
    ],
    "outputs": {
    },
    "resources": {
      "aws_s3_bucket.hoge": {
        "type": "aws_s3_bucket",
        "primary": {
          "id": "hoge",
          "attributes": {
            "acl": "private",
            "bucket": "hoge",
            "id": "hoge"
          }
        }
      },
      "aws_s3_bucket.fuga": {
        "type": "aws_s3_bucket",
        "primary": {
          "id": "fuga",
          "attributes": {
            "acl": "private",
            "bucket": "fuga",
            "id": "fuga"
          }
        }
      }
    }
  }
}
```
If you want to merge exported tfstate to existing terraform.tfstate, specify --tfstate --merge=/path/to/terraform.tfstate option. You can overwrite existing terraform.tfstate by specifying --overwrite option together.

- Existing terraform.tfstate:
```
# /path/to/terraform.tfstate

{
  "version": 1,
  "serial": 88,
  "remote": {
    "type": "s3",
    "config": {
      "bucket": "terraforming-tfstate",
      "key": "tf"
    }
  },
  "modules": {
    "path": [
      "root"
    ],
    "outputs": {
    },
    "resources": {
      "aws_elb.hogehoge": {
        "type": "aws_elb",
        "primary": {
          "id": "hogehoge",
          "attributes": {
            "availability_zones.#": "2",
            "connection_draining": "true",
            "connection_draining_timeout": "300",
            "cross_zone_load_balancing": "true",
            "dns_name": "hoge-12345678.ap-northeast-1.elb.amazonaws.com",
            "health_check.#": "1",
            "id": "hogehoge",
            "idle_timeout": "60",
            "instances.#": "1",
            "listener.#": "1",
            "name": "hoge",
            "security_groups.#": "2",
            "source_security_group": "default",
            "subnets.#": "2"
          }
        }
      }
    }
  }
}
```

- To generate merged tfstate:
```
$ terraforming s3 --tfstate --merge=/path/to/tfstate
```
```
{
  "version": 1,
  "serial": 89,
  "remote": {
    "type": "s3",
    "config": {
      "bucket": "terraforming-tfstate",
      "key": "tf"
    }
  },
  "modules": {
    "path": [
      "root"
    ],
    "outputs": {
    },
    "resources": {
      "aws_elb.hogehoge": {
        "type": "aws_elb",
        "primary": {
          "id": "hogehoge",
          "attributes": {
            "availability_zones.#": "2",
            "connection_draining": "true",
            "connection_draining_timeout": "300",
            "cross_zone_load_balancing": "true",
            "dns_name": "hoge-12345678.ap-northeast-1.elb.amazonaws.com",
            "health_check.#": "1",
            "id": "hogehoge",
            "idle_timeout": "60",
            "instances.#": "1",
            "listener.#": "1",
            "name": "hoge",
            "security_groups.#": "2",
            "source_security_group": "default",
            "subnets.#": "2"
          }
        }
      },
      "aws_s3_bucket.hoge": {
        "type": "aws_s3_bucket",
        "primary": {
          "id": "hoge",
          "attributes": {
            "acl": "private",
            "bucket": "hoge",
            "id": "hoge"
          }
        }
      },
      "aws_s3_bucket.fuga": {
        "type": "aws_s3_bucket",
        "primary": {
          "id": "fuga",
          "attributes": {
            "acl": "private",
            "bucket": "fuga",
            "id": "fuga"
          }
        }
      }
    }
  }
}
```

After writing exported tf and tfstate to files, execute terraform plan and check the result. There should be no diff.
```
$ terraform plan
```
No changes. Infrastructure is up-to-date. This means that Terraform
could not detect any differences between your configuration and
the real physical resources that exist. As a result, Terraform
doesn't need to do anything.
