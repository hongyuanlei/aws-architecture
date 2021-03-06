### VPC

The Amazon Virtual Private Cloud(VPC) is a custom-defined virtual network within the AWS Cloud. You can provision your own logically isolated section of AWS, similar to designing and implementing a separate independent network that would operate in an on-premises data center.

Amazon VPC is the networking layer for Amazon Elastic Compute Cloud(Amazon EC2), and it allows you to build your own virtual network within AWS. You control various aspects of your Amazon VPC, including:
- Selecting your own IP address range.
- Creating your own subnets
- Configuring your own route tables, network gateways, and security setting.

Within a region, you can create multiple Amazon VPCs, and each Amazon VPC is logically isolated even if it shares its IP address space.

When you create an Amazon VPC, you must specify the IPv4 address range by choosing a Classless Inter-Domain Routing(CIDR) block, such as 10.0.0.0/16. The address range of the Amazon VPC cannot be changed after the Amazon VPCs created. An Amazon VPC address range may be as large as `/16`(65536 available addresses) or as small as `/28`(16 available addresses) and should not overlap any other network with which they are to be connected.

Here's an example, an Amazon VPC with an address space of 10.0.0.0/16, two subnets with different address ranges (10.0.0.0/24 and 10.0.1.0/24) placed in different Availability Zones, and a route table with the local route specified.

<img src="./images/aws-vpc-vpc.png" width="720"/>

An Amazon VPC consists of the following components:
- Subnets
- Route tables
- Dynamic Host Configuration Protocol (DHCP) option sets 
- Security groups
- Network Access Control Lists (ACLs)

An Amazon VPC has the following optional components: 
- Internet Gateways (IGWs)
- Elastic IP (EIP) addresses
- Elastic Network Interfaces (ENIs)
- Endpoints
- Peering
- Network Address Translation (NATs) instances and NAT gateways
- Virtual Private Gateway (VPG), Customer Gateways (CGWs), and Virtual Private Networks (VPNs)

### Subnets

A subnet is a segment of an Amazon VPC's IP address range where you can launch AWS resources, CIDR define subnets. The smallest subnet that you can create is a `/28`(16 IP address). **AWS reserves that first four IP addresses and the last IP address of every subnet for internal networking purposes**.

- 10.0.0.0: Network address.
- 10.0.0.1: Reserved by AWS for the VPC router.
- 10.0.0.2: Reserved by AWS. The IP address of the DNS server is always the base of the VPC network range plus two; however, we also reserve the base of each subnet range plus two. For VPCs with multiple CIDR blocks, the IP address of the DNS server is located in the primary CIDR.
- 10.0.0.3: Reserved by AWS for future use.
- 10.0.0.255: Network broadcast address. We do not support broadcast in a VPC, therefore we reserve this address.

After creating an Amazon VPC, you can add one or more subnets in each Availability Zone. **Subnets reside within one Availability Zone and cannot span zones.**

Subnets can be classified as public, private, or VPN-only. 
- A public subnet is one in which the associated route table directs the subnet's traffic to the Amazon VPC's IGW.
- A private subnet is one in which the associated route table does not direct the subnet's traffic to the Amazon VPC's IGW.
- A VPN-only subnet is one in which the associated route table directs the subnet's traffic to the Amazon VPC's VPG and does not have a route to the IGW. 

Regardless of the type of subnet, the internal IP address range of the subnet is always private(that is non-routable on the internet).

### Route Tables

A route table is a logical construct within an Amazon VPC that contains a set of rules that are applied to the subnet and used to determine where network traffic is directed. You can modify the route tables and add your own custom routes. You can also use route tables to specify which subnets are public(by directing Internet traffic to the IGW) and which subnets are private(by not having a route that directs traffic to IGW).

Each route table contains a default route called the local route, which enables communication within the Amazon VPC, and this route cannot be modified or removed. Additional route can be added to direct traffic to exit the Amazon VPC via the IGW, the VPC, or the NAT instance.

You should remember the following points about route tables:

- Your VPC has an implicit router.
- Your VPC automatically comes with a main route table that you can modify.
- You can create additional custom route tables for your VPC.
- Each subnet must be associated with a route table, which controls the routing for the subnet. If you don't explicitly associate a subnet with a particular route table, the subnet uses the main route table.
- You can replace the main route table with a custom table that you've created so that each new subnet is automatically associated with it.
- Each route in a table specifies a destination CIDR and a target; for example, traffic destined for 172.16.0.0/12 is targeted for the VPG. AWS uses the most specific route that matches the traffic to determine how to route the traffic.

### Internet Gateways

An Internet Gateway(IGW) is a horizontally scaled, redundant, and highly available Amazon VPC component that allows communication between instances in your Amazon VPC and the Internet. An IGW provides a target in your Amazon VPC route tables for Internet-routable traffic, and it performs network address translation for instances that have been assigned public IP addresses.

**Amazon EC2 instances within an Amazon VPC are only aware of their private IP addresses**. When traffic is sent from the instance to the Internet, the IGW translates the reply address to the instance's public IP address(or EIP address) and maintains the one-to-one map of the instance private IP address and public IP address. When an instance receives traffic from the Internet, the IGW translates the destination address(public IP address) to the instance's private IP address and forwards the traffic to the Amazon VPC.

You must do the following to create a public subnet with Internet access:
- Attach an IGW to your Amazon VPC.
- Create a subnet route table rule to send all non-local traffic(0.0.0.0/0) to the IGW.
- Configure your network ACLs and security group rules to allow relevant traffic to flow to and from your instance.

You must do the following to enable an Amazon EC2 instance to send and receive traffic from the Internet:
- Assign a public IP address or EIP address.

You can scope the route to all destinations not explicitly known to the route table(0.0.0.0/0), or you can scope the route to a narrower range of IP addresses, such as the public IP addresses of your company's public endpoint outside of AWS or the EIP addresses of other Amazon EC2 instances outside your Amazon VPC.

Here's an example, an Amazon VPC with an address space of 10.0.0.0/16, one subnet with an address range of 10.0.0.0/24, a route table, an attached IGW, and a single Amazon EC2 instance with a private IP address and an EIP address. The route table contains two routes: the local route that permits inter-VPC communication and a route that sends all non-local traffic to the IGW (igw-id). Note that the Amazon EC2 instance has a public IP address (EIP = 198.51.100.2); this instance can be accessed from the Internet, and traffic may originate and return to this instance.

<img src="./images/aws-vpc-route-table.png" width="720"/>

### Dynamic Host Configuration Protocl (DHCP) Option Sets

DHCP provides a standard for passing configuration information to hosts on a TCP/IP network. The options field of a DHCP message contains the configuration parameters. Some of those parameters are the domain name, domain name server, and the netbios-node-type.

AWS automatically creates and associates a DHCP option set for your Amazon VPC upon creation and sets two options: 
- domain-name-server(defaulted to AmazonProvidedDNS)
AmazonProvidedDNS is an Amazon Domain Name System(DNS) server, and this option enables DNS for instances that need to communicate over the Amazon VPC's IGW.
- domain-name(defaulted to the domain name for your region). 

The DHCP option sets element of an Amazon VPC allows you to direct Amazon EC2 hostname assignments to your own resources. To assign your own domain name to your instances, create a custom DHCP option set and assign it to your Amazon VPC. You can configure the following values within a DHCP option set:

- domain-name-servers - The IP addresses of up to four domain name servers, separated by commas. The default is AmazonProvidedDNS.
- domain-name - Specify the desired domain name here(for example, mycompany.com).
- ntp-servers - The IP addresses of up to four Network Time Protocol(NTP) servers.
- netbios-name-servers - The IP addresses of up to four NetBIOS name servers, separated by commas.
- netbios-node-type - Set this value to 2.

Every Amazon VPC must have only one DHCP option set assigned to it.

### Elastic IP Addresses(EIPs)

AWS maintains a pool of public IP addresses in each region and makes them available for you to associate to resources within your Amazon VPCs. An Elastic IP address is a static, public IP address in the pool for the region that you can allocate to your account(pull from the pool) and release(return to the pool). EIPs allows you to maintain a set of IP addresses that remain fixed while the underlying infrastructure may change over time. Here are the important points to understand:

- You must first allocate an EIP for use within a VPC and then assign it to an instance.
- EIPs are specific to a region.
- There is a one-to-one relationship between network interfaces and EIPs.
- You can move EIPs from one instance to another, either in the same Amazon VPC or a different Amazon VPC within the same region.
- EIPs remain associated with your AWS account until you explicitly release them.
- There are charges for EIPs allocated to your account, even when you are not associated with a resource.

### Elastic Network Interfaces(ENIs)

An ENI is a network interface that you can attach to an instance in an Amazon VPC. ENIs are only available within an Amazon VPC, and they are associated with a subnet upon creation. They can have one public IP address and multiple private IP addresses. If there are multiple private IP addresses, one of them is primary(eth0). An ENI created independently of a particular instance persists regardless of the lifetime of any instance to which it is attached; if an underlying instance fails, the IP address may be preserved by attaching the ENI to a replacement instance.

ENIs allows you:
- Create a management network 
- Use network and security appliances in your Amazon VPC
- Create dual-homed instances with workloads/roles on distinct subnets.
- Create a low-budget, high-availability solution.

#### Creating a Management Network

You can create a management network using network interfaces. In this scenario, the primary network interface(eth0) on the instance handles public traffic and the secondary network interface(eth1) handles backend management traffic and is connected to a separate subnet in your VPC that has more restrictive access controls. The public interface, which may or may not be behind a load balancer, has an associated security group that allows access to the server from the internet(for example, allow TCP port 80 and 443 from 0.0.0.0/0, or from the load balancer) while the private facing interface has an associated security group allowing SSH access only from an allowed range of IP addresses either within the VPC or from the internet, a private subnet within the VPC or virtual private gateway.

To ensure failover capabilities, consider using a secondary private IPv4 for incoming traffic on a network interface. In the event of an instance failure, you can move the interface and/or secondary private IPv4 address to a standby instance.

<img src="./images/aws-vpc-eni.png" width="720"/>

Subnet A Route Table

Destination    | Target
-------------- | ------
192.168.0.0/16 | local
0.0.0.0/0      | igw-1a2b3c4d

Subnet B Route Table

Destination    | Target
-------------- | ------
192.168.0.0/16 | local
0.0.0.0/0      | vgw-4c5d6e7f

Webserver Route Table

Network        | Interface
-------------- | ---------
0.0.0.0/0      | 192.168.0.10  (eth0 Private IP)
172.16.0.0     | 192.168.1.201 (eth1 Private IP)
192.168.0.0    | 192.168.1.201 (eth1 Private IP)
Defaut gateway | 192.168.0.10  (eth0 Private IP)

Security Group for eth0

Source        | Protocol | Port Range | Comments
------------- | -------- | ---------- | --------
0.0.0.0/0     | All      | 80         | Allow inbound traffic from the Internet to port 80.

Security Group for eth1

Source        | Protocol | Port Range | Comments
------------- | -------- | ---------- | --------
172.16.0.0/16 | All      | 22         |Allow Secure Shell (SSH) traffic from Corporate data center.

#### Use Network and Security Appliances in Your VPC

Some network and security appliances, such as load balancers, network address translation(NAT) servers, and proxy servers prefer to be configured with multiple network interfaces. You can create and attach secondary network interfaces to instances in a VPC that are running these types of applications and configure the additional interfaces with their own public and private IP addresses, security groups, and source/destination checking.

#### Creating Dual-homed Instances with Workloads/Roles on Distinct Subnets

You can place a network interface on each of your web servers that connect to a mid-tier network where an application server resides. The application server can also be dual-homed to a backend network(subnet) where the database server resides. Instead of routing network packets through the dual-homed instances, each dual-homed instance receives and processes requests on the front end, initiates a connection to the backend, and then sends requests to the servers on the backend network.

#### Create a Low Budget High Availability Solution

If one of your instances serving a particular function fails, its network interface can be attached to a replacement or hot standby instance pre-configured for the same role in order to rapidly recover the service. For example, you can use a network interface as your primary or secondary network interface to a critical service such as a database instance or a NAT instance. If the instance fails, you can attach the network interface to a hot standby instance. **Because the interface maintains its private IP addresses, Elastic IP addresses, and MAC address, network traffic begins flowing to the standby instance as soon as you attach the network interface to the replacement instance**. Users experience a brief loss of connectivity between the time the instance fails and the time that the network interface is attached to the standby instance, but no changes to the VPC route table or your DNS server are required.

### Security Groups

A security group is virtual stateful firewall that controls inbound and outbound network traffic to AWS resources and Amazon EC2 instances. All Amazon EC2 instances must be launched into a security group. **If a security group is not specified at launch, then the instance will be launched into the default security group for the Amazon VPC**. The default security group allows communication between all resources within the security group, allows all outbound traffic, and denies all other traffic. You may change the rules for the default security group, but you may not delete the default security group.

Here are the settings of the default security group.

- Inbound

Source        | Protocol | Port Range | Comments
------------- | -------- | ---------- | --------
sg-xxxxx      | All      | All        | Allow inbound traffic from instances within the same security group. 

- Outbound

Destination   | Protocol | Port Range | Comments
------------- | -------- | ---------- | --------
0.0.0.0/0     | All      | All        | Allow all outbound traffic

For each security group, you add rules that control the inbound traffic to instances and a separate set of rules that control the outbound traffic. 

For example, here's a security group for web servers.

- Inbound

Source                                | Protocol | Port Range | Comments
------------------------------------- | -------- | ---------- | --------
0.0.0.0/0                             | TCP      | 80         | Allow inbound traffic from the Internet to port 80.
Your network's public IP address range | TCP      | 22         | Allow Secure Shell (SSH) traffic from your company network.
Your network's public IP address range | TCP      | 3389       | Allow Remote Desktop Protocol (RDP) traffic from your company network.

- Outbound

Destination                                                      | Protocol | Port Range | Comments
---------------------------------------------------------------- | -------- | ---------- | --------
The ID of the security group for your MySQL database servers     | TCP      | 3306       | Allow outbound MySQL access to instances in the specified security group.
The ID of the security group for your Microsoft database servers | TCP      | 1433       | Allow outbound Microsoft SQL Server access to instances in the specified security group.

Here are the important points to understand about security groups:

- You can create up to 500 security groups for each Amazon VPC.
- You can add up to 50 inbound and 50 outbound rules to each security group. If you need to apply more than 100 rules to an instance, **you can associate up to five security groups with each network interface**.
- **You can specify allow rules, but not deny rules**. This is an important difference between security groups and ACLs.
- You can specify separate rules for inbound and outbound traffic.
- By default, no inbound traffic is allowed until you add inbound rules to the security group.
- By default, new security groups have an outbound rule that allows all outbound traffic. You can remove the rule and add outbound rules that allow specific outbound traffic only.
- **Security groups are stateful. This means that responses to allowed inbound traffic are allowed to flow outbound regardless of outbound rules and vice versa**. This is an important difference between security groups and network ACLs.
- Instances associated with the same security group can’t talk to each other unless you add rules allowing it (with the exception being the default security group).
- You can change the security groups with which an instance is associated after launch, and the changes will take effect immediately.

### Network Access Control Lists (ACLs)

A network access control lists(ACL) is another layer of security that acts as a stateless firewall on a subnet level. A network ACL is a numbered list of rules that AWS evaluates in order, starting with the lowest numbered rule, to determine whether traffic is allowed in or out of any subnet associated with the network ACL. Amazon VPCs are created with modifiable default network ACL associated with every subnet that allows all inbound and outbound traffic. When you create a custom network ACL, its initial configuration will deny all inbound and outbound traffic until you create rules that allow otherwise. You may set up network ACLs with rules similar with your security groups in order to add a layer of security to your Amazon VPC, or you may choose to use the default network ACL that does not filter traffic traversing the subnet boundary. Overall, every subnet must be associated with a network ACL.

#### Comparison of Security Group and Network ACLs

Security Group                                                              | Network ACL
--------------------------------------------------------------------------- | ------------
Operate at the instance level(first layer of defense)                       | Operates at the subnet level (second layer of defense) 
Supports allow rules only                                                   | Supports allow rules and deny rules
Stateful: Return traffic is automatically allowed, regardless of any rules  | Stateless: Return traffic must be explicitly allowed by rules
AWS evaluates all rules before deciding whether to allow traffic            | AWS processes rules in number order when deciding whether to all traffic
Applied selectively to individual instance                                  | Automatically applied to all instances in the associated subnets; this is a backup layer of defense, so you don’t have to rely on someone specifying the security group.

### Peering

An Amazon VPC peering connection is a networking connection between 2 Amazon VPCs that enables instances in either Amazon VPC to communicate with each other as if they are within the same network. You can create an Amazon VPC peering connection between your own Amazon VPCs or with an Amazon VPC in other AWS account within a single region. A peering connection is neither a gateway nor an Amazon VPN connection and does not introduce a single point of failure for communication.

Peering connections are created through a request/accept protocol. The owner of the requesting Amazon VPC sends a request to peer to the owner of the peer Amazon VPC. If the peer Amazon VPC is within the same account, it is identified by its VPC ID. If the peer VPC is within a different account, it is identified by Account ID and VPC ID. The owner of the peer Amazon VPC has one week to accept or reject the request to peer with the requesting Amazon VPC before the peering request expires.

An Amazon VPC may have multiple peering connections, and peering is a one-to-one relationship between Amazon VPCs, meaning 2 Amazon VPCs cannot have 2 peering agreements between them. Also, peering connections do not support transitive routing.

VPC peering connections do not support transitive routing.

<img src="./images/aws-vpc-peering.png" width="720"/>

VPC A has two peering connections with two different VPCs: VPC B and VPC C. Therefore, VPC A can communicate directly with VPCs B and C. Because peering connections do not support transitive routing, VPC A cannot be a transit point for traffic between VPCs B and C. In order for VPCs B and C to communicate with each other, a peering connection must be explicitly created between them.

Here are the important points to understand:

- You cannot create a peering connection between Amazon VPCs that have matching or overlapping CIDR blocks.
- **You cannot create a peering connection between Amazon VPCs in different regions**.
- Amazon VPC peering connections do not support transitive routing.
- You cannot have more than one peering connection between the same two Amazon VPCs at the same time.

### Network Address Translation (NAT) Instances and NAT Gateways

By default, any instance that you launch into a private subnet in an Amazon VPC is not able to communicate with the Internet through the IGW. This is problematic if the instances within private subnets need direct access to the Internet from the Amazon VPC in order to apply security updates, download patches, or update application software. AWS provides NAT instances and NAT gateways to allow instances deployed in private subnets to gain Internet access. For common use cases, we recommend that you use a NAT gateway instead of a NAT instance. The NAT gateway provides better availability and higher bandwidth and requires less administrative effort than NAT instances.

#### NAT Instance

A NAT instance is an Amazon Linux Amazon Machine Image(AMI) that is designed to accept traffic from instances within a private subnet,  translate the source IP address to the public IP address of the NAT instance, and forward the traffic to the IGW. In addition, the NAT instance maintains the state of the forwarded traffic in order to return response traffic from the Internet to the proper instance in the private subnet. These instances have the string amzn-ami-vpc-nat in their names, which is searchable in the Amazon EC2 console.

To allow instances within a private subnet to access Internet resources through the IGW via a NAT instance, you must do the following:
- Create a security group for the NAT with outbound rules that specify the needed Internet resources by port, protocol, and IP address.
- Launch an Amazon Linux NAT AMI as an instance in a public subnet and associate it with the NAT security group.
- Disable the Source/Destination Check attribute of the NAT.
- Configure the route table associated with a private subnet to direct Internet-bound traffic to the NAT instance (for example, i-1a2b3c4d).
- Allocate an EIP and associate it with the NAT instance.

This configuration allows instances in private subnets to send outbound Internet communication, but it prevents the instances from receiving inbound traffic initiated by someone on the Internet.

#### NAT Gateway

A NAT gateway is an Amazon managed resource that is designed to operate just like a NAT instance, but it is simpler to manage and highly available within an Availability Zone.

To allow instances within a private subnet to access Internet resources through the IGW via a NAT gateway, you must do the following:
- Configure the route table associated with the private subnet to direct Internet-bound traffic to the NAT gateway (for example, nat-1a2b3c4d).
- Allocate an EIP and associate it with the NAT gateway.

Like a NAT instance, this managed service allows outbound Internet communication and prevents the instances from receiving inbound traffic initiated by someone on the Internet.

**To create an Availability Zone-independent architecture, create a NAT gateway in each Availability Zone and configure your routing to ensure that resources use the NAT gateway in the same Availability Zone.**

### Virtual Private Gateways (VPGs), Customer Gateways (CGWs), and Virtual Private Networks (VPNs)

You can connect an existing data center to Amazon VPC using either hardware or software VPN connections, which will make Amazon VPC an extension of the data center. Amazon VPC offers two ways to connect a corporate network to a VPC: VPG and CGW.

A virtual private gateway (VPG) is the virtual private network (VPN) concentrator on the AWS side of the VPN connection between the two networks. A customer gateway (CGW) represents a physical device or a software application on the customer’s side of the VPN connection. After these two elements of an Amazon VPC have been created, the last step is to create a VPN tunnel. The VPN tunnel is established after traffic is generated from the customer’s side of the VPN connection.

#### VPC with VPN connection to a customer network

Here's an example of a single VPN connection between a corporate network and an Amazon VPC.

<img src="./images/aws-vpc-vpn.png" width="720"/>

You must specify the type of routing that you plan to use when you create a VPN connection. If the CGW supports Border Gateway Protocol (BGP), then configure the VPN connection for dynamic routing. Otherwise, configure the connections for static routing. If you will be using static routing, you must enter the routes for your network that should be communicated to the VPG. Routes will be propagated to the Amazon VPC to allow your resources to route network traffic back to the corporate network through the VGW and across the VPN tunnel.
Amazon VPC also supports multiple CGWs, each having a VPN connection to a single VPG (many-to-one design). In order to support this topology, the CGW IP addresses must be unique within the region.

Amazon VPC will provide the information needed by the network administrator to configure the CGW and establish the VPN connection with the VPG. The VPN connection consists of two Internet Protocol Security (IPSec) tunnels for higher availability to the Amazon VPC.

Following are the important points to understand about VPGs, CGWs, and VPNs:

- The VPG is the AWS end of the VPN tunnel.
- The CGW is a hardware or software application on the customer’s side of the VPN tunnel.
- You must initiate the VPN tunnel from the CGW to the VPG.
- VPGs support both dynamic routings with BGP and static routing.
- The VPN connection consists of two tunnels for higher availability to the VPC.

### IPV4 address classification

Traditional IP address classification divides IP addresses into Class A, Class B, Class C, Class D, and Class E

```
     ---------      -----      -----      -----
A  | 0******* | . | Host | . | Host | . | Host |
     ---------      -----      -----      -----
0 ~ 126

127 (01111111) is a IP address of Class A，
it is reserved for loopback testing, so it can not be used in the network.

     ---------      --------      -----      -----
B  | 10****** | . | NetWork | . | Host | . | Host |
     ---------      --------      -----      -----
128 ~ 191

     ---------      --------      --------      -----
C  | 110***** | . | NetWork | . | NetWork | . | Host |
     ---------      --------      --------      -----
192 ~ 223 
```

Here are the private IP addresses

Class   | Private Address Range
------- | ----------------------------
A       | 10.0.0.0    ~ 10.255.255.255
B       | 172.16.0.0  ~ 172.31.255.255
C       | 192.168.0.0 ~ 192.168.255.255

Generally, 
- private ip addresses are used for the internal network.
- public ip addresses are used for the external network.
