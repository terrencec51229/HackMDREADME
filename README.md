:::info
###### tags: `PureStorage` `PureAccelerate` `CBS`
:::

<style>
.fontColor {
  color: #FF5733;
}
.fontColorH2 {
  color: #FF960F
}
.fontColorH3{
  color: #FFB432
}
.fontColorH4{
  color: #2E86C1
}
.fontFace {
  font-weight: Bold;
  font-style: Italic;
}
</style>

[TOC]

# Pure Accelerate 2019 Re-cap
The following outlines do not cover ALL the events. I was capturing some of the them that were worth to be kept in mind instead. 

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

Although both FlashArray and Cloud Block Store (CBS) adopt the same software and architecture design, however, from the overall performance perspective, CBS meets about 70% level of FlashArrary.

Based on current design, the vShelf cannot be NDU (non-disruptive upgrade). Each controller's specification (c5n.9xlarge/18xlarge) has its own vShelf instance-type and they cannot mix.

- When the controller is c5n.9xlarge then its vShelf would be 1x i3.2xlarge (1x 1.9 NVMe SSD).
- When the controller is c5n.18xlarge then its vShelf would be 4x i3.8xlarge (4x 1.9 NVMe SSD).

Refer to [Cloud Block Store Architecture](https://hackmd.io/@terrencec51229/cloud-block-store-architecture) for more details.