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

# Globally Operate Your Network Transport via Cloud WAN

[TOC]

## <span class="fontColorH2">Retrospect</span>

In my old post, [The Evolution of Cloud Networking on AWS](/MlL4xLyQRgG8PFagIgU7FQ) I elaborated what and why Transit Gateway could revamp your network transport. Although Transit Gateway has been generally available since [November 2018](https://aws.amazon.com/about-aws/whats-new/2018/11/introducing-aws-transit-gateway/) (ready to 4^th^ anniversary), it is still the most powerful feature in the Cloud Networking space across the board.

If you think that Transit Gateway will be the last fascinating networking offering then you are definitely wrong!

## <span class="fontColorH2">New Launch</span>

In July 2022, AWS formally announced another cool feature called [Cloud WAN](https://aws.amazon.com/about-aws/whats-new/2022/07/general-availability-aws-cloud-wan/). As the matter of fact, I was a bit confused about its name due to WAN typically means external/public networks; however, what Cloud WAN is responsible for is not really about WAN, instead, ++it is more about globally consolidate all of your network ingredients, e.g. VPC, Transit Gateway, Site-to-Site VPN, and SD-WAN across regions into a single and unified management console++.

Other than the name, I was also a bit confused about what are the key differences when compared with Transit Gateway in terms of the standpoint of their functionalities especially after I read the [Preview](https://aws.amazon.com/blogs/networking-and-content-delivery/introducing-aws-cloud-wan-preview/) post. As a result, the following questions came up in my brain;

- Is Cloud WAN able to completely take over all the functionalities that Transit Gateway supports?
- Does Cloud WAN aim to replace Transit Gateway entirely?

![AWS Cloud WAN Components](https://i.imgur.com/IFcYMFv.png)

In terms of the standpoint of the key offering, I do not think that AWS is willing to see there are two products without any differentiators. The best way to demystify any uncertainty is always to deploy, verify, and observe instead of reading documents without any implementations.

## <span class="fontColorH2">Demystification</span>
### <span class="fontColorH3">Comparison</span>

From my viewpoint, it would be more straightforward to dig a new feature by comparisons; hence I use the Transit Gateway architecture to compare with the Cloud WAN one. Its architecture involves the following scenarios;

![Transit Gateway Topology](https://i.imgur.com/tls7k9l.png)

- Each Transit Gateway has four Transit Gateway Route Tables which aka VRFs :grin:.
- Cross-region communications are via Transit Gateway Peerings.

When transforming to the Cloud WAN architecture, it looks quite similar to the Transit Gateway one; however, there are a few of differences in between.

![Cloud WAN Topology](https://i.imgur.com/wBFUpsk.png)

1. The role and capability of Core Network Edge or CNE is typically identical to Transit Gateway; the only difference is that ++all the CNEs within the Cloud WAN core network automatically link with each other by nature++. When turning to Transit Gateway, you need to manually create the Transit Gateway Peering for cross-region communications instead.

<p>
    
:::spoiler <span class="fontColor2">:bulb:Transit Gateway Route Table Attachment</span>

As the following screenshots, all kinds of the attachments, e.g. VPC and Site-to-Site VPN are associated with corresponding Core Network Edges as well as the Transit Gateway model; however, you might be interested in what is the Transit Gateway Route Table attachment?

![cloud-wan-attachments](https://i.imgur.com/EidY3Aq.png)

Typically, ++the Transit Gateway Route Table attachment is equivalent to the Transit Gateway Peering attachment++; create the peering connection and then associate it with the segment. In addition, one thing needs to keep in mind is that ++Direct Connect does not support to directly associate with Core Network Edge yet++; therefore, you need to leverage Transit VIF at this stage.
    
![cloud-wan-tgw-peering](https://i.imgur.com/nUx5lUI.png)
    
![cloud-wan-attachments-tgw](https://i.imgur.com/KXBnFrt.png)
:::

</p>

2. Each Cloud WAN segment exactly functions in the same way as Transit Gateway Route Table does; a slight difference in between is that ++you could leverage specified tag set(s) to move any Cloud WAN attachment to another segment++. In my case, I use the key `cloud-wan-segment` as a condition and the value `Development` as a result, that means once this criterion matches, this attachment will be moved to the Development segment. When turning to Transit Gateway Route Table, you need to re-associate the specified attachment instead. From my viewpoint, this feature is extremely handy, especially you do not need to involve any change of the configuration.

![cloud-wan-segment](https://i.imgur.com/Q12IANh.png)
 
3. ++A whole Cloud WAN core network could be completely managed via a JSON file by nature and every change is well-recorded (versioning), too.++ When turning to Transit Gateway, it does not have this feature in place, you need to manage it by your own automated framework instead. In terms of the change management, this feature is a significant spotlight of Cloud WAN from my perspective. Because of the versioning, you are able to roll back any change more efficiently; even before you commit any changes, you are able to see the comparion between the current profile and new profile. This convenience lets me recall Cisco IOS-XR and Junos due to that is what they behave :grin:.

![cloud-wan-verioning](https://i.imgur.com/YgTdf1e.png)

![cloud-wan-editor](https://i.imgur.com/GH9cvbb.png)

<p>
    
:::spoiler <span class="fontColor2">:bulb:My Initial Environment</span>
```shell=JSON
{
  "version": "2021.12",
  "core-network-configuration": {
    "vpn-ecmp-support": true,
    "asn-ranges": [
      "4200000001-4200000100"
    ],
    "edge-locations": [
      {
        "location": "ap-southeast-1"
      },
      {
        "location": "ap-northeast-1"
      }
    ]
  },
  "segments": [
    {
      "name": "Shared",
      "require-attachment-acceptance": true,
      "allow-filter": [
        "Production",
        "Development",
        "Staging"
      ]
    },
    {
      "name": "Production",
      "require-attachment-acceptance": true,
      "allow-filter": [
        "Shared"
      ]
    },
    {
      "name": "Development",
      "require-attachment-acceptance": true,
      "allow-filter": [
        "Shared"
      ]
    },
    {
      "name": "Staging",
      "require-attachment-acceptance": true,
      "allow-filter": [
        "Shared"
      ]
    }
  ],
  "segment-actions": [
    {
      "action": "create-route",
      "segment": "Shared",
      "destination-cidr-blocks": [
        "0.0.0.0/0"
      ],
      "destinations": [
        "attachment-0123456789abcdefg"
      ]
    },
    {
      "action": "create-route",
      "segment": "Production",
      "destination-cidr-blocks": [
        "0.0.0.0/0"
      ],
      "destinations": [
        "attachment-0123456789abcdefg"
      ]
    }
  ],
  "attachment-policies": [
    {
      "rule-number": 100,
      "condition-logic": "and",
      "conditions": [
        {
          "type": "tag-value",
          "operator": "equals",
          "key": "cloud-wan-segment",
          "value": "Production"
        }
      ],
      "action": {
        "association-method": "tag",
        "tag-value-of-key": "cloud-wan-segment"
      }
    },
    {
      "rule-number": 101,
      "condition-logic": "and",
      "conditions": [
        {
          "type": "tag-value",
          "operator": "equals",
          "key": "cloud-wan-segment",
          "value": "Development"
        }
      ],
      "action": {
        "association-method": "tag",
        "tag-value-of-key": "cloud-wan-segment"
      }
    },
    {
      "rule-number": 102,
      "condition-logic": "and",
      "conditions": [
        {
          "type": "tag-value",
          "operator": "equals",
          "key": "cloud-wan-segment",
          "value": "Staging"
        }
      ],
      "action": {
        "association-method": "tag",
        "tag-value-of-key": "cloud-wan-segment"
      }
    },
    {
      "rule-number": 103,
      "condition-logic": "and",
      "conditions": [
        {
          "type": "tag-value",
          "operator": "equals",
          "key": "cloud-wan-segment",
          "value": "Shared"
        }
      ],
      "action": {
        "association-method": "tag",
        "tag-value-of-key": "cloud-wan-segment"
      }
    }
  ]
}
```
:::

</p>

4. Other than leverage the Transit Gateway Route Tables, you could also separate various business intentions via more than one Transit Gateways. When turning to the Cloud WAN architecture, ++each region could only have a Core Network Edge within a Cloud WAN core network++. As you see the above architecture diagrams, all the components reside in the Cloud WAN core network; therefore, ++the separation is taken place on the core network++ (well...please ignore Blackhole :sweat_smile:).

![core-network-separation](https://i.imgur.com/dR0HmTF.png)

### <span class="fontColorH3">Anything Else?</span>

Other than above mentioned capabilities, there are additional two differences between Transit Gateway and Cloud WAN.

- **Simpler operations** - for operators who are keen to address the following situations, Cloud WAN is what they seek for;
    - They are not familiar with managing multi-hierarchies/segments of the network transport.
    - They aim to streamline the whole Transit Gateway manipulation, e.g., segmentation across a variety of Transit Gateway Route Tables or cross-region routing exchange across a number of Transit Gateway Peerings.
    - The Network-as-Code is a preferred method for routine operation.
- **Higher CapEx** - intrinsically, the charge model and pricing are quite close or even identical between Transit Gateway and Cloud WAN except for Core Network Edge. ++Transit Gateway itself is not charged, but each Core Network Edge charges $0.5/hour.++ Imagine that you have 4 Core Network Edges across regions in your environment, that means you need to extra pay $1,440/month without any attachments and data processes when compared with Transit Gateway.

Typically, if you are familiar with the whole Transit Gateway manipulation and all the modifications have involved in your automated framework already then Cloud WAN may not really an interesting solution for you because of the cost that highly depends on how many Core Network Edges will be provisioned.

## <span class="fontColorH2">Conclusion</span>

For mitigating any unwanted request toward your service origins as close to the sources (requesters) as possible, we typically defense them on the CDN (edge) side. When looking at the cloud security on the infrastructure level, both Transit Gateway and Cloud WAN adopt the same strategy as well; the restriction/isolation happens on the segment instead of via Security Groups.

Furthermore, why Cloud WAN is more fascinating than Transit Gateway is that it simplifies the operations a lot albeit I have to admit that its overall pricing is not so tasty :expressionless:.

> Published on 6^th^ October 2022. 

:::info
###### tags: `AWS` `Architecture` `NativeCloudNet` `TransitVPC` `TransitGateway` `CloudWAN`
:::

:::warning
:repeat: Return To [Index](https://bit.ly/terrencec51229).
:::

{%hackmd BJrTq20hE %}