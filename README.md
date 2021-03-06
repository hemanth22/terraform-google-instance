# Google Compute Instance Terraform Module

Terraform module which creates a Google compute instance within
the default networking of an existing project.

## Features

Deploys a Google compute instance to the existing networking of
an existing Google cloud project.

## Usage

Call it as a module and deploy the instance.

```hcl
module "terraform-google-instance" {
  source = "git@github.com:newcontext-oss/terraform-google-instance.git"
}
```

## Requirements

Terraform version must be less than 0.12. (See below for tips on migrating to 0.12)

## Development

Feel free to submit pull requests to make changes to the module.

To begin developing on this module please have a Google Compute Project.

### Required Setup
- See the script bin/example-setup-ubuntu.sh for the complete setup. However, the steps are listed below.

#### Install Terraform (options below)

- [https://github.com/kamatama41/tfenv](https://github.com/kamatama41/tfenv)
- brew install terraform
- [https://www.terraform.io/downloads.html](https://www.terraform.io/downloads.html)

#### Install Ruby (options below)

- [https://github.com/rbenv/rbenv](https://github.com/rbenv/rbenv)
- brew install ruby # or other package managers
- [http://ruby-lang.org/](http://ruby-lang.org/)

#### Install JQ

- brew install jq # or other package managers
- [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)

#### Google IAM Console

Download a credentials JSON file from a user with proper permissions.
[https://console.cloud.google.com/iam-admin/iam](https://console.cloud.google.com/iam-admin/iam)

Save the file to the root of the repository directory called: `credentials.json`

#### Install gcloud CLI

- [https://cloud.google.com/sdk/gcloud/](https://cloud.google.com/sdk/gcloud/)

Set up the gcloud command line client:

```sh
gcloud auth activate-service-account --key-file credentials.json
gcloud config set project $(jq -r '.project_id' credentials.json)
gcloud config set compute/zone us-west1-a
```

#### Install Kitchen-Terraform and many other required Ruby Gems. 

```sh
gem install bundler --no-rdoc --no-ri
bundle install
```

#### Create an environment variables file

Create a file in the repository directory called: `.env`
It will have environment variables that Terraform uses to run.

```sh
cat > .env <<HEREDOC
export GOOGLE_APPLICATION_CREDENTIALS="$(pwd)/credentials.json"
export GCLOUD_PROJECT=$(jq -r '.project_id' $GOOGLE_APPLICATION_CREDENTIALS)
export GCLOUD_REGION="us-west1"
export TF_VAR_gcloud_project=$GCLOUD_PROJECT
my_public_ip=\$(dig +short myip.opendns.com @resolver1.opendns.com)
export TF_VAR_engineer_cidrs="[\"\$my_public_ip/32\"]"
export TF_VAR_ssh_key="$(pwd)/ubuntu.pub"
HEREDOC

```

#### Run Terraform and Tests

Common setup to be run once before any number of the rest of the following:

```sh
source .env
yes | ssh-keygen -f ubuntu -N '' >/dev/null
```

To run Terraform via Test-Kitchen:

```sh
bundle exec kitchen converge
```

Test-Kitchen will run the module code that is called via this file:
`test/fixtures/tf_module/main.tf`

To run InSpec via Test-Kitchen:

```sh
bundle exec kitchen verify
```

Test-Kitchen will run the InSpec controls via this file:
`test/integration/kt_suite/controls/default.rb`

To destroy everything via Test-Kitchen:

```sh
bundle exec kitchen destroy
```

## Migration to Terraform 0.12+

This repository does not support Terraform 0.12+ out of the box. 
Here are **some** of the things necessary to migrate.
1. Edit Gemfile to change version requirements to be this:
   - gem 'inspec', '~> 4.0'
   - gem 'kitchen-terraform', '>= 4.0.0'
1. Testing (verify stage) does not pass - there seem to be problems with the inspec plugins. 

## Authors

Module managed by [Nick Willever](https://github.com/nictrix).

## License

Apache 2 Licensed. See LICENSE for full details.
