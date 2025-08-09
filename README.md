# Creating AWS resources with functions &amp; Introducing Arrays

Step-by-step process for the mini project *"Creating AWS resources with functions & Introducing Arrays"* — including the final script summary so it’s clear what we’ve learned and built.


## **Step-by-Step Process**

### **1. Project Objective**

The goal was to create a Bash script that provisions AWS resources (EC2 instances and S3 buckets) using:

* **Functions** (for modularity and reuse)
* **Arrays** (to handle multiple values like department names)
* **Environment variables** (for flexibility)
* **Basic error handling**

---

### **2. Set Up Environment Variables**

* Accept an environment (e.g., `local`, `testing`, `production`) as a script argument.
* Store it in `ENVIRONMENT=$1`.
* This allows running the same script for multiple environments without rewriting it.

---

### **3. Validate Arguments**

* Added `check_num_of_args` function to ensure exactly **one argument** (environment) is passed.
* If not, display usage instructions and exit.

---

### **4. Activate Environment**

* Wrote `activate_infra_environment` function to:

  * Check the value of `$ENVIRONMENT`.
  * Display which environment the script will run for.
  * Exit with an error if the environment is invalid.

---

### **5. AWS CLI & Profile Checks**

* **`check_aws_cli`**:

  * Verifies if AWS CLI is installed.
  * Exits if not found.
* **`check_aws_profile`**:

  * Ensures `$AWS_PROFILE` environment variable is set before proceeding.

---

### **6. Provision EC2 Instances (Function #1)**

* Function: `create_ec2_instances`
* Uses:

  * Instance type (`t2.micro`)
  * AMI ID (`ami-0cd59ecaf368e5ccf`)
  * Key pair name (`MyKeyPair`)
  * Region (`eu-west-2`)
* Calls `aws ec2 run-instances` with parameters.
* Checks if the command ran successfully and prints the result.

---

### **7. Create S3 Buckets with Arrays (Function #2)**

* Function: `create_s3_buckets`
* Defines:

  * `company="datawise"` (used as prefix)
  * Array `departments=("Marketing" "Sales" "HR" "Operations" "Media")`
* Loops through each department in the array and:

  * Builds a bucket name:

    ```
    datawise-<department>-Data-Bucket
    ```
  * Runs `aws s3api create-bucket` for each bucket.
  * Prints success/failure message.

---

### **8. Script Execution Flow**

At the end of the script:

```
check_num_of_args
activate_infra_environment
check_aws_cli
check_aws_profile
create_ec2_instances
create_s3_buckets
```

* Runs the functions in the right order to avoid failures mid-process.

---

### **9. Skills Learned in This Mini Project**

* **Bash Functions** — To structure scripts into reusable and logical blocks.
* **Arrays in Bash** — To handle multiple values and loop over them efficiently.
* **AWS CLI Commands** — For provisioning EC2 instances and creating S3 buckets.
* **Error Handling** — Using `$?` to check command exit status.
* **Environment Variables** — For flexible environment-based deployments.
* **Looping & String Interpolation** — For dynamic resource naming.

---

### **10. Complete Script (Clean Version)**

```bash
#!/bin/bash

# Environment variable from argument
ENVIRONMENT=$1

# Function to check number of arguments
check_num_of_args() {
    if [ "$#" -ne 1 ]; then
        echo "Usage: $0 <environment>"
        exit 1
    fi
}

# Function to activate environment
activate_infra_environment() {
    case "$ENVIRONMENT" in
        local)
            echo "Running script for Local Environment..."
            ;;
        testing)
            echo "Running script for Testing Environment..."
            ;;
        production)
            echo "Running script for Production Environment..."
            ;;
        *)
            echo "Invalid environment specified. Use 'local', 'testing', or 'production'."
            exit 2
            ;;
    esac
}

# Check if AWS CLI is installed
check_aws_cli() {
    if ! command -v aws &>/dev/null; then
        echo "AWS CLI is not installed. Please install it before proceeding."
        exit 3
    fi
}

# Check if AWS profile is set
check_aws_profile() {
    if [ -z "$AWS_PROFILE" ]; then
        echo "AWS profile environment variable is not set."
        exit 4
    fi
}

# Function to create EC2 instances
create_ec2_instances() {
    instance_type="t2.micro"
    ami_id="ami-0cd59ecaf368e5ccf"
    count=2
    region="eu-west-2"
    key_name="MyKeyPair"

    aws ec2 run-instances \
        --image-id "$ami_id" \
        --instance-type "$instance_type" \
        --count $count \
        --key-name "$key_name" \
        --region "$region"

    if [ $? -eq 0 ]; then
        echo "EC2 instances created successfully."
    else
        echo "Failed to create EC2 instances."
    fi
}

# Function to create S3 buckets using arrays
create_s3_buckets() {
    company="datawise"
    departments=("Marketing" "Sales" "HR" "Operations" "Media")
    region="eu-west-2"

    for department in "${departments[@]}"; do
        bucket_name="${company}-${department}-data-bucket"
        aws s3api create-bucket --bucket "$bucket_name" --region "$region"
        if [ $? -eq 0 ]; then
            echo "S3 bucket '$bucket_name' created successfully."
        else
            echo "Failed to create S3 bucket '$bucket_name'."
        fi
    done
}

# Execution order
check_num_of_args "$@"
activate_infra_environment
check_aws_cli
check_aws_profile
create_ec2_instances
create_s3_buckets
```
