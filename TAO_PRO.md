# TAO Pro

## Notes

- Terraform Tests??
- Manual approval to deploy on prod account
	- It makes sense, but you have nonprod to test the deployment, right?
- `-input=false`
	- Great to avoid command to asking for inputs in CI/CD
	- Add it on local aliases and CI/CD commands
- CLI docs >>> Browser docs
- "Options" and "flags" are the same
- Plugin cache directory for local laptop
- Add to the version control system the .terraform.lock.hcl file
- No connectivity?
	1. Download locally the plugins
		- `terraform providers mirror <YOUR_LOCAL_PATH>`
		- It downloads all the providers defined in your TF files
	2. Add provider_installation TF block in $HOME/.terraformrc file
		```hcl
				provider_installation {
				  # filesystem_mirror
				  filesystem_mirror {
				    path    = "/usr/share/terraform/providers"
				    include = ["example.com/*/*"]
				  }
				  # Direct download form Internet
				  direct {
				    exclude = ["example.com/*/*"]
				  }
				}
		```
- Checkov
	- Code scanning
		- Check specific "checks"
			- `checkov -f/-d <FILE>/<PATH> --check <CHECK_CODE>`
			- It supports wildcards (\*)
		- Skip check
			- `--skip-check`
			- In code, using comment `#checkov:skip=<CHECK_ID>:<Sign-off / Justification>`
	- Plan scanning?
		- Pass the plan file as an argument to `checkov -f` command

- Terraform plan in a file?
	- Ensuring consistency at apply time.
		- You just apply the content of the plan file, even if you modify your TF code
	- Plan file is a binary
		- You need to use "terraform show" command to see the content
		- Or use -no-color and save the terraform plan output directly to a file (>)

- Terraform import (terraform import <source> <destination>)
	- Process
		1. Use import block in a tf file (e.g. import.tf)
		2. Then, execute "terraform plan -generate-config-out=<FILE>"" (Terraform 1.5 and above)
		3. `terraform apply`
	- Provider region **MUST** match ther resource region

- Resource Targeting
	- Use `-target` flag in `terraform plan`
	- You cannot targeting multiple resources at a time
		- Just one or all of them
	- If you target a resource that depends on another, you will target both of them indirectly

- Random number - Generate "unique" identifiers for resources names/IDs
	```hcl
	  resource "random_integer" "this" {
		min = 1
		max = 100
	  }
	```

- Moved blocks (terraforom state mv <source> <destination>)
	- Process
		1. Rename the original resources
		2. Add a `moved` block
			```hcl
			  moved {
			  	from = original_resource_name
			  	to = current_resource_name
			  }
			```

- Terraform state commands
	- `list` - list all Terraform-managed resources
	- `show <resource_name>` - get resource HCL definition
	- `pull` - retrieve state file
	- `rm <resource_name>` - remove a resource from Terraform state
	- `mv <source_resource> <destination_resource>` - move a resource to a different address without recreation

- Terraform Remote State data source
	- https://registry.terraform.io/providers/hashicorp/terraform/latest/docs/data-sources/remote_state
		```hcl
			data "terraform_remote_state" "s3" {
			  backend = "s3"
			  config = {
			    bucket = ""
			    key = ""
			    region = ""
			  }
			}
		```

- Terraform test to verify at plan and apply stage if you want
	- `.tftest.json` or `.tftest.hcl` file extension in a tests/ folder 
		```hcl
			provider "aws" {
				region = "ap-southeast-1"
			}

			variables {
				x = "y"
			}


			run "name" {
				command = <plan|apply>

				# Similar to variables validation
				assert {
					condition =
					error_message = ""
				}
			}
		```
		- By default, terraform test will apply resources
		- Provider configuration to modify terraform test apply region
			- Same definition as the terraform configuration
		- You can define variables values in the `variables` block
			- If not, it'll use the default provider configuration
			- Highest preferences to choose

- Terraform Modules
	- Avoid hardcoded values. Use variables instead.
	- It should include `terraform.required_providers` to set the providers supported versions

## Examp Tips

- Backups of the scenarios
- Configure `provider "aws" {region  = ""	access_key = ""	secret_access_key = ""}` for all scenarios

## HCP Terrafom / Terraform Enterprise (tfe)

- Security Team wants to attach Sentinel Policy in HCP Terraform for all the Developer Team. What is the attachment scope for Policy sets?
	- **Policies enforced globally**: HCP Terraform automatically enforces this global policy set on all of an organization's existing and future workspaces.
	- **Policies enforced on selected projects and workspaces**: Use the text fields to find and select the workspaces and projects to enforce this policy set on. This affects all current and future workspaces for any chosen projects.

- You want to verify if specific HTTP endpoint is working during Terraform infrastructure deployment, however you don't want to block the deployment if HTTP data source fails. What is the way to achieve this?
	- Define the http data source inside the check blocks
	- https://developer.hashicorp.com/terraform/tutorials/configuration-language/checks#use-a-data-source-within-a-check
