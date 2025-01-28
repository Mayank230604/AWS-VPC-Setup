# AWS VPC Setup: Public and Private Subnets in Two Availability Zones

![image](https://github.com/user-attachments/assets/44a2b3cb-bdf0-47c0-8931-470f97ee617b)


This README provides step-by-step guidance to create an AWS VPC with the following architecture:

- A single VPC.
- Two Availability Zones (AZs): **Zone A** and **Zone B**.
- Each AZ includes one public subnet and one private subnet:
  - **Zone A**: Public Subnet 1 & Private Subnet 1.
  - **Zone B**: Public Subnet 2 & Private Subnet 2.
- Public subnets connect to the internet via an **Internet Gateway (IGW)**.
- Private subnets communicate securely using a **Virtual Private Gateway (VPG)**.

---

## Key Components

### 1. Virtual Private Cloud (VPC)
A VPC is an isolated network within AWS, where you can launch resources in a customized network. It provides control over IP ranges, subnets, route tables, and gateways.

### 2. Subnets
Subnets partition your VPC into smaller IP ranges within specific Availability Zones. Subnets can be:
- **Public**: Accessible from the internet.
- **Private**: Isolated from direct internet access.

### 3. Internet Gateway (IGW)
The IGW is attached to the VPC to allow instances in public subnets to access the internet.

### 4. Virtual Private Gateway (VPG)
A VPG facilitates secure communication between private subnets and other private networks, such as on-premises systems or other VPCs.

### 5. Route Table
A route table defines rules to control how traffic is directed within the VPC.

### 6. Availability Zone (AZ)
An AZ is a physical data center within an AWS Region. Using multiple AZs enhances fault tolerance and high availability.

---

## Architecture Details

### VPC Configuration
- **CIDR Block**: `10.0.0.0/24`

### Subnets
- **Zone A**:
  - Public Subnet 1: `10.0.0.0/26`
  - Private Subnet 1: `10.0.0.128/26`
- **Zone B**:
  - Public Subnet 2: `10.0.0.64/26`
  - Private Subnet 2: `10.0.0.192/26`

### Gateways
- **IGW**: Connects public subnets to the internet.
- **VPG**: Enables communication between private subnets.

---

## Step-by-Step Guide

### Step 1: Create the VPC

1. Open the **VPC Dashboard** in the AWS Management Console.
2. Click **Create VPC** and configure:
   - **Name**: `My-VPC`.
   - **IPv4 CIDR Block**: `10.0.0.0/24`.
3. Leave all other options as default and click **Create VPC**.

### Step 2: Create Subnets

#### For Zone A
1. Go to **Subnets** → **Create Subnet**:
   - **Name**: `Public-Subnet-1`.
   - **VPC**: Select `My-VPC`.
   - **AZ**: Choose an AZ for Zone A (e.g., `us-east-1a`).
   - **CIDR Block**: `10.0.0.0/26`.
2. Click **Create Subnet**.
3. Repeat the steps to create `Private-Subnet-1` with:
   - **AZ**: `us-east-1a`.
   - **CIDR Block**: `10.0.0.128/26`.

#### For Zone B
1. Create `Public-Subnet-2`:
   - **AZ**: Select a different AZ for Zone B (e.g., `us-east-1b`).
   - **CIDR Block**: `10.0.0.64/26`.
2. Create `Private-Subnet-2` with:
   - **AZ**: `us-east-1b`.
   - **CIDR Block**: `10.0.0.192/26`.

### Step 3: Set Up an Internet Gateway (IGW)

1. In **Internet Gateways**, click **Create Internet Gateway**:
   - **Name**: `My-IGW`.
2. Attach the IGW to `My-VPC`:
   - Select the IGW → **Actions** → **Attach to VPC** → Choose `My-VPC`.

### Step 4: Set Up a Virtual Private Gateway (VPG)

1. In **Virtual Private Gateways**, click **Create Virtual Private Gateway**:
   - **Name**: `My-VPG`.
   - Leave the default ASN setting.
2. Attach the VPG to `My-VPC`:
   - Select the VPG → **Actions** → **Attach to VPC** → Choose `My-VPC`.

### Step 5: Configure Route Tables

#### Public Route Table
1. In **Route Tables**, click **Create Route Table**:
   - **Name**: `Public-Route-Table`.
   - **VPC**: Select `My-VPC`.
2. Add an internet route:
   - **Routes** tab → **Edit Routes** → Add:
     - **Destination**: `0.0.0.0/0`.
     - **Target**: `My-IGW`.
3. Associate this route table with the public subnets:
   - **Subnet Associations** → **Edit Subnet Associations** → Select `Public-Subnet-1` and `Public-Subnet-2`.

#### Private Route Table
1. Create a second route table:
   - **Name**: `Private-Route-Table`.
   - **VPC**: Select `My-VPC`.
2. Enable propagation for the VPG:
   - **Route Propagation** → **Edit Route Propagation** → Select `My-VPG`.
3. Associate this route table with the private subnets:
   - **Subnet Associations** → **Edit Subnet Associations** → Select `Private-Subnet-1` and `Private-Subnet-2`.

---

### Step 6: Launch EC2 Instances

#### For Public Subnets
1. Launch an EC2 instance in `Public-Subnet-1`:
   - Choose an AMI (e.g., Amazon Linux 2).
   - Assign a public IP.
2. Repeat for `Public-Subnet-2`.

#### For Private Subnets
1. Launch an EC2 instance in `Private-Subnet-1`:
   - Assign to `Private-Subnet-1`.
   - Disable public IP assignment.
2. Repeat for `Private-Subnet-2`.

---

### Step 7: Security Group for SSH Access

#### Create a Security Group
1. In **Security Groups**, click **Create Security Group**:
   - **Name**: `Private-SSH-Access`.
   - **VPC**: Select `My-VPC`.
2. Add an inbound rule:
   - **Type**: `SSH`.
   - **Source**: Enter your IP or a custom CIDR block.

#### Attach to Instances
1. Open the **EC2 Dashboard**.
2. Select your instance → **Actions** → **Security** → **Change Security Groups**.
3. Assign `Private-SSH-Access` to your private instances.

---

## Verification
- Verify public subnets can access the internet.
- Confirm private subnets communicate only via VPG.
- Test connectivity between instances using SSH or ping.

---

## References
- [AWS VPC Overview](https://docs.aws.amazon.com/vpc/latest/userguide/)
- [Setting Up Internet Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [VPG Configuration](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPC_VPN.html)
