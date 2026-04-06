# advanced_business_analytics
Group 1 Project in DSBA Advanced Business Analytics 6211

#General Google Colab information
As Students we have access to 2 tiers, the free tier and the Pro .edu tier
Python Runtime versions are only 2025.10 and 2025.07

Free Tier:
  Free tier offers us a the T4 GPU and v5e-1 TPU hardware acceleration options. 
  Colab could limit/deny accelerators depending on demand or recent usage due to limit fluctuations which could terminate
  runtimes. Notebooks will timeout at the 12 hour mark or if idle.
  T4 GPU works well with Pytorch CUDA workflows: (Tesla T4 Express 16GB HBM2 256 GB/s) 
  v5e-1 TPU works well with tensorflow: (197 TFLOPs, 16GB with 819 GB/s) 
  
Colab Pro.edu Tier:
  100 Compute units per month. Compute units expire after 90 days. (which means up to 300 Compute Units in 3 Months)
  Access to more GPU and RAM 
  H100 - 18.05 CU/hour
  A100 - 5.37 -> 11.77 CU/hour
  L4 - 3 -> 4.82 CU/hour
  TPU v2-8 -> 1.76 CU/hour

  
https://archive.ics.uci.edu/datasets
https://fred.stlouisfed.org/
https://datahub.io/collections
https://www.earthdata.nasa.gov/data
https://github.com/awesomedata/awesome-public-datasets
https://datasetsearch.research.google.com/

https://nasa-power.s3.us-west-2.amazonaws.com/index.html
https://power.larc.nasa.gov/dashboard/


**top 10 sku by count**
{0: '3/4_SCH_40_90_DEG_ELBOW_SXS_35_PK',
 1: '3/4_SIDE_OUTLET_90_DEGREE_SOCKET',
 2: '3/4_SCH_40_TEE_SXSXS_(20_PACK)',
 3: '3/4_PVC_COUPLING_DEEP_SOCKET_SXS',
 4: '11/4X2FT_PVC_SCH_40_PIPE',
 5: '3/4_SCH_40_COUPLING_SXS_(25_PACK)',
 6: '1_PVC_SIDE_OUTLET90_DEG_ELBOW_SXSXS',
 7: '3/4_PVC_RUNNING_TRAP_SPGXSPG',
 8: '1/2X2_FLOWGUARD_GOLD_CPVC',
 9: '11/2X2_PVC_MALE_ADAPTER_MPTXS'}