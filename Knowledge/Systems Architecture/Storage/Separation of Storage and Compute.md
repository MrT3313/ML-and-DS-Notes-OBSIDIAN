---
note_kind: concept
aliases:
  - separation of storage and compute
  - storage-compute separation
  - disaggregated storage and compute
  - disaggregation
up: "[[Cloud-Native Architecture]]"
sources:
  - "[[DDIA Ch01 Trade-Offs in Data Systems Architecture]]"
confidence: draft
---

## Definition

Traditional systems architecture uses the same computer for both storage and computation: the disk and the CPU and RAM that read it are in one box. Separating them, or *disaggregating* them, means the bytes live in one service and the code that reads them runs in another, with a network in between.

Amazon S3 is the plainest example. It only stores files. If you want to analyze the data in it, you have to run the analysis code somewhere outside S3.

## Formal statement

Disaggregation buys durability with latency, and the exchange rate is roughly an order of magnitude and a half.

**The latency side.** A read from a disk attached to the host is a local device access. The standard table of latency numbers, in the 2019 revision that circulates under the title "These Are the Numbers Every Computer Engineer Should Know", puts an SSD random read at about 16 microseconds and a round trip inside one data center at about 500 microseconds, so reaching a disk over the network costs something like

$$\frac{500\ \mu\text{s}}{16\ \mu\text{s}} \approx 30\times$$

the local read, before the remote service has done any work of its own. Amazon's documentation lands in the same place from the other direction: io2 Block Express, the lowest-latency EBS type, is designed for an average latency under 500 microseconds on a 16 KiB operation.

**The durability side.** That factor of thirty is what buys a number a local disk does not have. AWS documents gp3 and io1 volumes at 99.8% to 99.9% durability, an annual failure rate of 0.1% to 0.2%, and io2 Block Express at 99.999%, an annual failure rate of 0.001%. A local disk carries no annual figure at all, because its life is not measured in years: it ends when the instance ends.

**RAID is redundancy without separation.** RAID (Redundant Array of Independent Disks) maintains copies of the data on several disks attached to the same machine. It can be implemented in hardware or in software by the operating system, and either way it is transparent to the application reading the filesystem, which sees one disk. A disk can fail and the data survives, but nothing has moved off the machine: lose the machine and you lose access to every copy at once. That is the contrast that makes disaggregation visible, and it is why redundancy and separation have to be counted as independent properties.

**Local disk against virtual disk** is where the separation actually starts. Both look like a block device to the operating system, and they behave differently at exactly the moment you care about.

| | local disk | virtual disk |
|---|---|---|
| what it is | a disk physically attached to the host machine your instance runs on | not a physical disk at all, but a cloud service provided by a separate set of machines that emulates the behavior of one |
| survives the instance | no. Cloud systems treat local disks as ephemeral caches rather than long-term storage | yes. It exists independently of the running life of any instance |
| moves between instances | no. It cannot be detached from one instance and attached to another | yes, that is the point of it |
| survives a resize | no. Replacing the instance with a bigger or smaller one loses the contents | yes. Detach, launch the new instance, reattach |
| redundancy | whatever the single disk gives you | the service replicates the data across multiple machines on its own |

Amazon's two products are the reference case: an EC2 instance store is documented as disks physically attached to the host, whose data persists across a reboot but not across a stop, a hibernate, a termination or a change of instance type, and which cannot be detached and reattached elsewhere. An EBS volume persists independently of the instance, attaches and detaches, and is automatically replicated across multiple servers. The separation is real but not unlimited: an EBS volume and the instance it attaches to must be in the same Availability Zone, so the disk outlives the machine without outliving the datacenter.

A virtual disk separates storage from compute at the level of blocks. [[Object Storage]] separates them at the level of whole files, and gives up more of the filesystem interface in return for hiding the machines entirely.

## Where it is used

[[Cloud-Native Architecture]] has this as its defining structural property, which is why a cloud-native database can grow its storage and its query capacity on separate schedules while a self-hosted one grows both by buying a bigger machine. [[Node]] is what the separation survives: a virtual disk outlives the instance it was attached to, so losing a node stops being the same event as losing data. [[Object Storage]] is the most extreme version, storage that offers no computation at all. [[Online Analytical Processing]] is the workload that benefits most, since a query that scans an enormous amount of data wants compute sized to the query rather than to the data, and that is only possible once the two are not the same machine.
