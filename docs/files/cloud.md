### What are some of reasons to access a remote computer system?

*This material is from Data Carpentry, 2015.

 * Your computer does not have enough resources to run the desired analysis (memory, processors, disk space, network bandwidth).
 * You want to produce results faster than your computer can.
 * You cannot install software in your computer (application does not have support for your operating system, conflicts with other existing applications)

### Main Computational Resources

![Computer Node](https://raw.githubusercontent.com/datacarpentry/cloud-genomics/master/lessons/images/Computer.png)

   * High Performance Computing (HPC): large assemble of physical machines and a homogeneous operating system (e.g., your institutions' HPC, XSEDE's HPC)
   * Cloud Computing: virtual machines, distributed platforms and/or applications offered as a service (e.g., Amazon Web Services, Microsoft Azure, Google Cloud Computing)

 * Virtual machine (VM): software computer that like a physical computer can run an operating system and applications
 * Operating system (OS): the basic software layer that allows execution and management of applications
 * Physical machine: the hardware (processors, memory, disk and network)

### HPC vs. Cloud:

| HPC | Cloud |
|:----|:------|
| User account on the system | root account on the system |
| Limited control of the system | Full control of the system |
| Central shared file system | Local file system |
| Jobs submitted into a queue | Jobs executed on each resource |
| Account-based isolation | OS-based isolation |
| Batch-oriented execution of applications | Support for batch or interactive applications |
| Request for resource and time allocation | Pay-per-use |

![HPC vs. Cloud](https://raw.githubusercontent.com/datacarpentry/cloud-genomics/master/lessons/images/HpcVsCloud.png)


### Resources:

 * Cloud computing offerings:
  * Amazon EC2: http://aws.amazon.com/ec2/
  * Microsoft Azure: https://azure.microsoft.com/en-us/
  * Google Cloud Platform: https://cloud.google.com/
  * iPlant's Atmosphere: http://www.iplantcollaborative.org/ci/atmosphere
  * iPlant's help page: https://pods.iplantcollaborative.org/wiki/display/atmman/Atmosphere+Manual+Table+of+Contents
 * HPC offerings:
  * XSEDE: https://www.xsede.org/high-performance-computing
