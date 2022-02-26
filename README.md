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

# The Transition of Responsibility From The Infrastructure Standpoint

[TOC]

## <span class="fontColorH2">Outline</span> 

I have had an idea that to share my perspective of what the infrastructure role will be after passing the AWS Solutions Architect - Professional exam. My intention is certainly not flaunting how the honor it is, instead, it is more about sharing the following thoughts in the cloud world specifically within these years.

- Moved from an infrastructure owner to an infrastructure consumer.
- To be more widely involve in various territories.

Obviously, one of the common strategies in the cloud world is that to <span class="fontColor">use</span> the services offered by each provider instead of <span class="fontColor">managing</span> them, no matter the solutions you adopted are IaaS, PaaS, or SaaS. For this reason, a bunch of the infrastructure routines offloaded for instance;

- Renew the warranty of each hardware/software resource.
- Assess the capacity of each component such as networking, server, storage, and data center facilities.
- Keep an eye on each component's health/available state - 100% uptime as close as possible.

Therefore, the next and expected question would be <span class="fontColor">what the infrastructure role will be</span> in the upcoming future? Before I dig into it, there is one thought in my mind that has not been changed as yet - <span class="fontColor">we are still valuable indeed</span>. Our values/contributions are just presented differently instead.

## <span class="fontColorH2">The Market Requirement Has Been Changed</span>

Typically, a full cloud environment takes place for most start-ups. The reason is quite easy to think - they do not have too many histories that need to be taken into account. Their culture is fresh and lightweight. Therefore, they do not really need a consolidated infrastructure team for designing the architecture, operating the deployments, and observing the availability of each service they launched. What they need is a DevOps team. The strategy is correct due to the following reasons;

- Automation instead of manual manipulation - the focus is on how to define and achieve one or more pipelines with minimum or even zero human intervention.
- Provider-managed service instead of self-managed architecture - the focus is on how to minimize the OpEx as much as possible.

If so, why the infrastructure roles are valuable nonetheless?

## <span class="fontColorH2">Lightweight Versus Overweight</span>

Apparently, most organizations have many histories/considerations that need to look after, hence they cannot move forward to the cloud world easily. Sometimes you want to go faster, but someone stops you due to something you need to bear in mind - those accumulated histories/considerations, over years than years.

However, the reality is that you cannot stop the trend of infrastructure modernization. Therefore, they have to move forward, no matter how many difficulties need to overcome. For this reason, our values/contributions are presented in an entirely different way.

- IaaS, PaaS, or SaaS? Which one is the most ideal platform - although we no longer involve that how many networking gears, servers, and storages required, we still could contribute those experiences 1) to evaluate if the Web server can be replaced with the S3 Website Endpoint, 2) to investigate if the API server can be replaced with the SAM model, and 3) to assess if the database can run on the RDS. Instead of launching all of them on the EC2 instances. That is because we know what the baseline is.
- With downtime or without it? Which one has the minimum compromise - plan a robust migration detail that is definitely what we are good at. The migration experience we have accumulated could be applied anywhere due to the requirement does not have any difference between the environments (on-premises/cloud). It is always 1) full downtime, 2) minimum downtime, and 3) without downtime. We do know the Business is not willing to see the downtime for every migration, as a result, we do know how to manage every migration as smoothly as possible.
- As the matter of fact, the infrastructure technologies are everywhere - there is no significant difference between some of the managed services and self-managed architecture due to they use the same domain expertise behind the scene for instance; FSx for Windows File Server and SMB. Therefore, we could take up these kinds of services very quickly. Also, some of the strategies still inherit from the common technique for instance; to avoid the database freeze when performing the backup, you would not consider the schedule EBS snapshot, you would prefer shipping the transaction log instead.

This domain knowledge will not be gone easily due to it is still in use over the world nowadays.

## <span class="fontColorH2">Transitions</span>

### <span class="fontColorH3">To Be Involved Different Territories</span>
Since you no longer act as an infrastructure owner, an infrastructure consumer instead, therefore, <span class="fontColor">it is definitely not enough if you are still only familiar with what you have been good at</span>. Unlike the on-premises infrastructure that has a precise ownership boundary, the cloud world is opposite - very blurry instead.

Here is an easy-to-see and clear case - launch an EC2 instance.

- First of all, you need to decide which type of the series is more suitable for the application **(Server)**; *General Purpose*? Or *Compute/Memory/Storage Optimized*?
- Secondly, you need to think about which type of EBS volume would meet the application requirement **(Storage)**; *gp* or *io*?
- Thirdly, you need to ensure that this instance will be able to be accessed once provisioned, therefore, which type of Security Group logic would be more efficient **(Security)**; *IP address* or *self-reference rule*?
- Lastly, this instance might need to be behind a load-balancer, hence you need to figure out which type of the load-balancer would be accurate **(Networking)**; *Application Load Balancer* or *Network Load Balancer*?

I am not even pointing out if this instance needs to have special permission to access other resources without the credentials **(Security)** - *Instance Profile*. Obviously, you have not faced this kind of composition in the on-premises world due to each space is managed by different owners.

### <span class="fontColorH2">Try To Be A DevOps</span>

<span class="fontColor">Since you no longer act as an infrastructure owner, an infrastructure consumer instead, therefore, it is definitely not enough if you are still only familiar with what you have been good at.</span> As a result, you need to think about how to make this next-generation operation as more efficiently as possible. Then you will be aware that the best way is to streamline it - automation. Every deployment has its own pipeline for either patch, fallback, or both, and without minimum or even zero human intervention. In the real world, you may not even aware that you have transited successfully. Therefore, <span class="fontColor">do not be afraid to change your mindset due to you even do not know how excellent you will be</span>.

As of today, we know *Ansible*, *Python*, and *Terraform* are very popular languages to streamline your routine jobs. However, do you really need to get involved in all of them? Not at all for sure. They are just tools for simplifying your routines. The DevOps does not mean that you have to get involved in those modern languages, instead, it is more about <span class="fontColor">how do you leverage the toolset that you have been familiar with then streamlining several manual clicks and repetitions</span>.

## <span class="fontColorH2">Conclusion</span>

I am certainly proud that I am an infrastructure guy due to it is where I come from. However, the reality is that you cannot stop the world transition, therefore, you need to or even have to follow up with it. We do not know when will next divide come up so that what we can do is accept it positively.

:::info
###### tags: `AboutMe` `Career`
:::

{%hackmd BJrTq20hE %}