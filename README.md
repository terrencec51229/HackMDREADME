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

# Pure Accelerate 2019 Re-cap

:::success
:bulb: *The following outlines do not cover ALL the events. I was capturing some of them that were worth to be kept in my mind instead.* 
:::

[TOC]

## <span class="fontColorH2">FlashArray</span>

### <span class="fontColorH3">DirectMemory</span>
The main functionality of DirectMemory is to accelerate the READ performance. Based on current design of FlashArray, both <span class="fontColor">X70</span> and <span class="fontColor">X90</span> are the only supported models.

Intrinsically, DirectMemory is for the caching purpose so that it does not protect by RAID. If one of the DirectMemory caches malfunctions during reading, it would affect the performance only (higher latency than usual).

### <span class="fontColorH3">Others</span>
- FlashArray supports NVMe-oF and it requires an additional 25/50Gb NIC module (MPO connector).
- SNAP2NFS supports NFSv3 now.
- FlashArray supports both the X-X/X-C replications.
  - For X-series to X-series: <span class="fontColor">1 X-platform is able to afford up to 32 replications.</span> For instance, one X90 platform could afford up to 32x X20 platform replications.
  - For X-series to C-series: <span class="fontColor">1 C-platform is able to afford up to 8 replications.</span> For instance, one C60 platform could afford up to 8x X20 platform replications.

## <span class="fontColorH2">Cloud Block Store</span>

### <span class="fontColorH3">Architecture</span>

The controllers are launched on c5n.9xlarge (10GbE) by default and it is able to upgrade to c5n.18xlarge (25GbE) if needed. Currently, those two are the only available instance types. The vShelfs (virtual drives) are launched on i3.2xlarge (1.9TB NVMe) by default. Refer to [Cloud Block Store Support Matrix](https://support.purestorage.com/FlashArray/PurityFA/Cloud_Block_Store/Cloud_Block_Store_Support_Matrix) for more details.

![CBS Architecture](https://i.imgur.com/VyYjLdZ.png)

Although both FlashArray and Cloud Block Store (CBS) adopt the same software and architecture design, however, from the overall performance perspective, CBS meets about 70% level of FlashArrary.

Based on current design, the vShelf cannot be NDU (non-disruptive upgrade). Each controller's specification (c5n.9xlarge/18xlarge) has its own vShelf instance-type and they cannot mix.

- When the controller is c5n.9xlarge then its vShelf would be 1x i3.2xlarge (1x 1.9 NVMe SSD).
- When the controller is c5n.18xlarge then its vShelf would be 4x i3.8xlarge (4x 1.9 NVMe SSD).

### <span class="fontColorH3">Procurement and Deployment</span>

The primary difference between Pure-As-a-Service and AWS Marketplace has three...

- Pure-As-a-Service is a compound package that composes of on-premises/cloud arrays.
- The contract of Pure-As-a-Service is directly signed to Pure.
- Pure-As-a-Service has not released for the Asia market yet.

![Procurement Options](https://i.imgur.com/ZCexF1Y.png)

### <span class="fontColorH3">Supported Regions</span>

![Supported Regions](https://i.imgur.com/8XcfIUB.png =x500)

## <span class="fontColorH2">References</span>
> [YouTube: TFD - Pure Storage Cloud Block Store Architecture](https://www.youtube.com/watch?v=5RBv-EG-NC4)

> [YouTube: TFD - Pure Storage Cloud Block Store Procurement Deployment, Manageability](https://www.youtube.com/watch?v=lL3ec0slyeo)

:::info
###### tags: `PureStorage` `PureAccelerate` `CBS`
:::

{%hackmd BJrTq20hE %}