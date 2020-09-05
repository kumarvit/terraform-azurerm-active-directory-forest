# Azure Virtual Machines with Active Directory Domain controller Terraform Module

This terraform module is designed to deploy azure Windows 2012R2/2016/2019 virtual machines with Public IP, Availability Set and Network Security Group support.

## Module Usage

```hcl
module "virtual-machine" {
  source = "github.com/tietoevry-infra-as-code/terraform-azurerm-active-directory-forest?ref=v1.0.0"

  # Resource Group, location, VNet and Subnet details
  resource_group_name  = "rg-tieto-internal-shared-westeurope-001"
  location             = "westeurope"
  virtual_network_name = "vnet-tieto-internal-shared-dev-westeurope-01"
  subnet_name          = "snet-management-shared-westeurope"

  # This module support multiple Pre-Defined Linux and Windows Distributions.
  # Windows Images: windows2012r2dc, windows2016dc, windows2019dc
  virtual_machine_name               = "vm-testdc"
  windows_distribution_name          = "windows2019dc"
  virtual_machine_size               = "Standard_A2_v2"
  admin_username                     = "tietoadmin"
  admin_password                     = "complex_password"
  private_ip_address_allocation_type = "Static"
  private_ip_address                 = ["10.1.2.4"]

  # Active Directory domain and netbios details
  # Intended for test/demo purposes
  # For production use of this module, fortify the security by adding correct nsg rules
  active_directory_domain       = "consoto.com"
  active_directory_netbios_name = "CONSOTO"

  # Network Seurity group port allow definitions for each Virtual Machine
  # NSG association to be added automatically for all network interfaces.
  # SSH port 22 and 3389 is exposed to the Internet recommended for only testing.
  # For production environments, we recommend using a VPN or private connection
  nsg_inbound_rules = [
    {
      name                   = "rdp"
      destination_port_range = "3389"
      source_address_prefix  = "*"
    },

    {
      name                   = "dns"
      destination_port_range = "53"
      source_address_prefix  = "*"
    },
  ]

  # Adding TAG's to your Azure resources (Required)
  # ProjectName and Env are already declared above, to use them here, create a varible.
  tags = {
    ProjectName  = "tieto-internal"
    Env          = "dev"
    Owner        = "user@example.com"
    BusinessUnit = "CORP"
    ServiceClass = "Gold"
  }
}
```

## Terraform Usage

To run this example you need to execute following Terraform commands

```hcl
terraform init
terraform plan
terraform apply
```

Run `terraform destroy` when you don't need these resources.

## Outputs

|Name | Description|
|---- | -----------|
`windows_vm_password`|Password for the windows Virtual Machine
`windows_vm_public_ips`|Public IP's map for the all windows Virtual Machines
`windows_vm_private_ips`|Public IP's map for the all windows Virtual Machines
`windows_virtual_machine_ids`|The resource id's of all Windows Virtual Machine
`network_security_group_ids`|List of Network security groups and ids
`vm_availability_set_id`|The resource ID of Virtual Machine availability set
`active_directory_domain`|The name of the active directory domain
`active_directory_netbios_name`|The name of the active directory netbios name
