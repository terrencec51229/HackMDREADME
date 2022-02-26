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

Before we dig into what is Distributed Cloud, let us see two concise explanations in advance.

:::spoiler <span class="fontColor2">Gartner</span>
Distributed cloud is the distribution of public cloud services to <span class="fontColor">different physical locations</span>, while the operation, governance, updates and evolution of the services are the responsibility of <span class="fontColor">the originating public cloud provider</span>.
:::

:::spoiler <span class="fontColor2">IBM Cloud</span>
Distributed cloud is a public cloud computing service that lets you run public cloud infrastructure in multiple different locations - not only on your cloud provider's infrastructure but <span class="fontColor">on premises, in other cloud providers’ data centers, or in third-party data centers or colocation centers</span> - and manage everything from a single control plane.

With this targeted, centrally managed distribution of public cloud services, your business can deploy and run applications or individual application components in a mix of cloud locations and environments that best meets your requirements for performance, regulatory compliance, and more. <span class="fontColor">Distributed cloud resolves the operational and management inconsistencies that can occur in hybrid cloud or multicloud environments.</span>

Maybe most important, <span class="fontColor">distributed cloud provides the ideal foundation for edge computing</span> - running servers and applications closer to where data is created.
:::


In summary, there are three key factors from the abovementioned definitions.

- Distributed Cloud is an extension of the region where the cloud service provider has not been launched yet.
- The resources of Distributed Cloud are completely managed by the cloud service provider nonetheless.
- One of the use cases for Distributed Cloud is edge computing.

When we turn to the AWS aspect, the corresponding solution is [Outposts](https://aws.amazon.com/outposts/) which designs by the HCI (Hyperconverged Infrastructure) concept. What Outposts can do according to the abovementioned factors are;

- Outposts is able to deliver to anywhere where meets [its requirement](https://docs.aws.amazon.com/outposts/latest/userguide/outposts-requirements.html).
- Outposts is a fully managed service and managed by an unified console as well as other resources.
- Since Outposts is able to deliver to anywhere, hence you could locate it to the place where is closed to the clients.

## <span class="fontColorH2">What Does It Differ From Edge Computing</span>

As mentioned, the edge computing is one of the use cases for Distributed Cloud; however, it does not mean that Distributed Cloud is completely designed for it. That is because Distributed Cloud also takes several non-technical drivers into account, for instance;

- The regions where the cloud service provider launches do not meet the organization's compliance requirement, for example; our business cannot launch in several APAC regions.
- All the organization assets must be stored in the visible stuff, for example; our data has to be stored in the physical and dedicated hardware.

However, they are very closed to each other in the real world.

### <span class="fontColorH3">Public Cloud Accelerates The Edge Computing</span>

Before the public cloud becomes popular, what is the primary requirement if an organization wants to broaden its business in different regions? The answer is quite easy to imagine - to build several physical infrastructure sites. However, it has a dependency of how many budget that the organization is willing to invest.

After the public cloud grew up, it re-defined what is the requirement of local service; the organization no longer need to build the physical sites, instead, it just needs to launch the service in the region where is closed to the clients. The organization could even adopt trial-and-error without a strict plan.

For this reason, the public cloud could be deemed as a prototype of the edge computing. In other words, it has a space to be optimized.

### <span class="fontColorH3">AWS Anywhere</span>

When more and more organizations embrace the public cloud, the human nature comes up accordingly; some organizations are not satisfiled by the regions each cloud service provider offers. They want more!

For instance, if an organization wants to broaden its business in Taiwan, but the closest region is Hong Kong, and this region is out of consideration due to it does not meet the compliance requirement. What can organization does? Nothing, unfortunately.

Before AWS released Outposts, the organizations were restricted to broaden their businesses according to the regions where AWS launches. However, after it released, all the technical/non-technical boundaries have disappeared.

![Public Cloud Anywhere via Outposts](https://i.imgur.com/l2VNLTV.png)

### <span class="fontColorH3">Enhanced AWS Anywhere</span>

Nowadays, CDN is not only just for content delivery (cache) but also content distribution (process); therefore, most of the CDN providers have involved the edge computing market. The compute resources behind the CDN provider locate in either the regions where AWS offers or any locations where the Outposts racks locate. As a result, other than the private high-speed transport backbone offers by each CDN provider, the optimization of end-to-end request process would be more significant than without Distributed Cloud. That is because we define where the boundary is.

![Public Cloud Anywhere via Outposts with CDN](https://i.imgur.com/wFNlZG9.png)

## <span class="fontColorH2">Conclusion</span>

Nowadays, all of us have been benefited by each public cloud provider in that they give us a prototype of edge computing - every deployment has transferred to several simple clicks or API calls.

Obviously, we all are greedy that want more. As a result, the Public Cloud Anyware solution comes up. The edge computing should be end-to-end as well - that is on the front-end side, we benefit from the high-speed/resiliency private transport offers by the CDN, and on the back-end side, we are not benefited via the formal regions but also the custome ones.

:::info
###### tags: `AWS` `Architecture` `HybridCloud` `EdgeCompute` 
:::

{%hackmd BJrTq20hE %}