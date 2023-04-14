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

Because of the following reasons (mainly from my aspect), more and more organisations have considered moving and launching their businesses on the cloud.

1. <span class="fontColor">++Pay as you go++</span> -  Obviously, it is absolutely attractive from the cost management perspective. The payment is only asked whenever you launch any of the services on the cloud.

    > One thing needs to keep in mind is that not everything is covered by this charge model due to <span class="fontColor">the storage is an exception</span>. When you turned off an EC2 instance, you would not be charged for compute resources, e.g. CPU, memory, or the OS license, until you turn them up; however, the disk space is allocated so that it would be charged nonetheless.

2. ++Unlimited resources (fewer depedencies)++ -  How long does a set of products or services to be deployed in the on-premises environment? The Product team only cares about how quickly the Infrastructure team could fulfil their requirement.  However, from the Infrastructure team aspect, they have to ensure that all of their managed resources are sufficient to be occupied beforehand. What if the resource is absent? Does the Product team accept any compromise? How long does the new procurement to be in place? Luckily, you no longer need to take the above-mentioned points into account in the cloud world due to they are completely managed by each CSP. In other words, you just need to focus on what is the most cost-effective/full-tolerance design and carry it out afterwards.

3. ++Rapid deployment (more flexibility)++ - You could provision anything whenever and wherever you are, then decommission everything once they are no longer required. All of your applications are able to serve across multiple AZs and even regions easily. Do you aim to launch a new application in a single region? Or even,  do you plan to stretch your business across multiple regions? If so, what you need to do is just carry them out without any constraint.

Besides building everything from scratch on the cloud, the quickest/easiest way is to have a copy there. Therefore, not only AWS-native services but also several 3rd party solutions are able to make it happen and worth keeping an eye on as well.

So, let us get started.

:::success
:bulb: *Since most enterprises primarily got started from virtual machines, which were launched on and managed by the VMware vSphere platform; in this manner, all of my elaborations are based on this background.*
:::

## <span class="fontColorH2">Conversion-dependent</span> 
### <span class="fontColorH3">VM Import</span>

VM Import is a conversion tool for transforming the on-premises virtual machine format (vmdk) into the Amazon Machine Images (AMI) format. In the reverse direction is VM Export.

:::success
:bulb: *Since this post focuses on how to migrate everything to the cloud rather than how to return them to the on-premises environment, hence VM Export is not covered here.*
:::

VM Import could be manually launched via [AWS CLI](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-image-import.html) or automatically launched by either [Server Migration Service (SMS)](https://docs.aws.amazon.com/server-migration-service/latest/userguide/server-migration.html) or the 3^rd^ party backup platforms, e.g. [Commvault](https://documentation.commvault.com/commvault/v11/article?p=30911.htm) and [Rubrik](https://www.rubrik.com/en/blog/technology/19/7/disaster-recovery-developer-environments). Compatibility failure is a thing that every conversion cannot avoid due to it is its nature; for instance, an unsupported format of the import file. The failure trace is sometimes clear to understand and rectify, but it is not always; here is an example, [FirstBootFailure](https://docs.aws.amazon.com/vm-import/latest/userguide/vmimport-troubleshooting.html#windows-vm-errors). For this reason, AWS offers [VMImportChecker](https://kb.msp360.com/cloud-vendors/amazon-aws/ensuring-vm-compatibility-with-ec2) to  tabulate all the potential possibilities.

However, one thing needs to keep in mind is that <span class="fontColor">the output of VMImportChecker might not point out the exact cause due to it is more about giving you guidance instead</span>. In my cases, I did have successful conversions with plenty of FAILED warnings and failed conversions without any FAILED warnings.

### <span class="fontColorH3">VM Import - Server Migration Service</span>

As its name, Server Migration Service (SMS) is used for migrating your on-premises virtual machines managed by either VMware vSphere or Microsoft Hyper-V. How does it function? SMS leverages SMS Connector to link up the on-premises hypervisor with your target AWS environment. <span class="fontColor">Each SMS Connector can only fasten a single account per region</span> so that means you will need more than one SMS Connector if your target is multiple.

SMS Connector is a virtual appliance as well as your virtual machines, nothing special. After the SMS Connector initialised, you no longer need to take care of it due to every manipulation will be performed on the AWS Console instead. [How can I initialise it?](https://docs.aws.amazon.com/server-migration-service/latest/userguide/VMware.html) Intrinsically, the whole process just composes of several clicks, hence you do not really need to prepare any instruction beforehand in that the intention of each step is pretty straightforward to understand.

Here is how it functions in detail once the initialisation completes.

![VMIE: Server Migration Service](https://i.imgur.com/S6HyuLw.jpg)

1. When the required replication jobs are defined on the SMS, it will liaise with the SMS Connector to follow up once the jobs are launched by either a one-time or a scheduled event.
2. When the SMS Connector is informed, it will ask the vCenter to take snapshots of all the target virtual machines and collect them afterwards.
3. SMS Connector uploads those snapshots to the S3 bucket created by SMS automatically.
4. SMS calls the VM Import API to fetch those snapshots, convert them, and validate if the corresponding EBS Snapshots and AMIs are able to be established.

### <span class="fontColorH3">VM Import - 3^rd^ Party</span>

Generally speaking, most well-known backup platforms have supported to protect workloads or data to the cloud. In essence, the back-end technology should not have too many differences due to it is just to protect the targets to be launched in another place from my perspective; as a result, let us directly jump into how they function in detail once the initialisation completes and what does it differ from the SMS as well.

![VMIE: 3rd Party Backup](https://i.imgur.com/8cfR9qL.jpg)

1. When the required replication jobs are defined on the backup platform, it will ask the vCenter to take snapshots of all the target virtual machines and collect them afterwards. Typically, all the backup platforms compare the current snapshot with the previous one and generate a new differential file before uploading or replicating it to somewhere. This behaviour is also called differential or incremental backup.

    > SMS will upload all the snapshots immediately once collected.

2. The backup platform uploads those differential snapshots to the S3 bucket you specified.

    > SMS cannot specify the target bucket.

3. The backup platform launches its converters, informs them to fetch those snapshots, and converts them afterwards. Typically, those backup platforms do not rely on the VM Import API for conversion, their own system architecture instead.
4. Once the conversions are completed, the converters will inform the VM Import API to validate if the corresponding EBS Snapshots and AMIs are able to be established.

    > SMS fully relies on the VM Import API for both conversion and validation. In addition, each SMS Connector can only fasten a single account per region; meaning you will need more than one SMS Connector if you have multiple targets. Those backup platforms are able to link up multiple accounts across regions with their system console without managing multiple platforms, e.g., the workloads that belong to the 1^st^ job will be replicated to the 1^st^ account and region, and the workloads that belong to the 2^nd^ job will be replicated to the 2^nd^ account and region.

### <span class="fontColorH3">CloudEndure</span>

Essentially, CloudEndure acts as the next generation of SMS; or even more accurately, AWS has promoted it as the primary migration tool rather than SMS. There are two operation models CloudEndure renders, [Migration](https://aws.amazon.com/cloudendure-migration/) and [Disaster Recovery](https://aws.amazon.com/cloudendure-disaster-recovery/); however, there is no difference between them from the architectural perspective. When I investigated the document again then I found out the key might be correlated with [Migration Acceleration Program (MAP)](https://docs.cloudendure.com/#Introduction/Introduction.htm#Understanding_the__Migration_Solution%3FTocPath%3DNavigation%7CIntroduction%7CCloudEndure%2520Solutions%7C_____1). When you receive the details of the MAP Project, e.g. the server-id mapping (on-premises vs. cloud), CloudEndure Migration is able to import this information as its inventory, but CloudEndure Disaster Recovery does not support this feature.

> ***What is MAP in an overview***
> 
> - First of all, the [MAP](https://aws.amazon.com/migration-acceleration-program/) is a comprehensive consulting package for any organisation deciding to kick off its cloud migration journey. There are abundant prerequisites that will take place, e.g. discussions (++what are the business drivers++), confirmations (++what have you done for strategy and technical design before the journey++), and documentation (++how many conversations have been well-archived++). The following screenshot shows up how many categories and some questions the MAP team covers. <p>![MRA Customer](https://i.imgur.com/9TKR24a.png)<p/>
> - Typically, the DR environment would be a cold site; in addition, its strategy would prefer keeping minimum compute resources as possible due to cost consideration. In other words, the DR environment will only be switched on by demand. Generally speaking, you will not kick off the MAP just because of the DR requirement.

How does CloudEndure launch the on-premises workload on the cloud? Conversion, as well as SMS and the 3rd party backup platforms (I will call them the VM Import based solutions for simplicity); however, <span class="fontColor">CloudEndure completely decouples the VM Import API</span>, it does not rely on it for both conversion and validation.

![Proprietary Conversion](https://i.imgur.com/ZIXg4qo.png)

> ***A black hole - VM Import***
> 
> - A successful conversion composes of two pieces - conversion and validation. Although the piece of conversion is successfully complete, the AMI and its EBS Snapshot(s) still cannot be established without passing the piece of validation. There is a case that our customer has encountered as yet. The conversions were complete by the VM Import based solutions, however, none of them was able to pass the validation by VM Import. It did not happen all the time, randomly instead albeit the targets were the same virtual machines. What we have done is collaborate with Support of the 3rd party platform for verifying if anything changed for calling the VM Import API at the outset, then escalating to AWS Support for verifying if anything changed for the behaviour of the VM Import API afterwards. Unfortunately, there was no good news after a long time investigation. The weird situation we were in was the issue resolved by itself and AWS Support confirmed that nothing changed.
> - During this failure period, we did confirm that CloudEndure did not rely on VM Import as highlighted on the white paper due to all the failure cases at that moment were able to form the AMI and EBS Snapshot(s).

CloudEndure composes of three components - agent, replication server, and converter. The behaviours of the replication server and converter are identical to those VM Import based solutions. The replication server accounts for uploading the snapshots to the S3 bucket it manages and the converter accounts for fetching the snapshots from the bucket and forming the AMIs and EBS Snapshots afterwards.

So, what about the agent? Unlike those VM Import based solutions that are natively integrated with the vCenter for manipulating virtual machines, <span class="fontColor">CloudEndure leverages its agents which are installed on all the target workloads to register all the target workloads on its system console</span>. Once the registration process completes, you are able to define either the replication schedule or launch the recovery procedure.

Intrinsically, what SMS does is ++convert the snapshots into the AWS format++, and what those 3rd party backup platforms do is ++not only perform the conversion but also launch the converted resources by demand++. Other than decoupling the dependency of VM Import, what CloudEndure primarily differs from those VM Import based solutions is the recovery plan. From my perspective, CloudEndure is designed from the VMware SRM (Site Recovery Manager) framework due to the following thoughts.

1.  The converted resources could be launched by either the [Test](https://docs.cloudendure.com/#Configuring_and_Running_Disaster_Recovery/Testing_the_Distaster_Recovery_Solution/Testing_the_Disaster_Recovery_Solution.htm#Testing_the_Disaster_Recovery_Solution%3FTocPath%3DNavigation%7CConfiguring%2520and%2520Running%2520Disaster%2520Recovery%7CTesting%2520the%2520Disaster%2520Recovery%2520Solution%7C_____0) or [Recovery](https://docs.cloudendure.com/#Configuring_and_Running_Disaster_Recovery/Performing_a_Disaster_Recovery_Failover/Performing_a_Disaster_Recovery_Failover.htm#Performing_a_Disaster_Recovery_Failover_and_Failback%3FTocPath%3DNavigation%7CConfiguring%2520and%2520Running%2520Disaster%2520Recovery%7CPerforming%2520a%2520Disaster%2520Recovery%2520Failover%2520and%2520Failback%7C_____0) mode albeit it is for indication only. <p>![Test-versuses-Recovery](https://i.imgur.com/kT4WYFl.png)<p/>
2. It not only offers the fail-over feature to the cloud but also renders the fail-back feature to the on-premises vCenter. <p>![CESM](https://i.imgur.com/db1opRo.png)<p/>

Here is how it functions in detail once the initialisation completes.

![No VM Import: CloudEndure](https://i.imgur.com/o78BXgA.jpg)

1. Install the CloudEndure agent on each target that needs to be protected. The agent functions immediately once deployed, there is no need to reboot the system to bring it up.
2. Once the agent successfully registers on the CloudEndure console then you are able to customise the replication plan for each workload or a group of them.
3. When the replication plan launches by either a one-time or a scheduled event, the replication server will be woken up and fetch the snapshot from the source workload via the agent. Thereafter, the converter will be woken up and convert the snapshot into the EBS Snapshot format. <span class="fontColor">Nothing about restoration at this stage, the preparation for it instead.</span>

    > Unlike those VM Import based solutions that will form both the AMIs and EBS Snapshots, <span class="fontColor">CloudEndure creates the EBS Snapshots only</span>.

4. When either the Test or Recovery event launches, a couple of the tasks will be taken place. First of all, each EBS Volume will be created by the EBS Snapshot you specified. Secondly, the EC2 instance will be established with the instance type you specified and attach the EBS Volume(s) created by the previous step.

### <span class="fontColorH3">Comparisons</span>

The following table summarises what they primarily differ from each other.

| Item                    | SMS                                 | 3^rd^ Party                              | CloudEndure                                                                                                                                                            |
| ----------------------- | ----------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dependency of VM Import | Yes                                 | Only required during validation          | <span class="fontColor">No</span>                                                                                                                                      |
| Output                  | AMI and EBS Snapshot                | AMI and EBS Snapshot                     | EBS Snapshot                                                                                                                                                           |
| Integration of vCenter  | Yes                                 | Yes                                      | Yes                                                                                                                                                 |
| Fallback of On-premises | Not Supported                       | <span class="fontColor">Supported</span> | <span class="fontColor">Supported</span>                                                                                                                               |
| Cost                    | <span class="fontColor">Free</span> | Charged                                  | [Conditional free (Migration)](https://aws.amazon.com/cloudendure-migration/pricing/) or [Charged (DR)](https://aws.amazon.com/cloudendure-disaster-recovery/pricing/) |

## <span class="fontColorH2">Conversion-independent</span> 

### <span class="fontColorH3">VMware Cloud on AWS</span>

Other than those conversion-based solutions, do we have another option if the following drivers are taken into account?

1. ++Minimise and even avoid the changes++: I neither need any fancy cloud-native service nor want to revamp the application architecture, instead, what I need is just a place to run the business on the cloud as soon as possible.
2. ++Keep consistency++: The cloud platform is able to seamlessly integrate with what we have manipulated for the on-premises environment. I do not want to panic the existing operation framework.

For those reasons, I personally believe that [VMware Cloud on AWS (VMConAWS)](https://cloud.vmware.com/vmc-aws/use-cases) would be the most ideal solution accordingly. As of today, VMware Cloud has been accomplished not only on AWS but also on [Azure](https://azure.microsoft.com/en-us/services/azure-vmware/), [Google](https://cloud.google.com/vmware-engine) and even [more](https://www.vmware.com/cloud-solutions.html) in the upcoming future. Let us dig into each above-mentioned driver in more detail.

#### <span class="fontColorH4">Minimise and Even Avoid The Changes</span>

There are two methodologies for migrating the on-premises workloads to the cloud - Site Recovery Manager (SRM) and Hybrid Cloud Extension (HCX). In essence, you could treat HCX as a free edition of SRM due to they have something in common.

1. From the viewpoint of replication, there is no difference between SRM and HCX due to they rely on vSphere Replication (vR) to function.
2. They function on a pair structure, that is one SRM/HCX resides in the on-premises environment and another one resides in the SDDC.

Typically, their replication behaviour does not differ from those conversion-based solutions due to they adopt the same ways; define the rules, fetch the files, replicate or upload those files, and launch the recovery procedure. Since it is a VMware-to-VMware manner, therefore, <span class="fontColor">none of the conversions is required and all the protected targets are able to wake up immediately</span>.

![No VM Import: VMware Cloud on AWS, Replication](https://i.imgur.com/akkmx5p.png)

However, when you compare them in more detail then you still can find out they are slightly different.

1. HCX is a free add-on, but SRM is a [subscription](https://cloud.vmware.com/vmware-site-recovery/pricing) add-on.
2. HCX can only protect up to [500](https://configmax.vmware.com/guest?vmwareproduct=VMware%20HCX&release=VMware%20HCX&categories=41-0,42-0,43-0,44-0,45-0) virtual machines. As a result, you need to additionally purchase the SRM license if the total workload that needs to be protected is over 500.
3. The architecture of HCX is fully decoupled, both the control (HCX Interconnect and Network Extension) and the management (HCX Manager) planes are separated. By contrast, SRM does not adopt this design.

![No VM Import: VMware Cloud on AWS, HCX Deep Dive](https://i.imgur.com/kNiMQST.jpg)

> The original diagram is from [viktorious.nl](https://www.viktorious.nl/2019/12/11/a-closer-look-at-hcx-in-combination-with-vmconaws-part-1/).

> ***Live migration - vCenter Hybrid Link Mode***
> - As its name, HCX is designed for cloud migration. What it can do is not only just the off-site migration but also the live migration; <span class="fontColor">Yes, that is vMotion.</span> In order to get there, there is one more prerequisite that needs to be in place beforehand  -  [vCenter Hybrid Link Mode (HLM)](https://docs.vmware.com/en/VMware-Cloud-on-AWS/services/com.vmware.vsphere.vmc-aws-manage-data-center-vms.doc/GUID-91C57891-4D61-4F4C-B580-74F3000B831D.html). What the vCenter HLM does is integrate the SSO domain of the on-premises vCenter with the SDDC vCenter so that we are able to operate all the virtual machines across the environments by a single vSphere console. <p>![No VM Import: VMware Cloud on AWS, vMotion](https://i.imgur.com/ULT5lDV.png)<p/>
> - The live migration relies on the Network Extension feature, or more accurately, this feature spreads the L2 domain of the on-premises network. As a result, <span class="fontColor">both the inbound and outbound connections still handle via the on-premises resources, e.g. firewall or load balancer</span>.

#### <span class="fontColorH4">Keep Consistency</span>

One of the primary cloud strategies is to use the service instead of managing them. The entire VMConAWS is fully managed by VMware (software) and AWS (underlying infrastructure), hence VMConAWS has not opened too many features that have not been restricted in the on-premises vSphere environment; for instance, you would not have the root permission and none of the external datastores can be mounted by yourself. The main driver for those rules is to ensure that every SDDC is consistent so that VMware Support is able to address the issue as quickly as possible.

Other than those cloud-native restrictions, there is nothing different with the on-premises vSphere environment, and none of the familiarities is from zero as well.

#### <span class="fontColorH4">Comparisons</span>

The following table summarises what VMConAWS primarily differs from those conversion-based solutions.

| Item                    | SMS                                 | 3^rd^ Party                              | CloudEndure                                                                                                                                                            | VMConAWS                                    |
| ----------------------- | ----------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| Conversion              | Required                            | Required                                 | Required                                                                                                                                                               | <span class="fontColor">Not Required</span> |
| Dependency of VM Import | Yes                                 | Only required during validation          | <span class="fontColor">No</span>                                                                                                                                      | <span class="fontColor">No</span>           |
| Output                  | AMI and EBS Snapshot                | AMI and EBS Snapshot                     | EBS Snapshot                                                                                                                                                           | VM                                          |
| Integration of vCenter  | Yes                                 | Yes                                      | Yes                                                                                                                                                                    | Yes                                         |
| Fallback of On-premises | Not Supported                       | <span class="fontColor">Supported</span> | <span class="fontColor">Supported</span>                                                                                                                               | <span class="fontColor">Supported</span>    |
| Cost                    | <span class="fontColor">Free</span> | Charged                                  | [Conditional free (Migration)](https://aws.amazon.com/cloudendure-migration/pricing/) or [Charged (DR)](https://aws.amazon.com/cloudendure-disaster-recovery/pricing/) | Charged                                     |
| Seamless Operation      | No                                  | No                                       | No                                                                                                                                                                     | <span class="fontColor">Yes</span>          |

> Published on 26^th^ January 2021. 

:::info
###### tags: `AWS` `VMwareCloud` `Architecture` `Migration` `VMImport` `SMS` `CloudEndure` `HCX`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}