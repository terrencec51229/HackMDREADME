<style>
.fontColor {
  color: #FF5733;
}
.fontColor2 {
  color: #96A5A5;
}
.fontColorH2 {
  color: #FF960F
}
.fontColorH3{
  color: #FFB432
}
.fontColorH4{
  color: #F08080
}
.fontFace {
  font-weight: Bold;
  font-style: Italic;
}
table th:first-of-type {
    width: 5%;
}
table th:nth-of-type(2) {
    width: 10%;
}
table th:nth-of-type(3) {
    width: 5%;
}
</style>

# The Evolution of Cloud Networking on AWS

[TOC]

## <span class="fontColorH2">Native Cloud</span>

Typically, from the architecture design perspective, you would not consider putting everything together within a single VPC. In most cases, the resources would be segregated from the role of the environment. For instance, Production, Developer, and Testbed. Although we certainly know what granular management that [tagging](https://d1.awsstatic.com/whitepapers/aws-tagging-best-practices.pdf) rendered, however, the division is required for the non-technical considerations, which is more about the management purpose instead.

As a result, the whole environment would be composed of a bunch of VPCs and even across different accounts/organisations. Therefore, AWS has rendered a couple of managed services to bridge all of them together.

:::success
:bulb: *Since this post much focuses on the VPC-level solution rather than the Endpoint-level solution, hence [PrivateLink](https://aws.amazon.com/blogs/aws/aws-privatelink-update-vpc-endpoints-for-your-own-applications-services/) is not covered here.*
:::

### <span class="fontColorH3">VPC Peering</span>

If the whole environment is composed of two VPCs then the quickest/easiest way is to leverage VPC Peering. It could be implemented via `VPC > Virtual Private Cloud > Peer Connections` and [Inter-region VPC Peering](https://aws.amazon.com/vpc/faqs/) is available globally in all commercial regions except for the China regions.

![peering-intro-diagram](https://i.imgur.com/jhUhQ83.png)

Other than the prefixes between VPCs cannot overlap, most importantly, <span class="fontColor">VPC Peering does not allow any transitive communication due to its design</span> (as the 1^st^ image below). Imagine that VPC Peering is designed as a back door instead of an IXP (Internet exchange point), hence if the requirement is to make all the resources that across different VPCs are able to liaise with each other, then the full-mesh architecture is required (as the 2^nd^ image below).

![transitive-peering-diagram](https://i.imgur.com/Y2Sm1nI.png)

![three-vpcs-peered-diagram](https://i.imgur.com/UzdxSwD.png)

> :mag: Visit [What is VPC peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) for more details.

### <span class="fontColorH3">Transit VPC</span>

If transitive communication is required (not only just between VPCs but also with external environments) then two methodologies could make it take place: **Transit VPC** and **Transit Gateway**. In other words, they leverage the architecture of Route Reflector (RR) to get rid of the full-mesh requirement.

In effect, <span class="fontColor">Transit VPC is a concept instead of a packaged service</span>. In essence, it is formed by a bunch of the Site-to-Site VPN tunnels or BGP over those tunnels between VPCs. It could be implemented via `VPC > Virtual Private Network (VPN)` and it is available globally in all commercial regions.

![Transit VPC](https://i.imgur.com/Q4nW6ji.png)

>  :mag: Because it is a concept, hence it could be fulfilled by the 3^rd^ party solutions. Essentially, you could adopt any on-premises solution that you have been familiar with if it is able to launch on AWS; you could visit [Transit VPC Solutions in AWS Marketplace](https://aws.amazon.com/marketplace/solutions/infrastructure-software/transit-VPC) for more details as well.

### <span class="fontColorH3">Transit Gateway</span>

Unlike Transit VPC is a concept that has to manage by yourself, Transit Gateway (TGW) is a native service that is built and managed by AWS completely. The following image is a high-level overview due to TGW could be applied in a more agile way as well as the 2^nd^ diagram.

![Transit Gateway](https://i.imgur.com/WAKWll0.png)

There are three kinds of TGW Attachments that could be associated with TGW;

1. ++VPC++ - Fasten the VPC which locates in the same region with TGW. It is used for the AWS environment only.
2. ++VPN++  -  Establish the IPSec VPN tunnels or the BGP adjacency over those tunnels with TGW. It could be used for either the AWS environment or external environments. However, there is one thing that needs to keep in mind, its capacity. <span class="fontColor">Each VPC/Peering Attachment has 50 Gbps capacity, but each VPN tunnel can only afford up to 1.25 Gbps volume of traffic.</span>
3. ++Peering++ -  Establish the API adjacency with TGWs which locate in either the same region or different regions. It is used for the AWS environment only.

All of them which bind with the same TGW form a TGW Route Table. What does a TGW Route Table differ from a VPC Route Table? That is a TGW Route Table acts as a BGP table which involves all feasible routes. However, the TGW Route Table does not account for the actual routing, the VPC Route Table handles it instead. All the above-mentioned features could be implemented via `VPC > Transit Gateways` and they are available globally in most commercial regions.

![Overview](https://i.imgur.com/OPxxbP1.png)

There is one more methodology that could more efficiently utilise and concentrate TGW, which is [Resource Access Manager (RAM)](https://aws.amazon.com/ram/faqs/). TGW is one of the resources that could be shared with multiple accounts even if they are governed by different Organizations. Typically, TGW would be deployed per region and leverage RAM to centrally associate with VPCs and link up with other TGWs.

![RAM](https://i.imgur.com/MSUqUXY.png)

![Shared Resources](https://i.imgur.com/crTBcUS.png)

>  :mag: Visit [Performance and limits](https://aws.amazon.com/transit-gateway/faqs/) for more guidance and [What is a transit gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) for more details.

### <span class="fontColorH3">Comparison of The Transit Family</span>

Like the situation of NAT Instance and NAT Gateway, the self-managed service shall only be considered and adopted when none of the native services is acceptable or available. If there is no significant driver to push managing the entire transport by yourself then I do not see any stumbling block to refuse TGW.

![Comparison](https://i.imgur.com/qNx8tzZ.png)

## <span class="fontColorH2">Hybrid Cloud</span>

In most cases, the hybrid cloud strategy is primarily formed by the cold DR requirement (replicate what we have in the on-premises environment to the cloud) and the go-cloud assessment (is the cloud ecosystem suitable for the organisation). Besides leveraging existing Internet connection to bridge all the resources within a VPC via the Site-to-Site VPN, what is the package that AWS offers with more reliability than the Internet?  The answer is [Direct Connect](https://aws.amazon.com/directconnect/features/?nc=sn&loc=2), a dedicated circuit that could link up the whole AWS world with your on-premises environment.

Direct Connect is just a medium, it cannot function by itself; it relies on integrating with **Virtual Private Gateway**, **Direct Connect Gateway**, or **Transit Gateway** to be functioning. Because of this reason, <span class="fontColor">Direct Connect is a layer 3 channel and it only supports BGP for routing exchange</span>.

Most well-known NSPs offer CloudConnect-as-a-service already, e.g. [PCCW](https://www.consoleconnect.com/clouds/connect-to-amazon-web-services/), [GCX](https://www.globalcloudxchange.com/about-2/cloud-networking/cloud-x-fusion/), and [Megaport](https://www.megaport.com/services/amazon-web-services/). One thing in common between those NSPs is they offer an all-in-one circuit which is able to link up multiple CSPs together by leveraging their IP/MPLS transport backbone. Another benefit we could get from those NSPs is that their point-of-presences (PoPs) are much more than AWS.

> :mag: Visit [Technical](https://aws.amazon.com/transit-gateway/faqs/) for more guidance and [What is AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html) for more details.

### <span class="fontColorH3">Virtual Private Gateway</span>

Each VPC has its own Virtual Private Gateway (VPG) that associates with a unique Autonomous System Number (ASN) which is defined via `VPC > Virtual Private Network (VPN) > Virtual Private Gateways`. Therefore, the BGP adjacency relies on this ASN to be established. Since the ASN has to be unique from the BGP aspect, for this reason, <span class="fontColor">you need to ensure that it is not duplicated across VPGs albeit AWS Console/CLI does not restrict this behaviour</span>.

The VPG could be used for bridging either the AWS environment or external environments. As the matter of fact, the VPG acts as a major component for constructing a Transit VPC architecture if none of the 3^rd^ party solutions is taken into account.

![Gateway type](https://i.imgur.com/qZeBNio.png)

![Virtual Private Gateway](https://i.imgur.com/6sIl6SN.png)

### <span class="fontColorH3">Direct Connect Gateway</span>

If you do not have too many VPCs that need to be linked up with your on-premises environment then using VPGs could be an option. However, if the situation is opposite (you have tons of VPC that need to be bridged) then concentrating all of them would be a more ideal way for management. That is why Direct Connect Gateway (DXG) comes up. <span class="fontColor">The DXG acts as a global container for grouping all the VPGs and TGWs per account.</span>

Each VPG would be behind the DXG, therefore, it does not matter if the ASN conflicts between them due to only DXG's ASN could be seen by the external world. It is defined via `Direct Connect > Direct Connect Gateways`. However, <span class="fontColor">you still need to ensure that each ASN is not duplicated across DXGs albeit AWS Console/CLI does  not restrict this behaviour as well</span>. Unlike VPG, the DXG is used for linking up with the on-premises environment only.

![Gateway type](https://i.imgur.com/lZ03pp1.png)

![Direct Connect Gateway](https://i.imgur.com/2nmVZCi.png)

### <span class="fontColorH3">Transit Gateway</span>

The same concept to the VPG and DXG, if you do not have too many DXGs that need to be linked up with your on-premises environment then using it could be an option. However, if the situation is opposite or any additional manipulation is required, e.g., to filter or summarise the prefixes when propagating, then the TGW is the only option.

As elaboration in `Native Cloud \ Transit Gateway`, the TGW could be used for bridging either the AWS environment or external environments. Refer to that section again for more details.

![Gateway type](https://i.imgur.com/JfvpTPf.png)

## <span class="fontColorH2">Conclusion</span>

Typically, non-transitive communication is rare especially when the entire application development composes of the on-premises and multiple CSPs resources. In this manner, choosing the right service or platform to bridge all of them together is certainly important.

By the end of 2020, the most powerful service that AWS provides is TGW definitely. However, it does not mean that the rest of the services are useless in that the adoption depends on various considerations.

> Published on 29^th^ December 2020. 

:::info
###### tags: `AWS` `Architecture` `NativeCloudNet` `HybridCloudNet` `VPCPeering` `TransitVPC` `TransitGateway` `DirectConnect`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}