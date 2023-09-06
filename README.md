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

[TOC]

<span class="fontColor2">*Leased line or IPSec VPN? These two terminologies always arise when you want to bridge two locations. In this cloud-massive era, IPSec VPN could still be easily deployed in any architecture because what it needs in essence is the Internet; however, when we turn to the leased line, what we could do since we have nothing in the on-premises?*</span>

# <span class="fontColorH2">*How Did We Adopt In The Past?*</span>

Leased line or IPSec VPN? The adoption primarily depends on one of the following factors, some of them, or even all of them if technologies are met, e.g. protocol/feature:

- **Permanent/temporary use** - If the scenario is a PoC, you typically would not prefer ordering a dedicated circuit to verify your requirement from either the schedule aspect, the cost aspect, or both. However, if the scenario is opposite, which means it is a production environment, the Internet-based VPN typically is not the first priority for consideration.
- **With/without SLA** - Not every case you would need a commercial agreement to safeguard your business even if the scenario is a production environment in that the requirement highly depends on what is the magnitude of the service.
- **Cost** - Every solution, no matter it is open-source-based or commercial-based, could be divided into two pieces: CapEx and OpEx. The CapEx primarily focuses on how much budget I need to scope and how much expenditure I need to pay. The OpEx primarily emphasises what needs to beware if leveraging any existing resource, e.g. capacity or reliability.

When we turn to technical requirements, e.g., do we have sufficient infrastructure resources to deliver (router/switch/firewall), do those resources have sufficient licenses to support (BGP/GRE/IPSec), we typically do not concern them too much because they are easily qualified by the existing environment.

But, what could we do for the leased line in the cloud world?

# <span class="fontColorH2">*Is Anything Changed Nowadays?*</span>

Intrinsically, you do not need to panic because the leased line architecture is still out there, it just functions differently. If so, you may be interested in what are the discrepancies between the eras. [Software-defined Cloud Interconnect (SDCI)](https://www.gartner.com/reviews/market/software-defined-cloud-interconnects-sdci) is the answer. The SDCI architecture provides a more agile, flexible, and modernised model to link up with any CSP environment. Compared with the traditional leased line model, the SDCI architecture has the following spotlights:

- **Self-provisioning** - In the past, you were able to follow up all the tasks on your side until the circuit was delivered; typically, the lead time took around 2-4 weeks. Based on the SDCI framework, nothing needs to be waited because everything could be manipulated by yourself. What you need to do basically is, choose the PoP where is close to your business and the required capacity, feed the credentials from the CSP you specified, and deploy. You could even integrate the existing deployment pipeline with the SDCI platform to achieve the infrastructure-as-code (IaC) principle. 
- **Commitment-free** - In the past, one of the key factors in the order process was bandwidth commitment, which meant you had to pay for the bandwidth you did not completely consume. The charge model of each SDCI component is subscription-based as well as those CSPs and SaaS vendors; you would only need to pay for how many resources you actually consume. 
- **Carrier-neutral** - Before Google announced its [Cross-Cloud Interconnect](https://cloud.google.com/blog/products/networking/announcing-google-cloud-cross-cloud-interconnect) offering, there was no way to bridge various clouds via a single cloud solution because each of them is proprietary. However, the SDCI framework crosses this boundary; it acts as an octopus that is capable of integrating every cloud with it simultaneously.

Therefore, let us look at [Megaport](https://www.megaport.com/services/) who is one of the well-known SDCI solution providers in the market.

# <span class="fontColorH2">*Solutions Outline*</span>

Leveraging either the CSP-managed VPN service or the 3^rd^ party firewall instance-based VPN to bridge the clouds is not really a concern if you do not have too many sites; however, it would be tedious and overwhelming once you have tons of environments that need to be managed. In the era of everything seeking out efficiency and as-code gives Megaport an opportunity to demonstrate its capability which is composed of three key offerings: **Port**, **MCR**, and **MVE**.

## <span class="fontColorH3">*Hybrid Cloud*</span>

<u>[Port](https://www.megaport.com/services/cloud-connectivity/) is delivered as a physical circuit that is capable of linking multiple clouds.</u> As a matter of fact, this offering is quite common, especially from the NSP's aspect, e.g. [PCCW](https://www.consoleconnect.com/services/layer-2/), [Equinix](https://www.equinix.com/products/digital-infrastructure-services/equinix-fabric), hence I do not want to dig it too much detailed.

![Port](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sqxz9i2snalzwolytcn8.png)

## <span class="fontColorH3">*Cloud-to-Cloud*</span>

<u>[MCR (Megaport Cloud Router)](https://www.megaport.com/services/megaport-cloud-router/) functions as a concentrated exchange point that is capable of managing the routing across multiple clouds.</u> Unlike Port adopts the layer-2 design to bridge the on-premises with the clouds (the routing relies on the network infrastructure in on-premises), MCR adopts the layer-3 design for every cloud-to-cloud scenario (the routing is totally handled by itself); for this reason, you could construct a hub-spoke transport architecture on either per-region, per-cloud, or even both basis.

![MCR](https://docs.megaport.com/mcr/img/multicloud.png)

Let us slow down our pace for a while. You are probably aware of one single term - **VXC (Virtual Cross Connect)** because it is associated with both Port and MCR. Actually, VXC is what we are looking for the leased line in the cloud world. <u>Each VXC represents a connector of the destination</u>, e.g. Amazon, Microsoft, or Google. In addition, each VXC could not function without associating with Megaport's services; otherwise, it is just an object.

![MCCA via Megaport](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1lghtty0pzbo24un93gg.png)

You are probably curious about how could each VXC be deemed a leased line? The answer is quite straightforward because every single VXC behind the scenes is a CSP-managed interconnect resource, e.g. AWS Direct Connect, Azure ExpressRoute, or GCP Cloud Interconnect. The only discrepancy is that you are not responsible for operating any network infrastructure in on-premises, Megaport takes over this ownership.

### <span class="fontColorH4">*Technical Deep-dive*</span>

I observed one thing that is worth keeping in mind during the PoC is how BGP Prefix Filter works on MCR because it functions differently when compared with how BGP behaves in general.

![BGP Prefix Filter](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mp4iim1ni6a2dfxprt5b.png)

Import Prefix Filter does not mean which prefix is received from your BGP peer is allowed to feed into your route table, instead, <u>it means which CIDR of the VPC/VNet is associated with this BGP connection</u>. As you see from my PoC architecture, there are two CIDRs (_10.150.16.0/20_, _10.150.224.0/20_) from my AWS account. From MCR's aspect, those two prefixes must be allowed to import, otherwise, you will not see any prefix in the Received section.

![PoC](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qv2w40onas9riiiewbvg.png)

![Import Filter](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lcgvl4h06pqumektzd95.png)

Export Prefix Filter does not mean which origin prefix you want to advertise to your BGP peer, instead, <u>it means which prefix you receive from your BGP peer is allowed to propagate; in other words, which prefix is able to transit over this BGP connection</u>. Looking at my PoC architecture again, there is a single CIDR (_10.160.224.0/20_) from my Azure account. From MCR's aspect, this prefix must be allowed to export, otherwise, you will not see any prefix in the Advertised section; in other words, you will not any prefix in your VPC/VNet Route Table, either.

![Export Filter](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o01cl5myk8r4gr091ofn.png)

## <span class="fontColorH3">*Enhanced SD-WAN*</span>

[MVE (Megaport Virtual Edge)](https://www.megaport.com/services/megaport-virtual-edge/) is based on the SD-WAN foundation to provide an optimised path between the on-premises and the CSP/SaaS platforms; in other words, MVE has a prerequisite of [supported vendor platforms](https://docs.megaport.com/mve/).

![MVE](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yj8248wkwk895mqb4nnk.png)

You may wonder why Megaport is able to optimise the network path for SaaS platforms via MVE? The answer is composed of two pieces: 

- First of all, the Megaport-managed network backbone, which is close to the colocations/datacentres where those CSPs locate. This factor is a prerequisite for most SD-WAN solution providers in the market under the hood.
- Secondly, a comprehensive ecosystem from Megaport Marketplace, the most decisive factor from my perspective. Every certified service provider could be represented by a VXC and associated with MVE.

Apparently, since everything is under Megaport's umbrella, the user experience could be eminently optimised.

# <span class="fontColorH2">*Wrap-up*</span>

One of the reasons why Megaport's offerings are fascinating is its highly elastic design from my perspective. Each service on the bottom could be represented by a VXC and each VXC could be associated with either Port, MCR, or MVE. Imagine that we are in the Lego world, every tier (VXC, Port, MCR, and MVE) could be deemed the Lego blocks, and what those blocks will present that depending on your imagination (requirement).

![VXC Types](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4t4fkwwh55ymhujf0iak.png)

Although one of the transitions in the cloud era is <u>the Internet is a new network</u>, when we take either the business drivers (e.g. SLA) or the operational concerns (e.g. security) into account, the SDCI solutions are worth evaluating and embracing for your modernised service framework nonetheless.

:::info
:date: Published on 6^th^ September 2023. This post is also published in my [dev.to](https://dev.to/@terrencec51229) space.
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}
