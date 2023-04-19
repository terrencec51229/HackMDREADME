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

# Public Cloud Anywhere - Distributed Cloud

[TOC]

## <span class="fontColorH2">Where Is It Comes From
</span> 

Before we dig into what is Distributed Cloud, let us see two concise explanations first.

- <span class="fontColorH3">**Gartner**</span>
    
    *Distributed cloud is the distribution of public cloud services to <span class="fontColor">different physical locations</span>, while the operation, governance, updates and evolution of the services are the responsibility of <span class="fontColor">the originating public cloud provider</span>.*

- <span class="fontColorH3">**IBM Cloud**</span>

    *Distributed cloud is a public cloud computing service that lets you run public cloud infrastructure in multiple different locations  -  not only on your cloud provider's infrastructure but <span class="fontColor">on premises, in other cloud providers' data centers, or in third-party data centers or colocation centers</span>  -  and manage everything from a single control plane.*

    *With this targeted, centrally managed distribution of public cloud services, your business can deploy and run applications or individual application components in a mix of cloud locations and environments that best meets your requirements for performance, regulatory compliance, and more. <span class="fontColor">Distributed cloud resolves the operational and management inconsistencies that can occur in hybrid cloud or multicloud environments.</span>*

    *Maybe most important, <span class="fontColor">distributed cloud provides the ideal foundation for edge computing</span> - running servers and applications closer to where data is created.*

In summary, there are three key factors from the abovementioned definitions.

- Distributed Cloud is an extension of the region where the cloud service provider has not launched any service yet.
- The whole Distributed Cloud framework is completely managed by the cloud service provider nonetheless.
- One of the use cases for Distributed Cloud is edge computing.

When we turn to the AWS aspect, the corresponding solution is [Outposts](https://aws.amazon.com/outposts/),  which is designed by the HCI (hyperconverged infrastructure) foundation. According to the abovementioned factors, here is what Outposts can do.

- Outposts could be delivered to any place where meets [its requirement](https://docs.aws.amazon.com/outposts/latest/userguide/outposts-requirements.html).
- Outposts is a fully managed offering, which operates by a unified console as well as other services.
- Since Outposts could be delivered everywhere, hence you could locate it at any place where is close to clients.

## <span class="fontColorH2">What Does It Differ From Edge Computing</span>

As mentioned, edge computing is one of the use cases of Distributed Cloud; however, it does not mean that Distributed Cloud is completely designed for it. That is because Distributed Cloud also takes several non-technical drivers into account, for instance;

- The regions where the cloud service provider launches do not meet the organisation's compliance requirement, e.g., our business is restricted to be served in the US region.
- All the organisation's assets must be stored in the visible stuff, e.g., our data have to be stored in the physical and dedicated hardware.

### <span class="fontColorH3">Public Cloud Accelerates Edge Computing</span>

Before the public cloud becomes popular, what is the primary requirement if an organisation wants to broaden its business in different regions? The answer is quite straightforward to imagine - to build several physical infrastructure sites. However, it has a dependency on how much budget the organisation is willing to invest.

After the public cloud grew up, the organisation no longer needs to build any physical site to support its business growth; instead, it just needs to launch any service they want in any region where is close to their clients. The organisation could even adopt trial-and-error without a strict plan. As a result, the public cloud could be deemed a prototype of edge computing; in other words, it gives a space for any organisation re-imagining its business territory.

### <span class="fontColorH3">AWS Anywhere</span>

When more and more organisations embrace the public cloud, human nature comes up accordingly due to some organisations are not satisfied by the regions each cloud service provider offers. They want more!

For instance, if an organisation wants to broaden its business in Taiwan, but the closest region is Hong Kong, where is out of their consideration due to it does not meet the compliance requirement. What an organization can do? Nothing, unfortunately.

Before AWS released Outposts, every organisation was restricted to broaden its business according to the regions where AWS launches. Luckily, once AWS released Outposts, most technical/non-technical boundaries have been cleared.

![Public Cloud Anywhere via Outposts](https://i.imgur.com/l2VNLTV.png)

Nowadays, CDN is not only just for content delivery (cache) but also content distribution (process); therefore, most CDN providers have involved their business portfolio in the edge computing market. [Akamai EdgeWorkers](https://techdocs.akamai.com/edgeworkers/docs/event-handler-functions) and [Cloudflare Workers](https://developers.cloudflare.com/workers/learning/how-workers-works/) are one of the examples. 

When we turn to AWS specifically, or even from the multi-CDN viewpoint, the implementation could be below;
- For applications that are able to launch on either the [Lambda@Edge](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html) or the 3^rd^ party solutions, e.g. Akamai EdgeWorkers or Cloudflare Worker, you could be benefited that the local requests are locally served, powered by either CloudFront or the 3^rd^ party vendors.
- For applications that are not able to launch on the abovementioned platforms, you could still leverage their private high-speed transport backbone to optimise every end-to-end request process that is served by compute resources in Outposts.

![Public Cloud Anywhere via Outposts with CDN](https://i.imgur.com/wR770KB.png)

Apparently, this comprehensive model redefines the boundary, no matter from the infrastructure or the business perspective.

## <span class="fontColorH2">Conclusion</span>

Nowadays, we have benefited from each public cloud provider due to plenty of agility and elasticity; one of them is a prototype of edge computing. Obviously, we all are greedy and  want more; therefore, the Public Cloud Anyware idea comes up, which gives us a way to unlock the business boundary.

> Published on 7^th^ August 2021.

:::info
###### tags: `AWS` `Architecture` `HybridCloud` `EdgeCompute` `Outposts`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}