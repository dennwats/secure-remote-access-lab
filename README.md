
# 🛡️ Secure Remote Access Lab – AWS Client VPN for Amazon Connect

## 🧠 Objective

To provide a scalable and secure VPN setup using AWS Client VPN with Elastic IPs (static IPs) that integrate with Amazon Connect, allowing 2–4 users to securely connect using the AWS VPN Client. This solution uses mutual authentication for simplicity.

---

## 1️⃣ Elastic IPs (Static Public IPs)

Go to **EC2 > Elastic IPs**  
Allocate 4 Elastic IPs:

| Elastic IP       | Label         |
|------------------|---------------|
| 174.129.181.28   | Elastic-IP1   |
| 23.20.132.38     | Elastic-IP2   |
| 3.212.19.151     | Elastic-IP3   |
| 52.72.254.176    | Elastic-IP4   |

---

## 2️⃣ Subnets (CIDR: 172.16.2.0/24 Block)

Use four `/27` subnets.  
Go to **VPC > Subnets** and create:

| Subnet Name     | CIDR               | Notes  |
|------------------|--------------------|--------|
| Elastic-Subnet1  | 172.16.2.0/27      | 32 IPs |
| Elastic-Subnet2  | 172.16.2.32/27     | 32 IPs |
| Elastic-Subnet3  | 172.16.2.64/27     | 32 IPs |
| Elastic-Subnet4  | 172.16.2.96/27     | 32 IPs |

✅ Associate with VPC: `vpc-022f3264c489cca83`

---

## 3️⃣ NAT Gateways (One per Subnet)

Go to **VPC > NAT Gateways**, create one per subnet:

| NAT Gateway Name | Elastic IP       | Subnet           |
|------------------|------------------|------------------|
| Elastic-Nat1     | 174.129.181.28   | Elastic-Subnet1  |
| Elastic-Nat2     | 23.20.132.38     | Elastic-Subnet2  |
| Elastic-Nat3     | 3.212.19.151     | Elastic-Subnet3  |
| Elastic-Nat4     | 52.72.254.176    | Elastic-Subnet4  |

---

## 4️⃣ Route Tables (One per Subnet)

Go to **VPC > Route Tables**, create:

| Route Table Name     | Subnet Association | Route                  |
|----------------------|--------------------|------------------------|
| Elastic-RouteTable1  | Elastic-Subnet1    | 0.0.0.0/0 → NAT1       |
| Elastic-RouteTable2  | Elastic-Subnet2    | 0.0.0.0/0 → NAT2       |
| Elastic-RouteTable3  | Elastic-Subnet3    | 0.0.0.0/0 → NAT3       |
| Elastic-RouteTable4  | Elastic-Subnet4    | 0.0.0.0/0 → NAT4       |

---

## 5️⃣ Server Certificate Setup (Mutual Authentication)

SSH into an Amazon Linux 2 instance, then:

```bash
sudo yum install git -y
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1 nopass
```

Download:
- `pki/ca.crt`
- `pki/issued/server.crt`
- `pki/private/server.key`

Upload to **AWS Certificate Manager (ACM)**:
- Certificate: `server.crt`
- Private Key: `server.key`
- Chain: `ca.crt`

✅ Save the Certificate ARN

---

## 6️⃣ VPN Endpoint Setup

Go to **VPC > Client VPN Endpoints**, configure:

| Setting               | Value                      |
|-----------------------|----------------------------|
| Client CIDR           | 172.16.3.0/22              |
| Server Certificate    | (from ACM)                 |
| Auth Type             | Mutual Authentication      |
| Client CA             | `ca.crt`                   |
| Protocol              | UDP                        |
| Port                  | 443                        |

---

## 7️⃣ Associate Subnets to VPN Endpoint

Go to **Target Network Associations**  
✅ Associate all 4 subnets (must be /27 or smaller)

---

## 8️⃣ VPN Route Configuration

Go to **Client VPN Endpoint > Route Table**  
Add:

| Destination CIDR | Target               |
|------------------|----------------------|
| 0.0.0.0/0        | All Associated Subnets |

---

## 9️⃣ Access Rules (Authorization)

Go to **Authorization Rules**  
Add:

| Destination CIDR | Grant Access To |
|------------------|------------------|
| 0.0.0.0/0        | All users        |

---

## 🔟 Client Setup

- Go to **Client VPN Endpoint > Download Configuration**
- Import into AWS VPN Client
- Connect → VPN client is assigned 1 of the 4 Elastic IPs

---

## 🧩 Amazon Connect Integration

Ensure:
- Amazon Connect and VPN are in same or peered VPCs
- Security groups/NACLs allow traffic from VPN to Connect

---

## 🧹 Cleanup After Event

- Delete VPN Endpoint
- Disassociate/Delete NAT Gateways
- Release Elastic IPs
- Delete Subnets & Route Tables
- Terminate EC2 used for cert generation

---

## ✅ Business Value

- Enabled secure, remote access to Amazon Connect
- Maintained identity accountability with Elastic IPs
- Achieved scalable VPN design in under 48 hours
- Built on AWS-native services for compliance and ease of management

