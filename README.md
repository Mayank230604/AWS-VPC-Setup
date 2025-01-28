# AWS VPC Setup: Public and Private Subnets Across Two Availability Zones

This guide details how to create an AWS Virtual Private Cloud (VPC) setup with the following architecture:

- A single VPC.
- Two Availability Zones (AZs): **Zone A** and **Zone B**.
- Each AZ contains:
  - **Zone A**: Public Subnet 1 and Private Subnet 1.
  - **Zone B**: Public Subnet 2 and Private Subnet 2.
- Public subnets connect to the internet using an **Internet Gateway (IGW)**.
- Private subnets securely communicate via a **Virtual Private Gateway (VPG)**.

---

## Components Overview

### 1. Virtual Private Cloud (VPC)
A logically isolated section of the AWS cloud that enables custom networking configurations like subnets, IP ranges, and gateways.

### 2. Subnets
Subnets divide a VPC into smaller IP ranges. Subnets are classified as:
- **Public**: Accessible via the internet.
- **Private**: Isolated and only accessible internally.

### 3. Gateways
- **Internet Gateway (IGW)**: Connects public subnets to the internet.
- **Virtual Private Gateway (VPG)**: Allows private subnets to communicate with other private networks.

### 4. Route Tables
Define rules for directing traffic between subnets, gateways, and external networks.

---

## Architecture Details

### VPC Configuration
- **CIDR Block**: `10.0.0.0/24`

### Subnet Configuration
- **Zone A**:
  - Public Subnet 1: `10.0.0.0/26`
  - Private Subnet 1: `10.0.0.128/26`
- **Zone B**:
  - Public Subnet 2: `10.0.0.64/26`
  - Private Subnet 2: `10.0.0.192/26`

### Gateways
- **IGW**: Routes public traffic to the internet.
- **VPG**: Enables private subnets to communicate securely.

---

## Step-by-Step Guide

### Step 1: Create the VPC
1. Log in to the AWS Management Console and open **VPC Dashboard**.
2. Click **Create VPC** and configure:
   - **Name**: `My-VPC`
   - **IPv4 CIDR Block**: `10.0.0.0/24`
3. Leave all other options as default and click **Create VPC**.

---

### Step 2: Create Subnets

#### For Zone A
1. Go to **Subnets** → **Create Subnet**:
   - **Name**: `Public-Subnet-1`
   - **VPC**: Select `My-VPC`
   - **AZ**: Select `us-east-1a`
   - **CIDR Block**: `10.0.0.0/26`
2. Repeat the steps to create `Private-Subnet-1`:
   - **CIDR Block**: `10.0.0.128/26`.

#### For Zone B
1. Create `Public-Subnet-2`:
   - **AZ**: Select `us-east-1b`
   - **CIDR Block**: `10.0.0.64/26`.
2. Create `Private-Subnet-2`:
   - **CIDR Block**: `10.0.0.192/26`.

---

### Step 3: Set Up the Internet Gateway (IGW)
1. In **Internet Gateways**, click **Create Internet Gateway**:
   - **Name**: `My-IGW`
2. Attach the IGW to `My-VPC`:
   - Select the IGW → **Actions** → **Attach to VPC** → Choose `My-VPC`.

---

### Step 4: Set Up the Virtual Private Gateway (VPG)
1. In **Virtual Private Gateways**, click **Create Virtual Private Gateway**:
   - **Name**: `My-VPG`
   - Leave the ASN setting as default.
2. Attach the VPG to `My-VPC`:
   - Select the VPG → **Actions** → **Attach to VPC** → Choose `My-VPC`.

---

### Step 5: Configure Route Tables

#### Public Route Table
1. In **Route Tables**, click **Create Route Table**:
   - **Name**: `Public-Route-Table`
   - **VPC**: Select `My-VPC`.
2. Add a route for internet access:
   - **Routes** tab → **Edit Routes** → Add:
     - **Destination**: `0.0.0.0/0`
     - **Target**: `My-IGW`.
3. Associate this route table with public subnets:
   - **Subnet Associations** → **Edit Subnet Associations** → Select `Public-Subnet-1` and `Public-Subnet-2`.

#### Private Route Table
1. Create another route table:
   - **Name**: `Private-Route-Table`
   - **VPC**: Select `My-VPC`.
2. Enable propagation for the VPG:
   - **Route Propagation** → **Edit Route Propagation** → Select `My-VPG`.
3. Associate this route table with private subnets:
   - **Subnet Associations** → **Edit Subnet Associations** → Select `Private-Subnet-1` and `Private-Subnet-2`.

---

### Step 6: Launch EC2 Instances

#### For Public Subnets
1. Launch an EC2 instance in `Public-Subnet-1`:
   - Choose an AMI (e.g., Amazon Linux 2).
   - Assign a public IP address.
2. Repeat for `Public-Subnet-2`.

#### For Private Subnets
1. Launch an EC2 instance in `Private-Subnet-1`:
   - Assign to `Private-Subnet-1`.
   - Disable public IP assignment.
2. Repeat for `Private-Subnet-2`.

---

### Step 7: Set Up Security Groups for SSH Access

#### Create a Security Group
1. In **Security Groups**, click **Create Security Group**:
   - **Name**: `SSH-Access`
   - **VPC**: Select `My-VPC`.
2. Add an inbound rule:
   - **Type**: `SSH`
   - **Source**: Enter your IP or a custom CIDR block.

#### Attach the Security Group to Instances
1. In the **EC2 Dashboard**, select the instance.
2. Click **Actions** → **Security** → **Change Security Groups**.
3. Assign `SSH-Access` to your instance.

---

## Testing and Verification
1. Test public instances for internet access using their public IPs.
2. Confirm private instances communicate securely through the VPG.
3. Use SSH or ping to check connectivity between instances.

---

## References
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [AWS Internet Gateway Guide](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [AWS Virtual Private Gateway Guide](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
