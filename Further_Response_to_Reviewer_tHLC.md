## Further Response to Reviewer `tHLC`

Table1: Comparison of simple physics-guided baselines under a unified Prithvi-WxC backbone on FireCast-CA. Fixed physical layers provide modest gains, while PyroCast yields stronger improvements in both predictive accuracy and physical consistency, and the full FireTrend achieves the best overall performance.

| Method                                                       |    IoU |     F1 |  AUPRC |  PDS |
| ------------------------------------------------------------ | -----: | -----: | -----: | ---: |
| Prithvi-WxC (Foundation Model introduced by NASA)            | 0.6058 | 0.6913 | 0.8769 | 0.84 |
| Prithvi-WxC + Wind-aligned smoothing                         | 0.6116 | 0.6964 | 0.8815 | 0.86 |
| Prithvi-WxC + Fixed anisotropic diffusion layer              | 0.6130 | 0.6980 | 0.8830 | 0.87 |
| Prithvi-WxC + Fixed advection-diffusion layer                | 0.6150 | 0.7002 | 0.8852 | 0.87 |
| Prithvi-WxC + **<span style="color:red">PyroCast</span>** post-hoc | 0.6205 | 0.7042 | 0.8893 | 0.89 |
| **<span style="color:red">FireTrend</span>** (Full model)    | 0.6345 | 0.7158 | 0.9009 | 0.91 |







