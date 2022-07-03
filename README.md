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

# Enhanced Management of Multi-cloud Networking and Security

[TOC]

## <span class="fontColorH2">Business Model Nowadays</span>

As of today, more and more organizations have either evaluated their cloud adoption strategy or investigated the requirement of the multi-cloud; obviously, the cloud world is no longer blurred.

Other than Application Modernization (serverless or containerization), another topic that has been paid attention is the multi-cloud networking. The multi-cloud networking has been limelighted by two threads;

- Multi-cloud network transit.
- Multi-cloud network management.

From the technical standpoint, the multi-cloud network transit is covered with the multi-cloud network management; when turning to the commmercial standpoint, they are supported by a different way.

- In terms of the multi-cloud network transit, it much emphasizes that ++your cloud-to-cloud communication is passed through a more elastic and reliable (private) transport backbone++.
- In terms of the multi-cloud network management, it means a lot; typically, it much emphasizes ++how to orchestrate your own network transport by abstracting each CSP's console++ (that is another format of the multi-cloud network transit) and ++how to make every dark side is visible++. The whole offering of the multi-cloud network management is very comprehensive due to it basically involves network security as well. Because of this reason, I would like to dig into this topic a bit more. 

## <span class="fontColorH2">What Are The Absences</span>

We have been conveyed that to spend your time on developing the applications instead of waste your time on managing the infrastructure in the cloud world; however, when turning to the network management, the reality is that we need a more flexible and transparent way to manipulate self-build network transport nonetheless. There is no doubt that AWS has much strengthened on the network transit feature than Azure and GCP do. You might be curious why, and the answer is [Transit Gateway](https://hackmd.io/@terrencec51229/the-evolution-of-cloud-networking-on-aws#Transit-Gateway); a fully managed service to construct your own cloud network transit. If you want to construct a cloud network transit on either Azure or GCP, you need to build a [Transit VPC](https://hackmd.io/@terrencec51229/the-evolution-of-cloud-networking-on-aws#Transit-VPC-) beforehand.

So, what are the absences of Transit Gateway albeit it looks quite faultless?

- In terms of the deployment, it is very straightforward to bridge everywhere within AWS, but you still cannot avoid managing additional VPN tunnels if the target is out of AWS.
- In terms of the visibility, although [Network Manager](https://docs.aws.amazon.com/vpc/latest/tgwnm/what-is-network-manager.html) gives an outline of what the whole network topology looks like, the thing is what it provided is not really helpful due to the information it presented is not human-readable (without custom tags).

![Network Manager](https://i.imgur.com/v8T6LM8.png)

Other than that, there are several pain points from the aspect of routine operations as well;

- What is the VPC that an IP prefix belongs to or a workload resides? Especially your deployments across numerous accounts and regions.
- Is there any way that we could centrally manage the egress access across the accounts, regions, and even clouds?
- How could we avoid the conflict of the CIDR across the accounts, regions, and even clouds?

For those reasons, let us look at the protagonist in this post - [Aviatrix](https://aviatrix.com/features/), who orchestrates and operates your cloud networks with the agility and simplicity.

## <span class="fontColorH2">How To Efficiently Operate Your Clouds</span>

The entire Aviatrix offering is supported by three components and they will be introduced in-depth in the later sections;

- Aviatrix Controller.
- Aviatrix (Transit/Spoke) Gateway.
- CoPilot.

When turning to the Aviatrix solution architecture, it is designed by the SDN (software-defined networking) framework; hence all the deployments are taken place on the Controller side (control plane), and all the traffic manipulations are handled by the Gateway side (data plane). Because of this segregation, all the Gateways are able to function completely without any influence once the Controller is out of service.

![Aviatrix Transit Architecture](https://i.imgur.com/YlDNJJz.png)

### <span class="fontColorH3">Global Transit Network</span>

As its name, ++what Global Transit Network does is to construct your own network transport across the clouds++. Essentially, Global Transit Network is the foundation of the Aviatrix offerings; everything gets started from it. Global Transit Network is primarily composed of Aviatrix Controller, Aviatrix Transit Gateway, and Aviatrix Spoke Gateway. All the Gateways are provisioned via the Controller and all of their details e.g. the image version and the configuration are fully managed by the Controller. 

Let us get started from the following screenshot which is taken from one of the publications of [Cisco Press](https://www.ciscopress.com/articles/article.asp?p=2448489). Typically, most of the enterprise network architectures adopt this way; all the end-users connect to the backbone network via the access layer and the routing decision is happened on the core/distribution layer. Why I accentuate it in the cloud networking topic? Because Global Transit Network inherits this design as well!

![Two-tier Model](https://i.imgur.com/OhGopEh.jpg)

In terms of the functionality, each Gateway acts the role below;

- Aviatrix Spoke Gateway accounts for the same role of the access layer does. What Spoke Gateway does is quite simple, ++it associates with either a single VPC or VNet and then takes over their route tables++. 
- Aviatrix Transit Gateway accounts for the same role of the core/distribution layer does. As you could imagine that ++Transit Gateway acts as the next-hop of each Spoke Gateway++.

++By default, each Aviatrix Transit Gateway could only associate with a single site which could be an Aviatrix Spoke Gateway or a pair ones behind the scenes; this nature is caused by the `Connected Transit` feature is disabled by default.++ Although it gives you a very granular and decoupled space for any adjustment you want to fulfill, it is too costly, and this extra expenditure may not be really necessary as well.

![Global Transit Network Standard Overview](https://i.imgur.com/6jplssp.png)

:::spoiler <span class="fontColor2">How To Enable Connected Transit</span>
![Connected Transit](https://i.imgur.com/4B3PF3a.png)
:::

Once the `Connected Transit` feature is enabled, each Aviatrix Transit Gateway is capable of associating with multiple sites; as a result, the whole architecture will like below which is more close to the actual world.

![Global Transit Network Connected Transit Overview](https://i.imgur.com/lknpDHU.png)

You could even aggregate all the Aviatrix Spoke Gateways by either a region or a CSP consideration and there is no fixed design as well. As the matter of fact, the design of what the whole Global Transit Network will look like is extremely depended on your imagination.

![Global Transit Network Aggregated Connected Transit Overview](https://i.imgur.com/TvXhCaY.png)

#### <span class="fontColorH4">:label:TGW Orchestrator</span>

Two points that I mentioned from the beginning;

- AWS has spent more efforts on the network transit feature when compared with the other Big-3 members.
- One of the primary goals of Aviatrix is to orchestrate your cloud networks with the simplicity.

As a result, that is why TGW Orchestrator is here.

First of all, ++this feature supports AWS only++ due to Transit Gateway is one of its VPC components. Secondly, this feature emphasizes segmentation which is called Security Domain in the Aviatrix world. ++Segmentation does support to be implemented by Aviatrix Transit Gateway as well++, hence there is no need to worry about if your business is on either Azure or GCP.

In essence, ++a Security Domain means a VRF++. ++Each Transit Gateway Route Table is the representative of Security Domain++ and you could use both Association and Propagation to manage and ensure the isolation. Let me use the following screenshots for the explanation.

- The VPC <span class="fontColor">*Aviatrix_Prod*</span> associates with its own Transit Gateway Route Table <span class="fontColor">*Aviatrix_Prod*</span>.
- Before any change takes place, the Transit Gateway Route Tables <span class="fontColor">*Aviatrix_FireNet_Domain*</span> and <span class="fontColor">*Aviatrix_Prod*</span> are isolated; that is none of the resources are able to communicate with each other.
- When connected those two Transit Gateway Route Tables, all their resources are able to communicate with each other. One thing needs to keep in mind is that the  propagation via TGW Orchestrator is the entire CIDR of the VPC; that is you need to leverage the AWS methods e.g. Web, CLI, or CDK for a more granular level e.g. subnet or host.

![Security Domain](https://i.imgur.com/085bwKN.png)

![Security Domain](https://i.imgur.com/RtD3lB5.png)

### <span class="fontColorH3">FireNet</span>

A firewall network or FireNet is a collaboration architecture between Global Transit Network and the firewall appliance e.g. AWS Network Firewall or Fortinet FortiGate. In essence, ++you could deem FireNet as an advanced Global Network Transit with the network security enhancement++. There are two working models of FireNet;

- Standard FireNet - a dedicated Aviatrix Transit Gateway acts as a FireNet Transit Gateway.
- Transit FireNet - enable the FireNet feature on the existing Aviatrix Transit Gateway.

#### <span class="fontColorH4">:label:Standard FireNet</span>

The foundation of Standard FireNet is AWS Transit Gateway; in other words, ++this model is purely designed for AWS++. Because of this reason, the Standard FireNet model closely collaborates with Security Domain.

![Firewall Network](https://i.imgur.com/r5FaEBr.png)

When a Security Domain connects to <span class="fontColor">*Aviatrix_FireNet_Domain*</span> which is automatically created when deploying AWS Transit Gateway via Aviatrix Controller, two actions will be taken place;

- A default route and the FireNet CIDR will be added to the linked Security Domain automatically.

![Transit Gateway Route Table](https://i.imgur.com/uDAYHIT.png)

- The entire VPC Route Table will be taken over by AWS Transit Gateway automatically.

![VPC Route Table](https://i.imgur.com/VBmVXr6.png)

#### <span class="fontColorH4">:label:Transit FireNet</span>

What if the customers do not want to extra deploy a dedicated Aviatrix Transit Gateway that just for FireNet? Or they are keen to benefit from the FireNet architecture for a more granular inspection albeit their resources are out of AWS or across different clouds? That is why Transit FireNet comes up, ++a FireNet architecture regardless of what the CSP is++.

Unlike Standard FireNet relies on AWS Transit Gateway to function, Transit FireNet is a built-in feature of Aviatrix Transit Gateway; as a result, the entire routing is still handled by Global Transit Network completely.

![Transit FireNet Overview](https://i.imgur.com/maayybo.png)

Because the routing is still handled by Global Transit Network, hence nothing will be changed on the VPC Route Table after enabled the Transit FireNet feature; all the changes will happen on Aviatrix Transit Gateway only e.g., additional interfaces will be created, and a default route will be pointed toward the firewall appliance.

![Transit FireNet Detail](https://i.imgur.com/yDgf9Uv.png)

### <span class="fontColorH3">Egress FQDN Filtering</span>

As its name, ++what Egress FQDN Filtering does is to restrict the Internet access on the FQDN level++. There are two working models of Egress FQDN Filtering as well as FireNet;

- Spoke FQDN Gateway - enable the feature on the existing Aviatrix Spoke Gateway.
- FireNet FQDN Gateway - a dedicated Aviatrix Spoke Gateway acts as a Egress FQDN Filtering Gateway.

#### <span class="fontColorH4">:label:Spoke FQDN Gateway</span>

The easiest and quickest way is to turn on this feature on the Aviatrix Spoke Gateway directly. However, it is not a good choice; it is more acceptable for the temporary purpose. The primary reason is that the FQDN gateway is external-face, the Source NAT feature has to be turned on accordingly. You will no longer be able to embrace Transit FireNet once you do so due to Aviatrix Spoke Gateway acts as a NAT Gateway; that is the external egress traffic will be terminated on it and will not be forwarded to Aviatrix Transit Gateway anymore. It will be a nightmare especially when you have numerous Spoke FQDN Gateways no matter they are within a single cloud or across the clouds.

#### <span class="fontColorH4">:label:FireNet FQDN Gateway</span>

The most ideal way to launch the FQDN gateway is based on either the Standard FireNet or Transit FireNet structure. All the details are elaborated in the whole ***FireNet*** section.

### <span class="fontColorH3">Observability</span>

Once you have all the features in place, the next task will be the visibility; that is CoPilot's responsibility. CoPilot is composed of several modules e.g. Topology, FlowIQ, and Performance. Typically, the information it presents is not easy to produce by yourself; it would be a lots of efforts if you want to e.g., deploy and manage your own probes across the clouds, develop the analysis framework, and then visualize the outcome. I do not want to introduce each module due to it will be a bit tedious, I would like to limelight some of them instead.

#### <span class="fontColorH4">:label:Topology</span>

There are three key spotlights;

- Everything is presented in a single pane; it does not matter your network transport is built on which account, region, and even cloud.  
- Compute resource inventory; are you satisfied if the visualization covers the networking stuff only? Well, the answer is No without doubt especially when your services are launched across the accounts, regions, and even clouds. That is one of the reasons why CoPilot is so fascinating due to you will be able to promptly find out where your compute resource resides. <span class="fontColor">Please allow me to use the following screenshot for the depiction which I took from [YouTube](https://www.youtube.com/watch?v=vJtEpX89ecA) due to I forgot to capture it during the assessment period...</span>

![Compute resource inventory](https://i.imgur.com/K3b2tSZ.jpg)

- Tags are able to present; in terms of the visibility, CoPilot could be deemed as an enhanced AWS Transit Gateway Network Manager due to Global Transit Network is not only able to view in multiple tiers e.g. cloud, region, and VPC/VNet, but also with keys and values.

![Tags](https://i.imgur.com/FaxGOj8.png)

#### <span class="fontColorH4">:label:AppIQ</span>

What AppIQ offers is an end-to-end analysis from the network latency aspect. In addition, it also includes the FlightPath report which is a very useful tool of diagnosing the reachability.

![AppIQ](https://i.imgur.com/fyUXLLB.png)

FlightPath is an end-to-end analysis of the reachability; it does not rely on CoPilot to function, instead, it is launched via Aviatrix Controller. The reason why FlightPath is so helpful due to its granularity is capable of narrowing down to the Security Group layer instead of just the Route Table layer. You could even recall how many times you got stuck in a broken reachability due to the absence of the visibility.

![FlightPath](https://i.imgur.com/cQi5fDU.png)

## <span class="fontColorH2">Conclusion</span>

Networking is a core and very fundamental component, therefore, it is almost invisible and even looks valueless, no matter it functions in the on-premises or the cloud. However, it does not mean that it could be ignored!

When turning to the cloud world, networking is one of the managed offerings of each CSP, therefore, how to manage the inter-communications across the clouds with a  straightforward approach, and how to enlarge the visibility to streamline the routine operation, that have become one of the key cloud adoption strategies nowadays.

> Published on 4^th^ July 2022. 

:::info
###### tags: `AWS` `Azure` `GCP` `OCI` `Aviatrix` `Architecture` `NativeCloudNet` `TransitVPC` `TransitGateway`  `Muiticloud`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}