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

# Supercloud 101

[TOC]

## <span class="fontColorH2">What Is Supercloud?</span>

What is Supercloud? You probably have several questions, e.g.,

- Is it a new terminology?
- What does it differ from the Private Cloud, Public Cloud, and Multi-cloud?

Before we dive into each of them, let us retrospect the transition between the cloud models at the outset.

- Typically, most organisations get started from on-premises data centres/colocations where aka the **Private Cloud**; their operation team fully handles the underlay infrastructure routines, e.g. procurement, installation, or optimisation.
- Because the benefits introduced by [cloud computing](https://aws.amazon.com/what-is-cloud-computing/) have been accepted widely, more and more organisations kick off their digital transformation journey and move their business on either Amazon, Microsoft, or Google these **Public Cloud** vendors.
- Essentially, when an organisation gets enough familiarity with a single CSP, its offering might not meet the organisation's business requirements across the functionality *(Google is strong in data analysis*) and risk *(should we not put everything on Amazon?)* considerations. As a result, the **Multi-cloud** strategy would be formulated.

++When an organisation moves to the multi-cloud phase, either multiple CSPs or with on-premises environments (aka **Hybrid Cloud**), one thing that would be emphasised is consistency, either architectural or operational.++ In terms of compute resource deployments, you definitely could extend the existing deployment framework to another CSP and even more; however, it takes time due to each CSP has proprietary service architectures and they are not compatible with others. For this reason, would it be more simply and efficiently if there is an abstracted layer between the operation team and clouds? In other words, the operation team does not require to directly communicate with respective CSPs, instead, they just need to talk to that orchestrator and it would be on their behalf to face those clouds. ++That is what Supercloud aims to be!++

![Segregation vs. Aggregation](https://i.imgur.com/dfh91ko.png)

Here is the definition from [Supercloud Working Group Definition](https://docs.google.com/document/d/1SP0G-3CEnJ4Zz1sPoZt6eA6Weq8F5Osk93jLcPLcK60).

> *Supercloud is an emerging ++distributed++ computing architecture that comprises a set of services ++abstracted from the underlying primitives of proprietary clouds++ (e.g., compute, storage, networking, security, and other native resources) to create a global system spanning to all clouds it is interfaced with. In principle, Supercloud has the capability to allow the integration of any current and future hyperclouds or other proprietary cloud architectures.*

The following sections will discover more.

## <span class="fontColorH2">Is Supercloud An Evolution of Multi-cloud?</span>
I know that the Supercloud is a bit controversial due to not everyone agrees with it as one of the cloud models. In addition, someone deems that it is a marketing term. As the matter of fact, I do not want to debate what it really is; ++however, one thing is worth us keeping in mind is that [the multi-cloud strategy has transitioned to a by-default pattern instead of by a design requirement](https://www.youtube.com/watch?v=KrYPKBQDcGM&t=386s)++.

Because of this transition, you probably wonder that could we deem the Supercloud a next-generation multi-cloud framework? Or, should we treat it as an enhanced **Distributed Cloud**? Well, in my humble opinion, it is neither the multi-cloud nor Distributed Cloud, it is both instead. That is to say, ++it shall be recognised as a multi-cloud based Distributed Cloud architecture++.

## <span class="fontColorH2">Representatives</span>

Although the Supercloud architecture does not belong to any specific cloud model, it is presented through the IaaS or SaaS manner.

### <span class="fontColorH3">Aviatrix: Air Space and CoPilot, IaaS</span>

For those who have not known [Aviatrix](https://aviatrix.com/cloud-network-platform/), here is a high-level introduction.

> *The pioneer of Intelligent Cloud Networking™, optimizes business-critical application availability, performance, security, and cost with multicloud networking software that delivers a simplified and ++consistent enterprise-grade operational model in and across cloud service providers++*.

What it actually does is be an orchestrator/abstracted layer to concentrate your network transport management across clouds. As the following diagram, every data-plane provision (e.g. Aviatrix Gateways or AWS Transit Gateway) and manipulation (e.g. adjustments of the VPC/VNet Route Table) is fully handled by Aviatrix Controller.

![Aviatrix Transit Architecture](https://i.imgur.com/YlDNJJz.png)

For this reason, the operation team does not need to integrate their deployment pipelines with individual CSPs; additionally, they do not need to login each CSP's console for either seeking where does the compute resource locate across accounts, regions, and landing zones, or observing if anything is abnormal on the transit network.

My post [Enhanced Management of Multi-cloud Networking and Security](https://bit.ly/enhanced-management-multicloud-networking-security) has a very deep-dive introduction in why an organisation might need Aviatrix and what are the benefits that an organisation could get from it. It is worth taking a look!

### <span class="fontColorH3">VMware: Cross-Cloud Services, SaaS</span>

Typically, most organisations adopted VMware vSphere for their compute resources before the concept of the cloud gets popular; therefore, the entire [VMware Cloud](https://www.vmware.com/cloud-solutions.html) offering would be a fascinating choice if they do not really need those comprehensive features each CSP caters.

![VMware Cross-Cloud Services](https://blogs.vmware.com/cloudprovider/files/2022/09/Picture1.jpg)

As of today, VMware has launched its vSphere stack on Amazon, Microsoft, Google, IBM, Oracle, and Alibaba for catering any preference across CSPs; in other words, you could leverage existing deployment pipelines to manage your virtual machines via vCenter and containers through Tanzu, regardless of which the CSP is. Other than that, both VMware Aria *(formerly vRealize Cloud Management)* and VMware NSX Advanced Load Balancer *(formerly Avi Networks)* adopt the same way as well. ++The VMware Cross-Cloud Services is not a single platform, instead, it is a total solution or even an ecosystem.++

My old post [Migrate On-premises Workloads To AWS](https://bit.ly/migrate-onpremises-workloads-to-aws#Conversion-independent) outlines the benefits you could get from VMware Cloud on AWS. Although it is a bit outdated (I released it in 2019), the core concept is still valid; additionally, you could get more information from the YouTube links below;

- Intro: [How VMware Cross-Cloud Services Works](https://www.youtube.com/watch?v=T2vBNKwU0N0)
- Demo: [Application Transformation](https://www.youtube.com/watch?v=6Gg8dHjh02Q&list=PL9MeVsU0uG66yIItfmR14P3Ti8c7KWuy_&index=31), [Accelerating Cloud Transformation](https://www.youtube.com/watch?v=WgxCSwuDk_c&list=PL9MeVsU0uG66yIItfmR14P3Ti8c7KWuy_&index=32), and [Hybrid Workspace](https://www.youtube.com/watch?v=MT5u7d--B1s&list=PL9MeVsU0uG66yIItfmR14P3Ti8c7KWuy_&index=33)

## <span class="fontColorH2">Conclusion</span>

Although the term "Supercloud" is not acceptive across the board and I rarely hear it from the market, I still agree with its principle - distributed and abstracted across clouds. Eventually, every organisation will move to this stage when its business is launched on the multi-cloud architecture.

In this extremely fast-paced era, how could an organisation augment its business territory more rapidly and operate its environments more efficiently? Consistency is the only answer, it does not matter whether you are a resource consumer or a platform provider.

> Published on 10^th^ Febuary 2023. 

:::info
###### tags: `Multicloud` `Distributedcloud` `Supercloud` `Aviatrix` `VMwareCloud`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}