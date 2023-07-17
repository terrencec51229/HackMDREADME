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

## <span class="fontColorH2">*Before We Get Started...*</span>

Is access leak easy to penetrate in my environment? The answer is **Yes** without any doubt. Therefore, another question may arise in your mind: if it really is a spotlight that is worth keeping an eye on, why did I not adopt any reaction in the past accordingly? The answer is **cloud**.

Before cloud adoption gets popular, every entry point of access is well-defined by either Security or the well-trained operation team.

- All kinds of management accesses, e.g. HTTP or SSH, could only be granted from specific sources, no matter the request is initialised from either the internal environment or the Internet.
- In addition, all bidirectional communications must be passed through the peripheral appliance, e.g. firewall.

Obviously, there are not too many ways to accidentally expose your services with unwanted profiles. However, what happened after cloud adoption had explosive growth? We all understand one of the cloud essentials is convenience, because of this strength, developers no longer require collaborating with the infrastructure team to publish their ideas over the world, they could do everything they want by themselves completely. On the other hand, this convenience results in unnecessary exposure. When we turn to the previous scenarios, they would look like below:

- Developers accidentally expose their resources, including databases, to the Internet directly.
- Developers accidentally expose not only the front-end data plane but also the back-end management plane to the Internet.
- Developers accidentally grant all the sources, including unknown ones, to communicate with their resources.

<u>Last but not least, developers have not been aware of those anomalies</u> (that is not their fault in essence due to their primary focus is development instead of operation), therefore, some unexpected security events took place afterwards, e.g. **ransom attack**.

## <span class="fontColorH2">*What Do You Need To Take Care Of*</span>

A well-protected framework does not mean blocking everything, instead, <u>every access is enforced with the least privilege principle</u>. The whole framework could be narrowed down to the following categories from my perspective:

- **Configuration review** aims to verify if any setup does not follow either standard practices or best practices.
- **Workload segmentation** aims to categorise all the resources into more granular groups and formulate different sets of communication boundary.
- **Privilege management** aims to ensure that every resource could only be manipulated by a limit of users or service roles with the least required permission.

### <span class="fontColorH3">*Configuration Review*</span>

In terms of reviewing your configuration on AWS, the most efficient way is leveraging [Security Hub](https://www.youtube.com/watch?v=oBac-GAoZJ8), a [cloud security posture management (CSPM)](https://www.youtube.com/watch?v=V4wmb5KVmKM) service that automates best practice checks, aggregates alerts, and supports automated remediation. The following demo gives you a high-level overview of what Security Hub does.

1. Choose the Security standards that meet your requirement.
2. Security Hub will summarise all the findings and present those details once the collection process completes.

![security hub](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t5ynd10k1nhsyi3aayzb.gif)

Essentially, Security Hub is to provide a set of security definitions instead of detecting and remediating any anomaly; these functions are handled by [Config](https://www.youtube.com/watch?v=MJDuAvNEv64) instead, a service that enables you to assess, audit, and evaluate the configurations of your AWS resources. As a result, enabling any of the Security Hub's security standards will automatically deploy the corresponding rules to Config.

![config rules](https://hackmd.io/_uploads/Sy2nTe4K3.png)

However, one thing needs to keep in mind is that <u>Config is disabled (off) by default</u>, it could not detect, collect, and remediate constantly until its status is enabled (on).

![config recorder](https://hackmd.io/_uploads/SJ1daxVKn.png)

Those predefined rules may not 100% meet your requirement, hence customisation is required. The following demo presents that a manual remediation rule is launched to inspect if any unauthorised source is detected. Every change made by Config, no matter automatically or manually, would be recorded by CloudTrail as well.

![config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/frr8uz26tgq8y587caa3.gif)

### <span class="fontColorH3">*Workload Segmentation*</span>

I emphasised how the importance of segmentation in my previous post [A Modernised Landing Zone Design Rather Than Just About Reachability](https://bit.ly/modernised-landing-zone-design-rather-than-just-about-reachability#Multi-site-Access-Segmentation), however, a robust security design not only consider the site segments but also the workload segments. Every workload segment could be easily accomplished via Security Group.

Everyone knows that each Security Group is responsible for access control by its nature, but what it does is more than that! When we look at the best practice principle, each component shall have its own Security Group. If a Security Group is either associated or shared with multiple resources then these resources belong to the same segment. In addition, you shall avoid using the IP address to manage your access, the self-referencing rule instead (Security Group over Security Group, or nested Security Group), which gives you a more granular and flexible way to authorise every communication.

![security group](https://hackmd.io/_uploads/SkQOLztY3.png)

The benefit you could get from the self-referencing rule architecture especially when the resource scales by the auto-scaling group due to the IP address allocation is floating in this manner. The IP address shall only be considered in the following scenarios:

- <u>The health-check request initialises from the Network Load Balancer (NLB) subnet(s)</u>
- <u>Any out-of-VPC communications, e.g. Internet or inter-VPC.</u>

Other than manipulating Security Groups to manage segments, you could also consider leveraging the 3^rd^ party software to streamline your operation.

- [Aviatrix - Distributed Cloud Firewall](https://aviatrix.com/resources/distributedcloudfirewall/webinar-distributed-cloud-firewall-reduce-cloud-infrastructure-costs-and-improve-cloud-security), an agentless solution that provides an orchestration layer to simplify every access rule's change. My previous post [Enhanced Management of Multi-cloud Networking and Security](https://bit.ly/enhanced-management-multicloud-networking-security) introduced why Aviatrix is worth considering and what are the benefits that an organisation could get from it as well.

![Aviatrix - Distributed Cloud Firewall](https://aviatrix.com/wp-content/uploads/2023/05/DCF-images.png)

- [Illumio - Illumio Core](https://www.illumio.com/resource-center/illumio-core-demo), an agent-based solution that could decouple each service group into a more granular slice (micro-segment). Illumio also highly advocates why Zero Trust Segmentation matters, especially from the aspect of preventing ransom attacks.

![Illumio - Illumio Core](https://hackmd.io/_uploads/HJVlBtot3.png)

### <span class="fontColorH3">*Privilege Management*</span>

We already talked about remediation and segmentation, let us move on to delegation. Obviously, IAM acts as a key factor in this section; however, it could only apply to AWS correlatives, e.g. IAM users or service profiles. If the OS layer is handled by an organisation and the organisation has leveraged Active Directory to manage user and service identities then implementing Group Policies would be the most efficient approach.

When we look at the whole privilege management framework, it primarily focuses on access and identity management from my perspective.

- In terms of privileged access management, its principle intrinsically is **zero-trust**, none of the users is able to manipulate any resource directly; instead, all the manipulations are by way of a managed orchestration layer, which minimises unnecessary exposure, e.g., several unknown communications caused by a malicious process inside an installed package.
- In terms of privileged identity management, it covers permission and credentials. Which user could manipulate which resources without needing any firewall between them; in addition, none of the resource credentials could be used directly, instead, every authorised user would be given a temporary key to manipulate. This bundle of the role-based access control (RBAC) foundation and the vault protection enhancement could minimise the risk effectively if credentials were revealed accidentally.

This [resource](https://www.wallix.com/blog/what-is-pam-privileged-access-management/) outlines a full picture of what privilege management looks like.

![pam](https://hackmd.io/_uploads/S1L570xc2.png)

## <span class="fontColorH2">*Let Us Wrap Up!*</span>

In the past 5 years or even a decade, we could trust that our business was well-safeguarded by the anti-virus and anti-malware software; however, this methodology is no longer enough at all nowadays due to attacks keep innovating as well as technologies. Additionally, because of the maturity and popularity of cloud adoption, everyone has the chance to fulfil their ideas without too many engagements across teams; that is to say, they even could complete everything by themselves. But, when everything becomes too easy and convenient, it sometimes gives us a heads-up that something may need your attention, which is security in most cases.

<u>Everyone needs to care and is responsible for security in this fast-paced generation</u>; more accurately, everyone has to always bear security in mind due to <u>anything that cannot be seen does not mean that it does not exist</u>, it is just because this trick-or-treat is patiently waiting for your carelessness. No one wants to get shocked, does not it?

:::info
:date: Published on 17^th^ July 2023. This post is also published in my [dev.to](https://dev.to/@terrencec51229) space.
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}