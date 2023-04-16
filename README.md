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

# A Modernised Landing Zone Design Rather Than Just About Reachability

[TOC]

## <span class="fontColorH2">Preface</span>

Let us forget anything about the cloud before we get started the subject today. In the past, when we needed to launch any service on top of the IT infrastructure, what we did could be summarised below typically;

- Evaluate how much capacity is required to afford the loading.
- Consider how to safeguard applications that expose to the Internet with minimum compromise or without compromising the degraded performance.
- Consider how to govern communications between applications across environments with elasticity and granularity.

When we look at the cloud era nowadays, the first task is completely offloaded to the CSPs, hence it is no longer a concern *(well, the only thing that you definitely need to care about is how to ensure that you will not get surprised when you receive the bill :expressionless:)*. However, the rest of the tasks are still our responsibility. Before considering anything about protection, management, or both, you need to build a place to accommodate those business-critical applications; but, what does the landing zone differ from the on-premises infrastructure design? Because a landing zone is able to deem an SDDC (software-defined data centre) in essence.

That is a great question, is not it :sunglasses:? ++Concisely, the traditional infrastructure focuses on reachability; however, the landing zone much focuses on application-driven design.++ What does it mean? The following scenarios will discover more.

## <span class="fontColorH2">A Modernised Landing Zone</span>

In the SDI (software-defined infrastructure) world, not all the requests would be handled by the same workflow; three different types of workflow come up accordingly, the control, data, and management planes. In the following scenarios, I will take the data and management planes to elaborate on what a modernised landing zone looks like.

### <span class="fontColorH3">Internet Access: Origin Visibility</span>

We all know that there are two types of origin from the CDN aspect: static and dynamic content.

- Static Content - Intrinsically, the output that could be fetched by way of the HTTP GET request belongs to this category, e.g. images (.jpg). In addition, these types of content are cacheable due to they could be gained directly without interacting with applications.
- Dynamic Content - If the output is fetched by way of several interactions between applications, e.g. HTTP POST, that is to say, it depends on the request then this output belongs to this category, e.g. Web pages (.aspx). Because of this reason, these types of content are not cacheable.

In other words, the dynamic content relies on the compute resources and the static content does not; as a result, that is why each CSP encourages every customer to serve their static content via the object storage, e.g. AWS S3 and GCP Cloud Storage, instead of the block storage. This adoption is not only about the requirement but also about cost optimisation.

CDN is an Internet-face service, meaning every origin must be exposed to the Internet; however, from the security standpoint, every exposure is risky. For this reason, we normally add additional logic on the CDN and origin side for enhancement.

- The first gate - When the CDN receives the request without any authorised HTTP header, this request would be dropped.
- The second gate - When the request passes the inspection by CDN, the CDN adds one or more additional HTTP headers; when the origin receives the request comes from CDN without those HTTP headers, this request would be dropped as well.

**But, what if exposure is not required?** It would be great from the security standpoint. Luckily, it is feasible when S3 collaborates with CloudFront via [OAI (origin access identity)](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-cloudfront-introduces-origin-access-control-oac/) or [OAC (origin access control)](https://aws.amazon.com/blogs/networking-and-content-delivery/amazon-cloudfront-introduces-origin-access-control-oac/).

![Internet Access](https://i.imgur.com/3owyJiQ.jpg)

The primary benefit we could get from this manner has two if compared with the 3^rd^ party vendors;

- ++The S3 bucket keeps private only++, meaning the origin is invisible from the Internet (the bucket must expose to the Internet for any 3^rd^ party vendor communicating with).
- You could still benefit from all the enhancements by CloudFront due to ++the communication between S3 and CloudFront is well-defined and well-protected on the resource level via the bucket policy++ (it is not straightforward to manage where the 3^rd^ party vendors come from).

### <span class="fontColorH3">Internal Access: VPC Endpoints</span>

When we turn to the internal environment, every Internet access must pass through the NAT process. However, it does not mean that every AWS public service adopts the same way; there are two exceptions, ++DynamoDB++ and ++S3++. Both DynamoDB and S3 could be accessed by way of the VPC Gateway Endpoint without involving address translation; all the resources within the VPC are able to access those services via their internal/private address.

But, not every service could be accessed publicly due to they have not been exposed outside of the AWS world and they will not, either. For instance, KMS, Session Manager, and System Manager. In order to reach those services which cannot communicate directly, the VPC Interface Endpoint is required.

![Application-driven Networking](https://i.imgur.com/o9lVX9Z.jpg)

The primary difference between the endpoints could be summarised below.

- ++The Gateway Endpoint needs to be associated with the VPC Route Table++, but the Interface Endpoint does not.
- ++The Interface Endpoint is able to customise the access control++, e.g. service policy and Security Group, but the Gateway Endpoint is not.

Most importantly, what is the message that the VPC Endpoint would like to share with us? That is to say, ++[every solution architecture must be application-aware](https://www.linkedin.com/posts/terrencec51229_a-new-role-for-network-pros-application-flow-activity-7026120850051407872-ZsQg?utm_source=share&utm_medium=member_desktop)++ due to one of the significant differentiators between on-premises and cloud is not everything is reachable on the cloud by default, require additional settings instead, e.g. adaptor (VPC Endpoint) or permission (IAM Policy); as a result, reachability is not a foundation anymore.

This point not only reflects the changes across technologies but also notices that our responsiblity enlarges due to the infrastructure design closely works in conjuction with the application functionality in a modern development framework.

### <span class="fontColorH3">Multi-site Access: Segmentation</span>

Unlike the on-premises environment where everything is put together,  which is caused by several factors, e.g. facility location and resource capacity, a system architecture on the cloud is typically composed of more than single one landing zone, e.g., the infrastructure components are deployed in zone 1, the application components are deployed in zone 2, and the security components are deployed in zone 3. This partition does not complicate the architectural design, instead, it gives more granularity and elasticity, especially from the ownership perspective.

Essentially, you shall not anticipate that any workload that resides in either the development or the staging environment is able to communicate with any service in the production environment. However, rely on the Network ACL and Security Group to grant access that could be optimised. ++Another gate that is worth considering is segmentation, to categorise the type of traffic.++ In order to accomplish traffic categorisation, you could leverage either the Transit Gateway Route Table or the Cloud WAN Segment; the adoption depends on which service (Transit Gateway or Cloud WAN) you use for bridging all the environments together. You could see more in-depth comparisons between Transit Gateway and Cloud WAN in my post - [Globally Operate Your Network Transport via Cloud WAN](/-lcMon2oQoKVp_C3z6xMaw).

![Transit Gateway Topology](https://i.imgur.com/tls7k9l.png)

![Cloud WAN Topology](https://i.imgur.com/wBFUpsk.png)

++Segmentation is just the outset due to it is a prerequisite of traffic inspection, which is a key piece of secure cloud networking.++ Every communication, no matter the east-west or the south-north access, would be inspected by the security appliance which could be either the Network Firewall or the 3^rd^ party software. You could even leverage the Security Group to form more granular and tiered access management.

![Multi-site Access](https://i.imgur.com/LMIaAnV.jpg)

### <span class="fontColorH3">Out-of-band: Session Manager</span>

In order to ensure that every underlying infrastructure component is manageable, typically, we would design a completely isolated network instead of operating them via service network, which is also shared with applications. This design gives us a back door to investigate what is happening whenever the service network malfunctions. When we turn to the cloud world, every remote access relies on either a dedicated connection (Direct Connect) or the Internet, and these ingredients belong to the service network in essence; that is to say, you will not be able to operate resources if they are being either maintained or malfunctioned. **Is any way to touch the environment without integrating any abovementioned feature?** Yes, that is Session Manager.

![OOB](https://i.imgur.com/60GJBY6.jpg)

You ++neither need to maintain any extra key pair++ nor operate any additional resource that Session Manager governs. The only thing you need to do is just connect your resource, which is the EC2 instance in particular, via Session Manager. In addition, another security optimisation is that ++none of the service ports, e.g. RDP and SSH, is required to open due to the communication between the EC2 instance and Session Manager is behind the scenes++. Pretty straightforward, is not it :sunglasses:?

## <span class="fontColorH2">Wrap-up</span>

As I mentioned at the outset, every landing zone could not inherit the traditional infrastructure criterion to design and implement due to their emphasis is totally diverse. Every landing zone acts as not only just a fundamental but also crucial role to support business to function with maximum flexibility and minimum security compromise.

If we look at Origin visibility, VPC Endpoints, segmentation, or Session Manager as an individual enhancement, then you perhaps ask yourself "Should I need them?"; however, whilst we take those enhancements into the same picture, then the outcome is absolutely different.

> Published on 17^th^ April 2023. 

:::info
###### tags: `AWS` `Architecture` `Security` `CloudFront` `VPCEndpoint` `Segmentation` `TransitGateway` `CloudWAN` `SessionManager`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}