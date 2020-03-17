# ibm-spectrum-scale-cloud-install

This repository contains terraform templates to provision public cloud (AWS, Azure) resources needed for IBM Spectrum
Scale cloud deployment.

- [Getting Started](#Getting-Started)
    + [Prerequisites](#Prerequisites)
    + [Usage](#Usage)
        + [AWS - New VPC Template](#AWS-New-VPC-Template)
        + [AWS - Existing VPC Template](#AWS-Existing-VPC-Template)
        + [Azure - New VNet Template](#Azure-New-VNet-Template)
        + [Azure - Existing VNet Template](#Azure-Existing-VNet-Template)
- [Documentation](docs/)
   * Template Parameters
     + [AWS - New VPC](docs/aws_new_vpc/README.md)
     + [AWS - Existing VPC](docs/aws_existing_vpc/README.md)
     + [Azure - New VNet](docs/azure_new_vnet/README.md)
     + [Azure - Existing VNet](docs/azure_existing_vnet/README.md)
- [Warnings](#Warnings)
- [Reporting Issues and Feedback](#Reporting-Issues-and-Feedback)
- [Disclaimer](#Disclaimer)
- [Contributing](#Contribute-Code)

## Getting Started

These instructions will help you provision resources needed for IBM Spectrum Scale deployment on cloud from your machine (on-premise node or a node in cloud).

### Prerequisites

1. Install Terraform
    ```
    $ yum install -y wget unzip
    $ wget https://releases.hashicorp.com/terraform/0.12.23/terraform_0.12.23_linux_amd64.zip
    $ sudo unzip ./terraform_0.12.23_linux_amd64.zip -d /usr/local/bin/
    ```
    For detailed install procedure, refer to [Installing Terraform](https://learn.hashicorp.com/terraform/getting-started/install.html).

    To verify installed terraform version
    ```
    $ terraform -v
    ```
    For detailed documentation related to CLI options, refer to [Terraform Commands](https://www.terraform.io/docs/commands/index.html).

2. Install Ansible
    ```
    $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    $ python get-pip.py --user
    $ pip install --user ansible
    ```
    In case your machine has default path as `/bin/ansible-vault`, create a soft link to `/usr/local/bin/ansible-vault`. 
    ```
    ln -s /bin/ansible-vault /usr/local/bin/ansible-vault
    ```
    For detailed install procedure, refer to [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

    | Note: The templates require ```ansible-vault``` for storing ssh keys (which will be auto-generated by the templates to enable password-less ssh setup between the provisioned nodes). |
    | --- |
 
3. Create a keyring file with desired passphrase
   (This will be used as parameter for `ansible-vault` option `--vault-password-file`).
    ```
    $ mkdir -p ~/tf_data_path/
    $ echo 'Spectrumscale!' >> ~/tf_data_path/keyring
    ```
    | Note: For security reasons, we recommend changing `Spectrumscale!` to your desired passphrase. The generated SSH key will be encrypted using the passphrase mentioned in keyring, keep this file safe for debug purpose. |
    | --- |

4. Configure desired cloud credentials

    **AWS**

    - Install AWS CLI (For more details, refer to [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)).
    - Create access keys for IAM user (For more details, refer to [Managing Access Keys for IAM Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)).
    - Configure AWS CLI on-premise machine (For more details, refer to [Quickly Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-quick-configuration)).

    **Azure**

    - Install Azure CLI (For more details, refer to [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)).
    - Login in Azure via `az login` command from on-premise machine (For more details, refer to [Sign in with Azure CLI](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest))   .

### Usage

Clone `ibm-spectrum-scale-cloud-install` repository to your on-premise machine.

    $ git clone https://github.com/IBM/ibm-spectrum-scale-cloud-install.git

Choose either of the deployment options (new or existing) that suits your need.

#### AWS-New VPC Template

The following steps will provision AWS resources (**new VPC, Bastion, compute and storage instances**) required for
IBM Spectrum Scale Cloud deployment.

1. Change working directory to `aws_new_vpc_scale/`.

    ```
    $ cd ibm-spectrum-scale-cloud-install/aws_scale_templates/aws_new_vpc_scale/
    ```

2. Create terraform variable definitions file (`aws_new_vpc_scale_inputs.auto.tfvars.json`) and provide infrastructure inputs.

   (Below is a sample. For details related to input parameters, refer to [AWS New VPC Template Input Parameters](docs/aws_new_vpc/README.md#inputs).)

    ```
    $ cat aws_new_vpc_scale_inputs.auto.tfvars.json
    {
        "region": "sa-east-1",
        "availability_zones": [
            "sa-east-1a",
            "sa-east-1c"
        ],
        "bastion_image_name": "Amazon-Linux-HVM",
        "key_name": "aws_key",
        "compute_ami_id": "ami-048b2348ac2ccfc53",
        "storage_ami_id": "ami-048b2348ac2ccfc53",
        "compute_instance_type": "t2.micro",
        "storage_instance_type": "t2.micro",
        "total_compute_instances": "2",
        "total_storage_instances": "2",
        "ebs_volume_size": "500",
        "ebs_volume_type": "gp2",
        "ebs_volumes_per_instance": 3,
        "operator_email": "operator@email.com"
    }
    ```
    | Note: In case of single availability zone, provide a single value for the `availability_zone` keyword. Ex: `"availability_zones"=["sa-east-1a"]` |
    | --- |

3. Run `terraform init` and `terraform apply -auto-approve` to provision resources.

#### AWS-Existing VPC Template

The following steps will provision AWS resources (**compute and storage instances in existing VPC**) required for
IBM Spectrum Scale Cloud deployment.

1. Change working directory to `instance_template/`.

    ```
    $ cd ibm-spectrum-scale-cloud-install/aws_scale_templates/sub_modules/instance_template/
    ```

2. Create terraform variable definitions file (`aws_scale_instances_inputs.auto.tfvars.json`) and provide infrastructure inputs.

   (Below is a sample. For details related to input parameters, refer to [AWS Existing VPC Template Input Parameters](docs/aws_existing_vpc/README.md#inputs).)
    ```
    $ cat aws_scale_instances_inputs.auto.tfvars.json
    {
        "region": "sa-east-1",
        "availability_zones": [
            "sa-east-1a",
            "sa-east-1c"
        ],
        "bastion_sec_group_id": "sg-05546f0e8c6ebd1ce",
        "private_instance_subnet_ids": ["subnet-03773a72bf9499420", "subnet-0bf0320b47f195f19"]
        "vpc_id": "vpc-04eef62f613ba98e0",
        "key_name": "aws_key",
        "compute_ami_id": "ami-048b2348ac2ccfc53",
        "storage_ami_id": "ami-048b2348ac2ccfc53",
        "compute_instance_type": "t2.micro",
        "storage_instance_type": "t2.micro",
        "ebs_volume_size": "10",
        "ebs_volume_type": "gp2",
        "ebs_volumes_per_instance": 3,
        "total_compute_instances": "2",
        "total_storage_instances": "2",
        "operator_email": "operator@email.com"
    }
    ```

    | Note: In case of single availability zone, provide a single value for the `availability_zone` keyword and one subnet to `private_instance_subnet_ids` keyword. Ex: `"availability_zones"=["sa-east-1a"], "private_instance_subnet_ids"=["subnet-03773a72bf9499420"]` |
    | --- |

3. Run `terraform init` and `terraform apply -auto-approve` to provision resources.

#### Azure-New VNet Template

The following steps will provision Azure resources (**new VNet, Bastion, compute and storage vm's**) required for
IBM Spectrum Scale Cloud deployment.

1. Change working directory to `azure_new_vnet_scale/`.

    ```
    $ cd ibm-spectrum-scale-cloud-install/azure_scale_templates/azure_new_vnet_scale/
    ```

2. Create terraform variable definitions file (`azure_new_vnet_scale_inputs.auto.tfvars.json`) and provide infrastructure inputs.

   (Below is a sample. For details related to input parameters, refer to [Azure New VNet Template Input Parameters](docs/azure_new_vnet/README.md#inputs).)
    ```
    $ cat azure_new_vnet_scale_inputs.auto.tfvars.json
    {
        "location": "eastus",
        "resource_group_name": "Spectrum-Scale-rg",
        "availability_zones": [
            1,
            2
        ],
        "vnet_name": "Spectrum-Scale-vnet",
        "vm_sshlogin_pubkey_path": "/Data/Code/id_rsa.pub",
        "total_compute_vms": "1",
        "compute_vm_os_publisher": "Canonical",
        "compute_vm_os_offer": "UbuntuServer",
        "compute_vm_os_sku": "16.04-LTS",
        "compute_vm_size": "Standard_D1_v2",
        "total_storage_vms": "2",
        "storage_vm_os_publisher": "Canonical",
        "storage_vm_os_offer": "UbuntuServer",
        "storage_vm_os_sku": "16.04-LTS",
        "storage_vm_size": "Standard_D1_v2",
        "total_disks_per_vm": "3",
        "data_disk_size": "1"
    }
    ```

    | Note: In case of single availability zone, provide a single value for the `availability_zone` keyword. Ex: `"availability_zones"=["1"]`|
    | --- |

3. Run `terraform init` and `terraform apply -auto-approve` to provision resources.

#### Azure-Existing VNet Template

The following steps will provision Azure resources (**compute and storage vm's in existing VNet**) required for
IBM Spectrum Scale Cloud deployment.

1. Change working directory to `vm_template/`.

    ```
    $ cd ibm-spectrum-scale-cloud-install/azure_scale_templates/sub_modules/vm_template/
    ```

2. Create terraform variable definitions file (`azure_scale_vms_inputs.auto.tfvars.json`) and provide infrastructure inputs.

   (Below is a sample. For details related to input parameters, refer to [Azure Existing VNet Template Input Parameters](docs/azure_existing_vnet/README.md#inputs).)

    ```
    $ cat azure_scale_vms_inputs.auto.tfvars.json
    {
        "location": "eastus",
        "resource_group_name": "Spectrum-Scale-rg",
        "availability_zones": [
            1,
            2
        ],
        "all_compute_nic_ids": ["/subscriptions/a1-05-4d-89-c9/resourceGroups/Spectrum-Scale-rg/providers/Microsoft.Network/networkInterfaces/spectrumscale-compute-nic-1"],
        "all_storage_nic_ids": ["/subscriptions/a1-05-4d-89-c9/resourceGroups/Spectrum-Scale-rg/providers/Microsoft.Network/networkInterfaces/spectrumscale-storage-nic-1",
                                "/subscriptions/a1-05-4d-89-c9/resourceGroups/Spectrum-Scale-rg/providers/Microsoft.Network/networkInterfaces/spectrumscale-storage-nic-2"],
        "private_zone_vnet_link_name": "private-snet",
        "vm_sshlogin_pubkey_path": "/Data/Code/id_rsa.pub",
        "total_compute_vms": "1",
        "compute_vm_os_publisher": "Canonical",
        "compute_vm_os_offer": "UbuntuServer",
        "compute_vm_os_sku": "16.04-LTS",
        "compute_vm_size": "Standard_D1_v2",
        "total_storage_vms": "2",
        "storage_vm_os_publisher": "Canonical",
        "storage_vm_os_offer": "UbuntuServer",
        "storage_vm_os_sku": "16.04-LTS",
        "storage_vm_size": "Standard_D1_v2",
        "total_disks_per_vm": "3",
        "data_disk_size": "1"
    }
    ```

    | Note: In case of single availability zone, provide a single value for the `availability_zone` keyword. Ex: `"availability_zones"=["1"]`|
    | --- |

3. Run `terraform init` and `terraform apply -auto-approve` to provision resources.

## Warnings

- Each `terraform apply` will generate a new SSH key, that causes replacement (destroy of old and creates new) of key dependent resources.

## Reporting Issues and Feedback

To file issues, suggestions, new features, etc., please open an [Issue](https://github.com/IBM/ibm-spectrum-scale-cloud-install/issues).

## Disclaimer

Please note: all templates / modules / resources in this repo are released for use "AS IS" without any warranties of
any kind, including, but not limited to their installation, use, or performance. We are not responsible for any damage
or charges or data loss incurred with their use. You are responsible for reviewing and testing any scripts you run
thoroughly before use in any production environment. This content is subject to change without notice.

## Contribute Code

We welcome contributions to this project, see [Contributing](CONTRIBUTING.md) for more details.
