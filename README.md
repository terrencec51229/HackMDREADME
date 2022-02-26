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

# Migrate On-premises Workloads To AWS

[TOC]

## <span class="fontColorH2">Outline</span> 

Because of the following reasons (mainly from my aspect), more and more organizations have considered to move and launch their businesses on the cloud. 

1. <span class="fontColor">++Pay as you go++</span> - Obviously, it is absolutely attractive from the cost management perspective. The payment is only asked whenever you launch any of the services on the cloud.

    > Not everything is covered by this charge model. <span class="fontColor">The exception is the storage</span>. When you turned off the EC2 instances, you would not charged any fees of the computing resources (the license of the OS, CPU, and Memory) until you turn them up. However, the disk space was allocated so that it cannot avoid to be billed.

2. ++Unlimited resources: less depedencies++ - How long does a set of product or service be deployed in the on-premises environment? The Product team only cares how quick the Infrastructure team could fulfill their requirement. However, from the Infrastructure team aspect, they have to ensure all of their managed resources are enough to be deployed beforehand. What if the resources lack? Does the Product team accept the compromise? How long does the new procurement be in place? Nevertheless, you certainly no longer need to take above-mentioned points into account in the cloud world due to they are fully managed by each CSP. The most importantly, you only need to focus on what is the most cost-effective/full-tolerance design then GO afterward.
3. ++Rapid deployment: more flexibilities++ - You could provision anything whenever and wherever you are, then decommission everything once they are no longer required. All of your deployments are able to under multiple AZs or even multiple regions easily. Do you want to build a new environment in the Tokyo region? Do you plan to stretch your businesses across multiple regions? If so, what you need to do is JUST GO.

Besides everything is from scratch on the cloud, the quickest/easiest way is to have a copy there. Therefore,  not only the AWS-native services but also several 3^rd^ party solutions are able to make it happen and worth to keep an eye on as well. So, let's get started.

:::success
:bulb: *Since my environment is primarily composed of Windows Server, Linux, and MS-SQL, and all of them are launched on/managed by the VMware vSphere platform, in this manner, all of my elaborations are based on this background.*
:::

## <span class="fontColorH2">Conversion-dependent</span> 

### <span class="fontColorH3">AWS: VM Import</span>

VM Import is a conversion tool for transforming the on-premises virtual machine format (ex: vmdk) into the Amazon Machine Images (AMI) format. In reverse way is VM Export.

:::success
:bulb: *Since this post focuses on how to migrate everything to the cloud rather than how to return them to the on-premises environment, hence VM Export is not covered here.*
:::

VM Import could be manually launched via [AWS CLI](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html) or automatically launched by either [Server Migration Service (SMS)](https://docs.aws.amazon.com/server-migration-service/latest/userguide/server-migration.html) or the 3^rd^ party backup platforms such as [Commvault](https://documentation.commvault.com/commvault/v11/article?p=30911.htm) and [Rubrik](https://www.rubrik.com/en/blog/technology/19/7/disaster-recovery-developer-environments).

However, every conversion cannot avoid any of compatible failures due to it is its nature. For instance, unsupported format of the import file. The failure trace is sometimes easy to understand then rectify afterward, but it is not sometimes such as [FirstBootFailure](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-troubleshooting.html#windows-vm-errors). For this reason, AWS offers [VMImportChecker](https://kb.msp360.com/cloud-vendors/amazon-aws/ensuring-vm-compatibility-with-ec2) for listing all the possibilities of failure.

> <span class="fontColor">The output of VMImportChecker might not point out the exact cause(s), it is more about to give you a guidance instead.</span> In my cases, I did have successful conversions with a plenty of FAILED warnings and failed conversisons without any FAILED warnings.

### <span class="fontColorH3">AWS: VM Import - Server Migration Service</span>

As its name, Server Migration Service (SMS) is used for migrating your on-premises virtual machines which managed by either VMware vSphere or Microsoft Hyper-V. 
How it functions? SMS leverages SMS Connector to link up the on-premises hypervisor with your target AWS environment. <span class="fontColor">Each SMS Connector can only fasten a single account/region</span> so that means you will need more than one SMS Connectors if your target is multiple.

SMS Connector is a virtual appliance as well as all of your virtual machines, nothing special. After initialized, you no longer need to take care of it due to every manipulation will be performed on AWS Console instead. [How can I initialize it?](https://docs.aws.amazon.com/server-migration-service/latest/userguide/VMware.html) Intrinsically, the whole process is just composed of several next-clicks, hence you do not really need to prepare any instructions in advance in that the intention of each step is pretty straightforward to be understood.

Here is how it functions in detail once the initialization is complete.

![VMIE: Server Migration Service](https://i.imgur.com/S6HyuLw.jpg)

1. When the required replication jobs are defined on SMS, it will liaise with SMS Connector to follow up once the jobs are launched by either an one-time or a schedule event.
2. When SMS Connector is informed, it will ask vCenter to take the snapshots of all the target virtual machines then collect all of them afterward.
3. SMS Connector uploads those snapshots to the S3 bucket created by SMS automatically.
4. SMS calls VM Import to fetch those snapshots, convert all of them, and validate if the corresponding EBS Snapshot(s) and AMI are able to be generated.

### <span class="fontColorH3">AWS: VM Import - 3^rd^ Party</span>

Generally speaking, most of well-known backup platforms have supported to protect the workload or data to the cloud. In essence, the back-end technology should not have too much differences due to it is just to protect the targets to another place from my perspective. As a result, let's directly jump in how they function in detail once the initialization is complete and what does it differ SMS as well.

![VMIE: 3rd Party Backup](https://i.imgur.com/8cfR9qL.jpg)

1. When the required replication jobs are defined on the backup platform, it will ask vCenter to take the snapshots of all the target virtual machines then collect all of them afterward. Typically, all the backup platforms would compare the current snapshot with previous one then generate a new differential file before uploading. This behavior is also called differential or incremental backup.

    > :dart: SMS will upload all the snapshots immediately once collected due to it is just a pure migration tool.

2. The backup platform uploads those differential snapshots to the S3 bucket you specified.

    > :dart: SMS cannot specify the target bucket.

3. The backup platform launches its converters and informs them to fetch those snapshots then convert all of them afterward. Typically, those backup platforms do not rely on VM Import for conversion, their own conversion architecture instead.
4. Once the conversions are completed, the converters will inform VM Import to validate if the corresponding EBS Snapshot(s) and AMI are able to be generated.

    > :dart: SMS fully relies on VM Import for both conversion and validation. In addition, each SMS Connector can only fasten a single account/region. That means you will need more than one SMS Connectors if your target is multiple. Those backup platforms are able to link up with multiple accounts/regions by their  job management without managing more than one platforms; for instance the workloads that belong to the 1^st^ job will be replicated to the 1^st^ account/region, the workloads that belong to the 2^nd^ job will be replicated to the 2^nd^ account/region, and etc. 

### <span class="fontColorH3">AWS: CloudEndure</span>

Essentially, CloudEndure acts as next-generation of SMS. More accurately, AWS has officially promoted it as the most ideal migration platform rather than SMS. There are two operation models that CloudEndure renders, [Migration](https://aws.amazon.com/cloudendure-migration/) and [Disaster Recovery](https://aws.amazon.com/cloudendure-disaster-recovery/). However, from its architecture perspective, there is no difference in between. When I investigated into the document again then I found out the key difference might be about [MAP (Migration Accleration Program)](https://docs.cloudendure.com/#Introduction/Introduction.htm#Understanding_the__Migration_Solution%3FTocPath%3DNavigation%7CIntroduction%7CCloudEndure%2520Solutions%7C_____1) <span class="fontColor2">~#1~</span>. When you receive the details of MAP Project; for instance the server-id mapping (on-premises vs. cloud), CloudEndure Migration is able to import this information for migration afterward <span class="fontColor2">~#2~</span>.

:::spoiler <span class="fontColor2">What is AWS MAP in overview</span>
> 1. The [AWS MAP](https://aws.amazon.com/migration-acceleration-program/) is a comprehensive consulting service when the organization decides to kick off the journey of cloud migration. There are abundant prerequisites will take place afterward such as discussions (++what are the business drivers++), confirmations (++what have you done for the strategy/technical design before the journey++), and documentation (++how many things that have been well-documented and able to share with them++). The following screeshot shows up how many categories (and some of the questions) that the AWS MAP team covers. <p>![MRA Customer](https://i.imgur.com/9TKR24a.png)<p/>Typically, a pure DR environment would be a cold site and also, its strategy would prefer keeping minimum compute resource as possible due to cost-driven. In other words, the hot DR environment will only be switched on by demand. As a result, you will not kick of MAP because of the DR requirement in general.
> 2. <span class="fontColor">The only supported service that could be launched by CloudEndure is EC2.</span> In other words, it is more suitable if you have not considered to re-architect the application framework, keep the strategy of a virtual machine replaces with an EC2 instance instead. 
:::

How does CloudEndure launch the on-premises workload on the cloud? Conversion, as well as SMS and the 3^rd^ party platforms (I will call them VM Import based solutions for simplicity). However, <span class="fontColor">CloudEndure completely decouples VM Import</span> <span class="fontColor2">~#3~</span>, it does not rely on VM Import for both conversion and validation.

![Proprietary Conversion](https://i.imgur.com/ZIXg4qo.png)

:::spoiler <span class="fontColor2">A black hole - VM Import</span>
> - A successful conversion composes of two pieces - conversion and validation. Although the piece of conversion is successfully complete, the AMI and its EBS Snapshot(s) still cannot be generated without passing the piece of validation. That is a case that our environment has encountered as yet. The conversions were complete by the VM Import based solutions, however, none of them were able to pass the VM Import validation. It did not happen all the time, randomly instead albeit the targets were the same virtual machines. What we have done is collaborating with Support of the 3^rd^ party platform for verifying if anything changed for calling VM Import at first, then escalating to AWS Support for verifying if anything changed for the behavior of VM Import afterward. Unfortunately, there was no any good news after a long time investigation. The weird situation we were in was the issue resolved by itself and AWS Support confirmed that nothing changed.
> - During this failure period, we did confirm that CloudEndure did not rely on VM Import as highlighted on the white paper due to all the failure cases at that moment were able to form the AMI/EBS Snapshot(s).
:::

CloudEndure composes of three components - agent, replication server, and converter. The behaviours of replication server and converter are identical to those VM Import based solutions. The replication server accounts for uploading the snapshots to the S3 bucket it manages and the converter accounts for fetching the snapshots from the bucket then forming the AMI/EBS Snapshot(s) afterward.

So, what about the agent? Unlike those VM Import based solutions are natively integrated with vCenter for manipulating all the virtual machines, <span class="fontColor">CloudEndure leverages its agents which installed on all the target workloads to register all the target workloads on its management console</span>. Once the register completes, you are able to define the replication schedule or launch the recovery procedure afterward.

Intrinsically, what SMS does is ++to convert the snapshot(s) into the AWS format++, and what those 3^rd^ party platforms do is ++not only to perform the conversion but also able to launch the converted resource(s) by demand++. Other than the independency of VM Import, what CloudEndure primarily differs them is the recovery plan. From my perspective, CloudEndure is in more VMware SRM (Site Recovery Manager) alike design due to the following thoughts.

1.  The converted resource(s) could be launched by either the [Test](https://docs.cloudendure.com/#Configuring_and_Running_Disaster_Recovery/Testing_the_Distaster_Recovery_Solution/Testing_the_Disaster_Recovery_Solution.htm#Testing_the_Disaster_Recovery_Solution%3FTocPath%3DNavigation%7CConfiguring%2520and%2520Running%2520Disaster%2520Recovery%7CTesting%2520the%2520Disaster%2520Recovery%2520Solution%7C_____0) or [Recovery](https://docs.cloudendure.com/#Configuring_and_Running_Disaster_Recovery/Performing_a_Disaster_Recovery_Failover/Performing_a_Disaster_Recovery_Failover.htm#Performing_a_Disaster_Recovery_Failover_and_Failback%3FTocPath%3DNavigation%7CConfiguring%2520and%2520Running%2520Disaster%2520Recovery%7CPerforming%2520a%2520Disaster%2520Recovery%2520Failover%2520and%2520Failback%7C_____0) mode albeit it is for indication only. <p>![Test-versuses-Recovery](https://i.imgur.com/kT4WYFl.png)<p/>
2. It is not only offering the package of failing over to the cloud but also failing back to the on-premises vCenter. <p>![CESM](https://i.imgur.com/db1opRo.png)<p/>

Here is how it functions in detail once the initialization is complete.

![No VM Import: CloudEndure](https://i.imgur.com/o78BXgA.jpg)

1. Install the CloudEndure agent on each target that needs to be protected. The agent functions immediately once deployed, there is no need to reboot the system to bring it up.
2. Once the agent successfully registers on the CloudEndure console then you are able to customize the replication plan for each individual workload or a group of them afterward.
3. When the replication plan launches by either an one-time or a schedule event, the replication server will be waken up then fetching the snapshot from the source workload via the agent. Afterward, the converter will be waken up then converting the snapshot into the EBS Snapshot format. <span class="fontColor">Nothing about the restoration in this stage, the preparation for it instead.</span>

    > :dart: Unlike those VM Import based solutions will form both the AMI/EBS Snapshot(s), <span class="fontColor">CloudEndure generates the EBS Snapshot(s) only</span>.

4. When either the Test or Recovery event launches, a couple of the tasks will be taken place. Firstly, the EBS Volume(s) will be created by the EBS Snapshot(s) you specified. Secondly, the EC2 instance will be established with the instance-type you specified, then attaching the EBS Volume(s) created by the previous step.

### <span class="fontColorH3">Comparisons</span>

The following table primarily summarizes what they differ each other.

| Item                    | SMS                                 | 3^rd^ Party                              | CloudEndure                                                                                                                                                            |
| ----------------------- | ----------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dependency of VM Import | Yes                                 | Partial (validation)                     | <span class="fontColor">No</span>                                                                                                                                      |
| Output(s)               | AMI/EBS Snapshot(s)                 | AMI/EBS Snapshot(s)                      | EBS Snapshot(s)                                                                                                                                                        |
| Integration of vCenter  | Yes (go cloud)                      | Yes (go cloud)                           | Yes (fallback)                                                                                                                                                         |
| Fallback of on-premises | Not supported                       | <span class="fontColor">Supported</span> | <span class="fontColor">Supported</span>                                                                                                                               |
| Cost of service itself  | <span class="fontColor">Free</span> | Charged                                  | [Conditional free (Migration)](https://aws.amazon.com/cloudendure-migration/pricing/) or [Charged (DR)](https://aws.amazon.com/cloudendure-disaster-recovery/pricing/) |

## <span class="fontColorH2">Conversion-independent</span> 

### <span class="fontColorH3">3^rd^ Party: VMware Cloud on AWS</span>

Other than those conversion-based solutions, do we have another option(s) if the following drivers are being taken into account.

1. ++Minimize and even avoid the change(s)++: I neither need any fancy cloud-native services nor want to re-architect the application framework. What I need is just a place to run the business on the cloud as soon as possible instead.
2. ++Keep consistency++: This cloud platform is able to seamlessly integrate with what we manipulated in the on-premises environments. I do not want to panic the existing operation model.

If so, I believe that [VMware Cloud on AWS](https://cloud.vmware.com/vmc-aws/use-cases) (VMConAWS) would be the most ideal solution accordingly. As of today, VMware Cloud has been accomplished not only on AWS but also on [Azure VMware Solution](https://azure.microsoft.com/en-us/services/azure-vmware/) and [Google Cloud VMware Engine](https://cloud.google.com/vmware-engine). Let's dig into each above-mentioned driver in more details.

#### <span class="fontColorH4">Minimize and Even Avoid The Change(s)</span>

There are two methodologies for migrating the on-premises workloads to the cloud - Site Recovery Manager (SRM) and Hybrid Cloud Extension (HCX). In essence, you could treat HCX as a free edition of SRM due to they have something in common.

1. From the viewpoint of replication, there is no difference in between due to they rely on vSphere Replication (vR) to function.
2. They are running on a pair structure, that is one SRM/HCX reside in the on-premises environment and the other ones reside on the SDDC.

Typically, their replication behavior do not have too much differences with those conversion-based solutions due to they adopt the same ways as well as them - 1) define the rules, 2) fetch the files, 3) replicate or upload those files to the remote end, and 4) launch the recovery procedure. Since it is a VMware-to-VMware solution, therefore, <span class="fontColor">none of the conversions are required and all the protected targets are able to wake up immediately</span>.

![No VM Import: VMware Cloud on AWS, Replication](https://i.imgur.com/akkmx5p.png)

However, when you compare them in more details then you still can find out they are slightly different; for instance... 

1. HCX is a free add-on, but SRM is a [subscription](https://cloud.vmware.com/vmware-site-recovery/pricing) add-on.
2. HCX can only protect up to [500](https://configmax.vmware.com/guest?vmwareproduct=VMware%20HCX&release=VMware%20HCX&categories=41-0,42-0,43-0,44-0,45-0) virtual machines.</span> As a result, you need to additionally leverage SRM if the total workload that needs to be protected is over 500.
3. The architecture of HCX is fully decoupled, both the control (HCX Interconnect/Network Extension) and the management (HCX Manager) plans are separated. By contrast, SRM does not adopt this design.

![No VM Import: VMware Cloud on AWS, HCX Deep Dive](https://i.imgur.com/kNiMQST.jpg)

> The original diagram is from [viktorious.nl](https://www.viktorious.nl/2019/12/11/a-closer-look-at-hcx-in-combination-with-vmconaws-part-1/).

:::spoiler <span class="fontColor2">Live migration - vCenter Hybrid Link Mode</span>
> - As its name, HCX is purely designed for the cloud migration. Therefore, what it can do is not only just the off-site migration <span class="fontColor">but also the live migration</span>. <span class="fontColor">Yes, that is vMotion.</span> In order to get there, there is one more prerequisite needs to be in place beforehand - [vCenter Hybrid Link Mode (HLM)](https://docs.vmware.com/en/VMware-Cloud-on-AWS/services/com.vmware.vsphere.vmc-aws-manage-data-center-vms.doc/GUID-91C57891-4D61-4F4C-B580-74F3000B831D.html). What vCenter HLM does is to integrate the SSO domain of the on-premises vCenter with the SDDC vCenter so that we are able to manage all the virtual machines aross the environments by a single vSphere console.  <p>![No VM Import: VMware Cloud on AWS, vMotion](https://i.imgur.com/ULT5lDV.png)<p/>
> - The live migration relies on the Network Extension feature or more accurately, this feature spreads the L2 domain of the on-premises network. As a result, <span class="fontColor">both the inbound/outbound connections are still proceeded via the on-premises resources (ex: firewall/load-balancer)</span>.
:::

#### <span class="fontColorH4">Keep Consistency</span>

One of the primary cloud strategies is to use the service instead of manage them. The entire VMConAWS is fully managed by VMware (vSphere API) and AWS (bare metal), hence VMConAWS has not opened several features that have not been restricted in the on-premises vSphere environment; for instance you will not have the root permission and none of the external datastores can be mounted by yourself. The main driver for those rules is to make every SDDC is consistent so that VMware Support is able to address the issue as quick as possible.

Other than those cloud-native restrictions, there is nothing different with the on-premises vSphere environment, and none of the familiarities are from zero as well.

#### <span class="fontColorH4">Comparisons</span>

The following table primarily summarizes what VMConAWS differs those conversion-based solutions.

| Item                    | SMS                                 | 3^rd^ Party                              | CloudEndure                                                                                                                                                            | VMConAWS                                    |
| ----------------------- | ----------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| Conversion              | Required                            | Required                                 | Required                                                                                                                                                               | <span class="fontColor">Not required</span> |
| Dependency of VM Import | Yes                                 | Partial                                  | <span class="fontColor">No</span>                                                                                                                                      | <span class="fontColor">No</span>           |
| Output(s)               | AMI/EBS Snapshot(s)                 | AMI/EBS Snapshot(s)                      | EBS Snapshot(s)                                                                                                                                                        | No                                          |
| Integration of vCenter  | Yes (go cloud)                      | Yes (go cloud)                           | Yes (fallback)                                                                                                                                                         | Yes                                         |
| Fallback of on-premises | Not supported                       | <span class="fontColor">Supported</span> | <span class="fontColor">Supported</span>                                                                                                                               | <span class="fontColor">Supported</span>    |
| Cost of service itself  | <span class="fontColor">Free</span> | Charged                                  | [Conditional free (Migration)](https://aws.amazon.com/cloudendure-migration/pricing/) or [Charged (DR)](https://aws.amazon.com/cloudendure-disaster-recovery/pricing/) | Charged                                     |
| Seamless operation      | No                                  | No                                       | No                                                                                                                                                                     | <span class="fontColor">Yes</span>          |

:::info
###### tags: `AWS` `VMwareCloud` `Architecture` `Migration` `VMImport` `SMS` `CloudEndure` `HCX`
:::

{%hackmd BJrTq20hE %}