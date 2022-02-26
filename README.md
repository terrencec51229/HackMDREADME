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

Typically, from the architecture design perspective, you would not consider to put everything together within an VPC. In most cases, the resources would be segregated from the role of the environment. For instance, Production, Developer, and Testbed. Although we certainly know what granular management that [tagging](https://d1.awsstatic.com/whitepapers/aws-tagging-best-practices.pdf) rendered, however, the devision is required for the non-technical considerations, that is more about the management purpose instead.

As a result, the whole environment would be composed of a bunch of VPCs and even across different accounts/organizations. Therefore, AWS has rendered a couple of managed services to bridge all of them together.

:::success
:bulb: *Since this post much focuses on the VPC-level solution rather than the Endpoint-level solution so that [PrivateLink](https://aws.amazon.com/blogs/aws/aws-privatelink-update-vpc-endpoints-for-your-own-applications-services/) is not covered here.*
:::

### <span class="fontColorH3">VPC Peering</span>

If the whole environment is composes of two VPCs then the quickest/easiest way is to leverage VPC Peering. It could be implemented via `VPC > Virtual Private Cloud > Peer Connections` and [Inter-region VPC Peering](https://aws.amazon.com/vpc/faqs/) is available globally in all commercial regions except for the China regions.

![peering-intro-diagram](https://i.imgur.com/jhUhQ83.png)

Other than the prefixes between VPCs cannot overlap, the most importantly, <span class="fontColor">VPC Peering does not allow any transitive communications due to its design</span> (as the 1^st^ image below). Imagine that VPC Peering is designed as a back-door instead of an IXP (Internet exchange point), hence if the requirement is to make all the resources that across different VPCs are able to liaise with each other, then the full-mesh architecture is required</span> (as the 2^nd^ image below).

![transitive-peering-diagram](https://i.imgur.com/Y2Sm1nI.png)

![three-vpcs-peered-diagram](https://i.imgur.com/UzdxSwD.png)

> :mag: Visit [What is VPC peering](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html) for more details.

### <span class="fontColorH3">Transit VPC </span>

If the transitive communication is required (not only just between VPCs but also with external environments) then there are two methodologies could make it take place: **Transit VPC** and **Transit Gateway**. In other words, they leverage the architecture of Route Reflector (RR) to get rid of the full-mesh requirement.

In effect, <span class="fontColor">Transit VPC is a concept instead of a packaged service</span>. In essence, it is formed by a bunch of the Site-to-Site VPN tunnels or BGP over those tunnels between VPCs. It could be implemented via `VPC > Virtual Private Network (VPN)` and it is available globally in all commercial regions.

![Transit VPC](https://i.imgur.com/Q4nW6ji.png)

>  :mag: Because it is a concept so that it could be fulfilled by the 3^rd^ party solutions. Essentially, you could adopt any of the on-premises solutions that you have been familiar with if they are able to launch on AWS. Visit [Transit VPC Solutions in AWS Marketplace](https://aws.amazon.com/marketplace/solutions/infrastructure-software/transit-VPC) for more details.

### <span class="fontColorH3">Transit Gateway</span>

Unlike Transit VPC is a concept and has to manage by yourself, Transit Gateway (TGW) is a native service that built and managed by AWS completely. The following image is a high-level overview due to TGW could be applied in more agile way as the 2^nd^ diagram.

![Transit Gateway](https://i.imgur.com/WAKWll0.png)

There are three kinds of TGW Attachments for bridging the resources with TGW.
1. ++VPC++ - Fasten the VPC which locates in the same region with TGW. It is used for pure AWS environment only.
2. ++VPN++ - Establish the IPSec VPN tunnels or the BGP adjacency over those tunnels with TGW. It could be used for either pure AWS or external environments. However, there is one thing needs to keep in mind, its capacity. <span class="fontColor">Each VPC/Peering Attachment has 50 Gbps capacity, but each VPN tunnel can only afford up to 1.25 Gbps volume of the traffic.</span> 
3. ++Peering++ - Establish the BGP adjacency with TGW which locates in different region. It is used for pure AWS environment only.

All of them which bind with the same TGW form an TGW Route Table. What does it differ from VPC Route Table? That is TGW Route Table acts as an BGP table which involves all the feasible routes. However, it does not account for the actual routing of VPC, it is responsible for VPC Route Table instead. All the above-mentioned features could be implemented via `VPC > Transit Gateways` and they are available globally in most of commercial regions.

![Overview](https://i.imgur.com/OPxxbP1.png)

There is one more methodology could more efficiently utilize/concentrate TGW and that is [Resource Access Manager (RAM)](https://aws.amazon.com/ram/faqs/). TGW is one of the resources that is able to share with multiple accounts. Typically, TGW shall be provisioned in the per-region basis and leverage RAM to centrally bind VPCs and link other TGWs.

![RAM](https://i.imgur.com/MSUqUXY.png)

![Shared Resources](https://i.imgur.com/crTBcUS.png)

>  :mag: Visit [Performance and limits](https://aws.amazon.com/transit-gateway/faqs/) for more overall guidances and [What is a transit gateway](https://docs.aws.amazon.com/vpc/latest/tgw/what-is-transit-gateway.html) for more in-depth details.

### <span class="fontColorH3">Comparison of The Transit Family </span>

Like the situation of NAT Instance and NAT Gateway, the self-managed service shall only be adopted when none of native services are acceptable or available. If there is no significant driver to push managing the entire transport by yourself then I do not see any stumbling blocks to stop embracing TGW.

![Comparison](https://i.imgur.com/qNx8tzZ.png)

## <span class="fontColorH2">Hybrid Cloud</span>

In most cases, the hybrid cloud strategy is primarily formed by the cold DR requirement (replicate what we have in the on-premises environment to the cloud) and the go-cloud assessment (is the cloud ecosystem suitable for the organization). Besides leverage existing Internet connection to bridge any resources within VPCs via Site-to-Site VPN, what is the package that AWS offers with more and more reliability than Internet is [Direct Connect](https://aws.amazon.com/directconnect/features/?nc=sn&loc=2), a dedicated circuit links up the whole AWS world with your on-premises environment.

Direct Connect is just a medium, it cannot function by itself so that it relies on the integration of **Virtual Private Gateway**, **Direct Connect Gateway**, or **Transit Gateway** to be functioned. Because of that, <span class="fontColor">Direct Connect is a pure L3 link and only supported BGP for the routing exchange</span>.

Most of well-known ISPs have offered CloudConnect-as-a-service already, such as [PCCW](https://www.consoleconnect.com/clouds/connect-to-amazon-web-services/), [GCX](https://www.globalcloudxchange.com/about-2/cloud-networking/cloud-x-fusion/), and [Megaport](https://www.megaport.com/services/amazon-web-services/). One thing in common in between is they offer all-in-one circuit which is able to link up multiple CSPs together by leveraging their MPLS-VPN transport backbone. Another benefit is their locations of Customer Edge (CE) are much more spreaded than AWS provided.

> :mag: Visit [Technical](https://aws.amazon.com/transit-gateway/faqs/) for more overall guidances and [What is AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html) for more in-depth details.

### <span class="fontColorH3">Virtual Private Gateway</span>

Each VPC has its own Virtual Private Gateway (VPG) that binds with an unique Autonomous System Number (ASN) which is defined via `VPC > Virtual Private Network (VPN) > Virtual Private Gateways`. Therefore, the BGP adjacency relies on it to be established. Since ASN has to be unique from the BGP aspect, for this reason, <span class="fontColor">you need to ensure that it is not duplicate across VPGs albeit AWS Console/CLI do not restrict this behavior</span>.

VPG could be used for bridging either pure AWS or external environments. As the matter of fact, VPG acts as a major component for constructing Transit VPC if none of the 3^rd^ party solutions are taken into account.

![Gateway type](https://i.imgur.com/qZeBNio.png)

![Virtual Private Gateway](https://i.imgur.com/6sIl6SN.png)

### <span class="fontColorH3">Direct Connect Gateway</span>

If you do not have too many VPCs need to be linked up with your on-premises environment(s) then using VPGs could be an option. However, if the situation is opposite (you have tons of VPCs need to be bridged) then concentrating them would be a more ideal way for management. That is why Direct Connect Gateway (DXG) comes up. <span class="fontColor">DXG acts as a global container for grouping VPGs and TGWs per account.</span>

Since VPG would be behind DXG, therefore, it does not matter if ASN conflicts in between due to only DXG's ASN could be seen by external world. It is defined via `Direct Connect > Direct Connect Gateways`. However, <span class="fontColor">you still need to ensure that it is not duplicate across DXGs albeit AWS Console/CLI do not restrict this behavior</span>.

Unlike VPG, DXG is used for linking up with the on-premises environment(s) only.

![Gateway type](https://i.imgur.com/lZ03pp1.png)

![Direct Connect Gateway](https://i.imgur.com/2nmVZCi.png)

### <span class="fontColorH3">Transit Gateway</span>

The same concept of VPG/DXG, if you do not have too many DXGs need to be linked up with your on-premises environment(s) then using it could be an option. However, if the situation is opposite or an additional manipulation is required (for instance, to filter or summarize the prefixes when propagating), then TGW is the only option.

As the in-depth elaration in `Native Cloud \ Transit Gateway`, TGW could be used for bridging either pure AWS or external environments. Refer to that section again for more details.

![Gateway type](https://i.imgur.com/JfvpTPf.png)

## <span class="fontColorH2">Conclusion</span>

Typically, the non-transitive communication is rare especially when the entire application development composes of the on-premises and multiple CSPs resources. In this manner, choose the right service(s) or platform(s) to bridge all of them together is certainly important.

By the end of 2020, the most powerful service that AWS provided is TGW definitely. However, it does not mean the rest of the services are useless in that the adoption depends on various consideratios.

:::info
###### tags: `AWS` `Architecture` `NativeCloudNet` `HybridCloudNet` `VPCPeering` `TransitVPC` `TransitGateway` `DirectConnect`
:::

{%hackmd BJrTq20hE %}