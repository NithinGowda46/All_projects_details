## 1. Infrastructure Setup

### 1.1 AWS Region

- **Region:** `ap-southeast-1` (Singapore)

### 1.2 Create VPC

The VPC was created with the following configuration:

| Setting | Value |
|---|---|
| VPC Name | `vpc1` |
| IPv4 CIDR | `10.18.0.0/16` |
| Tenancy | Default |
| DNS Resolution | Enabled |
| DNS Hostnames | Disabled |
| Default VPC | No |
| State | Available |

### 1.3 Create Subnets

The following subnets were created inside `vpc1`:

| Name | IPv4 CIDR |
|---|---|
| `1a_public_subnet` | `10.18.0.0/21` |
| `1b_public_subnet` | `10.18.8.0/21` |
| `1c_public_subnet` | `10.18.16.0/21` |
| `1a_private_subnet` | `10.18.24.0/21` |
| `1b_private_subnet` | `10.18.32.0/21` |
| `1c_private_subnet` | `10.18.40.0/21` |
| `2a_private_subnet` | `10.18.48.0/21` |
| `2b_private_subnet` | `10.18.56.0/21` |
| `2c_private_subnet` | `10.18.64.0/21` |

All shown subnets are in `vpc1` and are in the **Available** state.

![VPC Subnets](images/subnets.png)