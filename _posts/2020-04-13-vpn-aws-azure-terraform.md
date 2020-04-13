---
layout: post
title: "Terraform for a Highly Available VPN between AWS and Azure"
date: 2020-04-13 05:00:00
description: "Let's connect an AWS VPC with an Azure vNet using a highly available site-to-site VPN, writing all the bits and pieces in Terraform!"
image: "/img/clouds.jpg"
---

Hi there 👋 

Today we will go through a bunch of Terraform code to deploy a _highly available_ site-to-site VPN between AWS and Azure, aiming for a single **terraform apply** execution. In the end, we will have a tunnel between an AWS VPC and an Azure vNet, reaching resources from each cloud provider as if we were in the same local network. Think about `pinging` an AWS EC2 from an Azure Virtual Machine, without using any public IP.

## Considerations
A quick list of things to be aware of:

- This is not a comprehensive Terraform tutorial, but I'll try to explain as much as possible ;) the [Learn Terraform](https://learn.hashicorp.com/terraform) is all you need to follow along 
- For simplicity, I won't focus on Terraform remote state, modules or proper access credentials configuration, but they can all be implemented later
- The setup is _highly available_: enough resources will be created to avoid any disruption during a possible cloud provider maintenance
- A vanilla AWS and Azure account will be used to make sure we don't mess with already running environments

We will follow a step by step process to establish the site-to-site VPN and, later, _Terraform_ everything it with _only one_ command! 

## The Diagram ™️
 _A lot_ of resources are involved in setting up a VPN between the two cloud providers. The following ~~awfully draw~~ diagram will help you visualize how the components are connected. They also have _almost_ the same names from the Terraform code. If you feel a bit lost in the middle of the code, come back here and take a quick look 😉

![](/img/vpn-aws-azure.png)

## Step 1 – Laying Out Azure's Network
In the first step we will create the basic Azure infrastructure:
- One Resource Group
- One vNet, with the `10.0.0.0/16` network CIDR 
- A subnet for the virtual machines named **subnet_1** and a subnet for the VPN tunnel with a [mandatory](https://www.terraform.io/docs/providers/azurerm/r/virtual_network_gateway.html#subnet_id) name of **GatewaySubnet**


Create a file named `azure.tf` and add the following content. We will be appending more code in the next steps.
```terraform
provider "azurerm" {
  version = "2.1.0"
  features {}

  # IMPORTANT!
  # For simplicity, we are not setting up "proper" access through environment variables
  # Insert your access credentials here
  subscription_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  tenant_id       = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  client_secret   = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  client_id       = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}

# And not configuring a Terraform remote state
terraform {}

resource "azurerm_resource_group" "resource_group" {
  name     = "resource_group"
  location = "westeurope"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "vnet"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name
  address_space       = ["10.0.0.0/16"]
}

# The subnet where the Virtual Machine will live
resource "azurerm_subnet" "subnet_1" {
  name                 = "subnet_1"
  resource_group_name  = azurerm_resource_group.resource_group.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefix       = "10.0.1.0/24"
}

# The subnet where the VPN tunnel will live
resource "azurerm_subnet" "subnet_gateway" {
  # The name "GatewaySubnet" is mandatory
  # Only one "GatewaySubnet" is allowed per vNet
  name                 = "GatewaySubnet"
  resource_group_name  = azurerm_resource_group.resource_group.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefix       = "10.0.2.0/24"
}
```

Go for a `terraform init` and a `terraform apply`:
```bash
$ terraform init
... ok

$ terraform apply

...
Plan: 4 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: 
```

Enter `yes` and let it rip.

## Step 2 – Azure Virtual Gateway
The second step is about setting up a [Virtual Network Gateway](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways) in Azure. It's the component that understands VPN and exposes one or two public IP addresses for the connection with the other side of the tunnel (in our case, AWS). We will configure _two_ public IPs for high availability.

In your `azure.tf` file, add the following:
```terraform
resource "azurerm_public_ip" "public_ip_1" {
  name                = "virtual_network_gateway_public_ip_1"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  # Public IP needs to be dynamic for the Virtual Network Gateway
  # Keep in mind that the IP address will be "dynamically generated" after
  # being attached to the Virtual Network Gateway below
  allocation_method = "Dynamic"
}

resource "azurerm_public_ip" "public_ip_2" {
  name                = "virtual_network_gateway_public_ip_2"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  # Public IP needs to be dynamic for the Virtual Network Gateway
  allocation_method = "Dynamic"
}

resource "azurerm_virtual_network_gateway" "virtual_network_gateway" {
  name                = "virtual_network_gateway"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  type     = "Vpn"
  vpn_type = "RouteBased"

  # Configuration for high availability
  active_active = true
  # This might me expensive, check the prices  
  sku           = "VpnGw1"

  # Configuring the two previously created public IP Addresses
  ip_configuration {
    name                          = azurerm_public_ip.public_ip_1.name
    public_ip_address_id          = azurerm_public_ip.public_ip_1.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.subnet_gateway.id
  }

  ip_configuration {
    name                          = azurerm_public_ip.public_ip_2.name
    public_ip_address_id          = azurerm_public_ip.public_ip_2.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.subnet_gateway.id
  }
}
```

Apply your changes:
```bash
$ terraform apply
...


Plan: 3 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

**Buckle up: it takes up to 30 MINUTES to create a Virtual Network Gateway**. No kidding:
```bash 
azurerm_virtual_network_gateway.virtual_network_gateway: Still creating... [24m51s elapsed]
azurerm_virtual_network_gateway.virtual_network_gateway: Creation complete after 24m55s [id=/subscriptions/xxx/resourceGroups/resource_group/providers/Microsoft.Network/virtualNetworkGateways/virtual_network_gateway]
```

## Step 3 – AWS Basics
Let's switch to AWS for a bit. Let's create a basic AWS network infrastructure from scratch, including:
- VPC
- Subnet
- Internet Gateway
- Route Table & Route

> We are not using the default AWS network infrastructure to avoid changing anything already deployed ;)

Create a new file named `aws.tf` and insert the code below:
```terraform
provider "aws" {
  version    = "2.55.0"
  # IMPORTANT!
  # Setup your correct region, access_key and secret_key
  # Again, we are not focusing on credentials best practices here
  region     = "eu-west-1"
  access_key = ""
  secret_key = ""
}

resource "aws_vpc" "vpc" {
  cidr_block = "192.168.0.0/16"

  tags = {
    Name = "vpc"
  }
}

# The subnet where the Virtual Machine will live
resource "aws_subnet" "subnet_1" {
  vpc_id     = aws_vpc.vpc.id
  cidr_block = "192.168.1.0/24"

  tags = {
    Name = "subnet_1"
  }
}

resource "aws_internet_gateway" "internet_gateway" {
  vpc_id = aws_vpc.vpc.id

  tags = {
    Name = "internet_gateway"
  }
}

resource "aws_route_table" "route_table" {
  vpc_id = aws_vpc.vpc.id

  tags = {
    Name = "route_table"
  }
}

# Enabling the resources from subnet_1 to access the Internet
# So we can access it later via SSH
resource "aws_route" "subnet_1_exit_route" {
  route_table_id         = aws_route_table.route_table.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.internet_gateway.id
}

resource "aws_route_table_association" "route_table_association" {
  subnet_id      = aws_subnet.subnet_1.id
  route_table_id = aws_route_table.route_table.id
}
```

Apply tha changes:
```bash
$ terraform apply
...

Plan: 6 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

_Getting there..._

## Step 4 – AWS VPN stuff
I'm very proud you made to Step 4 in this adventure, we are _almost there_! Let's get some VPN goodies from AWS:

- One VPN Gateway
- Two Customer Gateways for HA
- Two VPN Connections: this piece has the external IP addresses used for the connection with Azure
- Two VPN Connection Routes to tell AWS about the Azure's vNet CIDR
- One entry in the Route Table to teach AWS how to get to Azure

We are also using Terraform's `data resource` to fetch the two public IP addresses from AWS. Why? Because those IP addresses are `Dynamic` and generated only after the Azure's Virtual Network Gateway is up and running. Also, you will see the `azurerm_public_ip` data resource has an interpolated name. Let me explain.

To achieve **one `terraform apply` command** we need to deal with inter-cloud dependencies. The critical one is getting the Azure's public IP addresses – _which are generated only after the Virtual Network Gateway is created_ – to be used in the AWS side. Using the interpolation in the `name` argument forces the data resource to run _after the Virtual Network Gateway is created_.

But wait. You could use `depends_on` on the data resource, right? Yes and no. It works on the first run, but since the `depends_on` forces the read of the data source to _always_ happen in the apply phase, it triggers changes in the AWS components depending on in. [The configuration never converges](https://www.terraform.io/docs/configuration/data-sources.html#data-resource-dependencies).

Ayyy, append the following code in your `aws.tf` file:
```terraform
data "azurerm_public_ip" "azure_public_ip_1" {
  # That neat name interpolation
  # The end result is _exactly_ the name of Azure's public IP
  name                = "${azurerm_virtual_network_gateway.virtual_network_gateway.name}_public_ip_1"
  resource_group_name = azurerm_resource_group.resource_group.name
}

data "azurerm_public_ip" "azure_public_ip_2" {
  name                = "${azurerm_virtual_network_gateway.virtual_network_gateway.name}_public_ip_2"
  resource_group_name = azurerm_resource_group.resource_group.name
}

resource "aws_customer_gateway" "customer_gateway_1" {
  bgp_asn = 65000

  # Using the previously fetched Azure's public IP
  ip_address = data.azurerm_public_ip.azure_public_ip_1.ip_address
  type       = "ipsec.1"

  tags = {
    Name = "customer_gateway_1"
  }
}

resource "aws_customer_gateway" "customer_gateway_2" {
  bgp_asn = 65000

  ip_address = data.azurerm_public_ip.azure_public_ip_2.ip_address
  type       = "ipsec.1"

  tags = {
    Name = "customer_gateway_2"
  }
}

resource "aws_vpn_gateway" "vpn_gateway" {
  vpc_id = aws_vpc.vpc.id

  tags = {
    Name = "vpn_gateway"
  }
}

# We will use information from this piece to finish the Azure configuration on the next Step
resource "aws_vpn_connection" "vpn_connection_1" {
  vpn_gateway_id      = aws_vpn_gateway.vpn_gateway.id
  customer_gateway_id = aws_customer_gateway.customer_gateway_1.id
  type                = "ipsec.1"
  static_routes_only  = true

  tags = {
    Name = "vpn_connection_1"
  }
}

# We will use information from this piece to finish the Azure configuration on the next Step
resource "aws_vpn_connection" "vpn_connection_2" {
  vpn_gateway_id      = aws_vpn_gateway.vpn_gateway.id
  customer_gateway_id = aws_customer_gateway.customer_gateway_2.id
  type                = "ipsec.1"
  static_routes_only  = true

  tags = {
    Name = "vpn_connection_2"
  }
}

resource "aws_vpn_connection_route" "vpn_connection_route_1" {
  # Azure's vnet CIDR
  destination_cidr_block = azurerm_virtual_network.vnet.address_space[0]
  vpn_connection_id      = aws_vpn_connection.vpn_connection_1.id
}

resource "aws_vpn_connection_route" "vpn_connection_route_2" {
  # Azure's vnet CIDR
  destination_cidr_block = azurerm_virtual_network.vnet.address_space[0]
  vpn_connection_id      = aws_vpn_connection.vpn_connection_2.id
}

# The route teaching where to go to get to Azure's CIDR
resource "aws_route" "route_to_azure" {
  route_table_id = aws_route_table.route_table.id

  # Azure's vnet CIDR
  destination_cidr_block = azurerm_virtual_network.vnet.address_space[0]
  gateway_id             = aws_vpn_gateway.vpn_gateway.id
}
```

Do it! Nothing is impossible:
```bash
$ terraform apply
...


Plan: 8 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

It takes about **7 minutes** to finish. Definitely not bad as Azure 😅

## Step 5 – Establishing the Connection on Azure
The last Step created all the AWS resources needed for the VPN. The final piece of the puzzle is to create the Azure components responsible for connecting to AWS, _all those arrows in the diagram_.

Be mindful that each `AWS VPN Connection` created has _two tunnels_ with IP addresses and secret keys. To accomplish full high availability, we need to point each Azure Virtual Network Gateway IP address to _two IPs_ on AWS, creating a connection mesh.

The last VPN deploy includes (I know, it is a lot!):
- 4 Local Network Gateways, one for each AWS VPN tunnel
- 4 Virtual Network Gateway Connection, one for each AWS VPN tunnel

Back to the `azure.tf` file, add:
```terraform
# Tunnel from Azure to AWS vpn_connection_1 (tunnel1)
resource "azurerm_local_network_gateway" "local_network_gateway_1_tunnel1" {
  name                = "local_network_gateway_1_tunnel1"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  # AWS VPN Connection public IP address
  gateway_address = aws_vpn_connection.vpn_connection_1.tunnel1_address

  address_space = [
    # AWS VPC CIDR
    aws_vpc.vpc.cidr_block
  ]
}

resource "azurerm_virtual_network_gateway_connection" "virtual_network_gateway_connection_1_tunnel1" {
  name                = "virtual_network_gateway_connection_1_tunnel1"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  type                       = "IPsec"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.virtual_network_gateway.id
  local_network_gateway_id   = azurerm_local_network_gateway.local_network_gateway_1_tunnel1.id

  # AWS VPN Connection secret shared key
  shared_key = aws_vpn_connection.vpn_connection_1.tunnel1_preshared_key
}

# Tunnel from Azure to AWS vpn_connection_1 (tunnel2)
resource "azurerm_local_network_gateway" "local_network_gateway_1_tunnel2" {
  name                = "local_network_gateway_1_tunnel2"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  gateway_address = aws_vpn_connection.vpn_connection_1.tunnel2_address

  address_space = [
    aws_vpc.vpc.cidr_block
  ]
}

resource "azurerm_virtual_network_gateway_connection" "virtual_network_gateway_connection_1_tunnel2" {
  name                = "virtual_network_gateway_connection_1_tunnel2"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  type                       = "IPsec"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.virtual_network_gateway.id
  local_network_gateway_id   = azurerm_local_network_gateway.local_network_gateway_1_tunnel2.id

  shared_key = aws_vpn_connection.vpn_connection_1.tunnel2_preshared_key
}

# Tunnel from Azure to AWS vpn_connection_2 (tunnel1)
resource "azurerm_local_network_gateway" "local_network_gateway_2_tunnel1" {
  name                = "local_network_gateway_2_tunnel1"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  gateway_address = aws_vpn_connection.vpn_connection_2.tunnel1_address

  address_space = [
    aws_vpc.vpc.cidr_block
  ]
}

resource "azurerm_virtual_network_gateway_connection" "virtual_network_gateway_connection_2_tunnel1" {
  name                = "virtual_network_gateway_connection_2_tunnel1"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  type                       = "IPsec"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.virtual_network_gateway.id
  local_network_gateway_id   = azurerm_local_network_gateway.local_network_gateway_2_tunnel1.id

  shared_key = aws_vpn_connection.vpn_connection_2.tunnel1_preshared_key
}

# Tunnel from Azure to AWS vpn_connection_2 (tunnel2)
resource "azurerm_local_network_gateway" "local_network_gateway_2_tunnel2" {
  name                = "local_network_gateway_2_tunnel2"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  gateway_address = aws_vpn_connection.vpn_connection_2.tunnel2_address

  address_space = [
    aws_vpc.vpc.cidr_block
  ]
}

resource "azurerm_virtual_network_gateway_connection" "virtual_network_gateway_connection_2_tunnel2" {
  name                = "virtual_network_gateway_connection_2_tunnel2"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  type                       = "IPsec"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.virtual_network_gateway.id
  local_network_gateway_id   = azurerm_local_network_gateway.local_network_gateway_2_tunnel2.id

  shared_key = aws_vpn_connection.vpn_connection_2.tunnel2_preshared_key
}
```

And apply it:
```
$ terraform apply

Plan: 8 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

_Wait for it..._

## WE DID IT
Let the connection sit a bit to get itself together. After a minute or two, inspect the connection in the dashboard.

On AWS, go to `VPC > Site-to-Site VPN Connections` and take a look at the `Tunnel Details` tab under `vpn_connection_1` and `vpn_connection_2`:
![](/img/aws_vpn_conn1.png)

![](/img/aws_vpn_conn2.png)

The green color feels very good, doesn't it?

On Azure, go to `Virtual Network Gateways > Connections` and this beauty will pop up:
![](/img/azure_vpn_conn.png)

To make sure the configuration is _converged_, just run another `terraform apply`:
```bash
$ terraform apply

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

YES, WE DID IT! 🎉🎊🍾

## The Final Test 
A VPN tunnel is beautiful, but we need to make sure it works. Let's create one Virtual Machine in each side and try to ping them through their local networks – no public IP pinging allowed here.

Create a new file named `vms.tf` and throw the following code there:
```terraform
### Azure
resource "azurerm_public_ip" "public_ip_vm" {
  name                = "public_ip_vm"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  allocation_method = "Static"
}

resource "azurerm_network_interface" "network_interface_vm" {
  name                = "network_interface_vm"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet_1.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id = azurerm_public_ip.public_ip_vm.id
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "vm"
  location            = azurerm_resource_group.resource_group.location
  resource_group_name = azurerm_resource_group.resource_group.name
  size                = "Standard_F2"
  admin_username      = "ubuntu"

  network_interface_ids = [
    azurerm_network_interface.network_interface_vm.id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  admin_ssh_key {
    username   = "ubuntu"
    # IMPORTANT
    # Add here your own public SSH key, so we can access the VM
    public_key = ""
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}

output "azure_vm_public_ip" {
  value = azurerm_linux_virtual_machine.vm.public_ip_address
}

output "azure_vm_private_ip" {
  value = azurerm_linux_virtual_machine.vm.private_ip_address
}

### AWS
resource "aws_security_group" "ssh" {
  vpc_id = aws_vpc.vpc.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "security_group_ssh"
  }
}

resource "aws_key_pair" "ssh_key" {
  key_name   = "ssh_key"
  # IMPORTANT
  # Add here your own public SSH key, so we can access the VM
  public_key = ""
}

resource "aws_instance" "vm" {
  ami           = "ami-0701e7be9b2a77600"
  instance_type = "t2.micro"

  vpc_security_group_ids      = [aws_security_group.ssh.id]
  subnet_id                   = aws_subnet.subnet_1.id
  associate_public_ip_address = true

  key_name = aws_key_pair.ssh_key.key_name
}

output "aws_vm_public_ip" {
  value = aws_instance.vm.public_ip
}

output "aws_vm_private_ip" {
  value = aws_instance.vm.private_ip
}
```

And, for the last time, let it rip:
```bash
$ terraform apply
...

Plan: 6 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
...

Apply complete! Resources: 6 added, 0 changed, 0 destroyed.

Outputs:

aws_vm_private_ip = 192.168.1.218
aws_vm_public_ip = 34.251.101.219
azure_vm_private_ip = 10.0.1.4
azure_vm_public_ip = 52.166.20.226
```

With the Terraform output, SSH into each machine with the `public_ip` and try to ping one another through the `private_ip` (with your own output data):

From AWS:
```bash
$ ssh ubuntu@34.251.101.219 ping 10.0.1.4 -c 4

PING 10.0.1.4 (10.0.1.4) 56(84) bytes of data.
64 bytes from 10.0.1.4: icmp_seq=1 ttl=64 time=21.3 ms
64 bytes from 10.0.1.4: icmp_seq=2 ttl=64 time=20.7 ms
64 bytes from 10.0.1.4: icmp_seq=3 ttl=64 time=21.3 ms
64 bytes from 10.0.1.4: icmp_seq=4 ttl=64 time=22.3 ms

--- 10.0.1.4 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 20.761/21.454/22.353/0.591 ms
```

From Azure:
```bash
$ ssh ubuntu@52.166.20.226 ping 192.168.1.218 -c 4

PING 192.168.1.218 (192.168.1.218) 56(84) bytes of data.
64 bytes from 192.168.1.218: icmp_seq=1 ttl=64 time=20.1 ms
64 bytes from 192.168.1.218: icmp_seq=2 ttl=64 time=20.2 ms
64 bytes from 192.168.1.218: icmp_seq=3 ttl=64 time=20.4 ms
64 bytes from 192.168.1.218: icmp_seq=4 ttl=64 time=20.5 ms

--- 192.168.1.218 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 20.119/20.338/20.539/0.169 ms
```

Awesome, right?

## Bonus – One Apply to Rule them All
A step-by-step process is amazing to learn, but in real life, we should strive to layout our Terraform code in a way that _one_ `terraform apply` brings everything up, especially when we are creating new environments. Everything so far was based on that philosophy. 

For the final proof, let's destroy the infrastructure and bring it back up.

> 🚨 If you are running this code in an established running environment – be it dev, staging or production – make sure you know what is being destroyed and that is safe to proceed!

```bash
$ terraform destroy
...

Plan: 0 to add, 0 to change, 35 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes
```

And, let's recreate _everything_. It will take a lot of time, let it run in a separate terminal and go pet a dog.

```bash
$ terraform apply

Plan: 35 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

...

Apply complete! Resources: 35 added, 0 changed, 0 destroyed.

Outputs:

aws_vm_private_ip = 192.168.1.83
aws_vm_public_ip = 34.252.37.174
azure_vm_private_ip = 10.0.1.4
azure_vm_public_ip = 137.117.140.139
```

Do the `ping` test again and voilà. You are the best.

## aaaaaaaaaaaaaaaaaaand, scene!
Well, that was a lot, but we did it! An enormous amount of Terraform code and our highly available site-to-site VPN between AWS and Azure is up and running! If you wanna dig deep, explore the documentation about each component to clearly understand their role in the infrastructure.

All the feedback is appreciated ✨