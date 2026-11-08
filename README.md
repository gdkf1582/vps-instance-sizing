# Dedicated Server for VPS Hosting: How Many Instances Fit on One Box, How Many IPs You Need, and What to Confirm Before You Order (Sizing Math + Checklist)

A dedicated server bought for VPS hosting is sized by four numbers: usable CPU cores, RAM left after the hypervisor takes its share, IPv4 addresses, and monthly transfer. Get those right and picking a plan takes five minutes. Get the IP count wrong and you end up with a machine that happily runs eight virtual machines but can only put three of them online.

What follows is the arithmetic for turning one physical box into a set of VPS, the point where oversubscription starts showing up as slow instances, and the checks worth running on the order page before you pay. SpinServers is the provider used as the reference point; plan lists and prices change, so verify current figures there.

> **Quick pick:** size for the instance count you expect in six months, and leave 25–30% of cores and RAM unallocated. For 5–10 small VPS that usually means one mid-range node with a /28 of IPv4 rather than the cheapest box listed. 👉 [See SpinServers' current dedicated server plans and prices](https://bit.ly/Spinservers)

## The four limits, in the order they usually bind

**RAM binds first.** You can hand out more vCPU than you own and stay stable, because most VPS sit idle most of the time. You cannot do the same with RAM without swapping, and swap on a node hosting eight tenants is a outage waiting for a busy hour. Subtract 1–2 GB for the host OS and the hypervisor before you divide.

**IPv4 binds second.** Every VPS that needs SSH, mail, or its own outbound reputation needs its own address. This is the number that most often forces a bigger plan or a support ticket, and it is the number least likely to be printed in a comparison table.

**Cores bind third.** Only when you decide to keep a 1:1 vCPU-to-core ratio, which is the right call for game servers, CI runners, and anything with sustained CPU load.

**Bandwidth binds last, and rarely.** Ten small VPS serving web pages will not saturate a gigabit port. Ten VPS running backups, media, or torrents will.

Disk sits outside this list because it fails differently. Two drives in RAID1 give you redundancy, not throughput: eight instances doing database writes on one mirror will queue, and no amount of CPU headroom fixes queue depth.

## The slicing math: from one box to N instances

Start with a profile per VPS. These are the shapes most small hosts sell:

| Profile | vCPU | RAM | Disk | Typical use |
| --- | --- | --- | --- | --- |
| Nano | 1 | 1 GB | 20 GB | DNS, monitoring, a single static site |
| Small | 1 | 2 GB | 40 GB | Small site, VPN, bot |
| Standard | 2 | 4 GB | 60 GB | A few sites, small app, mail |
| Large | 4 | 8 GB | 120 GB | Database, game server, build agent |

Now take a node with 8 cores and 32 GB. Reserve 1 core and 2 GB for the host: 7 cores and 30 GB remain.

RAM allows seven Standard instances (7 × 4 GB = 28 GB, with 2 GB spare). Those seven instances want 14 vCPU, which is a 2:1 overcommit on 7 cores. That is fine for web hosting and wrong for anything CPU-bound. If you want 1:1 instead, the same box carries three Standard instances with a core left over, and the RAM is half empty. The choice between those two outcomes is the whole design decision, and it is made before you order, not after.

Disk is the check people skip. Seven Standard instances need 420 GB of usable space plus the host. On a two-drive mirror that means 480 GB drives at minimum, and the IOPS are shared by everyone.

## What oversubscription actually costs: CPU steal time

When a hypervisor has more vCPU promised than physical cores available, it sometimes pauses one virtual machine so another can run. The paused machine's guest OS sees that wait as **CPU steal time**, and the customer experiences it as a VPS that is slow for no visible reason. Liquid Web's engineers describe the mechanism this way and treat high steal time on a parent server as a signal to move instances off it.

The practical rule for a host selling VPS: keep total sold vCPU at or below roughly 2× physical cores, and keep at least one core's worth of headroom on a busy node. Beyond 3:1 you are relying on all your tenants being idle at the same time, which holds until one of them gets a traffic spike. If you are the only tenant on your own box, this matters less, but it still matters if you plan to sell.

## IP addresses: the constraint that decides your plan

Count what you actually need before you look at prices.

- One IPv4 per VPS instance.
- One for the host node, so you can reach the hypervisor when a guest is broken.
- Two if you run your own nameservers, because registrars require two distinct IPs for NS records.
- One spare for a control panel or a migration.

Eight VPS plus a host plus two nameservers is eleven addresses. A /29 gives you six usable, a /28 gives you fourteen. That difference is the reason to ask for the larger block at order time: adding IPv4 later usually means a justification ticket to the regional registry, and some providers charge per extra address or refuse small blocks to new accounts.

Two things worth confirming in writing before you buy: how many IPv4 come with the plan at no extra cost, and what an additional block costs per month. Also ask whether an IPv6 block is included and how large it is. IPv6 does not replace IPv4 for customer-facing VPS (many tenants still need a v4 for outbound mail and legacy clients), but it is useful for internal addressing and for anything you sell as dual-stack.

👉 [Check what SpinServers includes per plan: IPv4 count, bandwidth, disk](https://bit.ly/Spinservers)

## Bandwidth: "unmetered" is a port speed, not a volume

A 1 Gbps port running flat out moves about 324 TB in 30 days. No provider sells that as a monthly allowance; when a plan says unmetered, it means you are limited by the port rather than by a counter, and the port is shared with other customers on the same switch.

Do the estimate anyway, because it tells you whether you need a gigabit port at all. Ten VPS at 500 GB each is 5 TB a month, which is nothing. Ten VPS running nightly off-site backups of 200 GB each is 60 TB a month, which is a different conversation and probably a second NIC or a second node.

Ask two questions: is the port 1 Gbps or 10 Gbps, and what happens if you sustain a high percentage of it for hours. Providers that throttle or null-route abusive tenants will say so; the ones that do not will tell you it is fine, which is also information.

## Per-host billing versus per-instance billing

There are two ways to pay for virtual machines on hardware you control, and confusing them makes price comparison meaningless.

The first is per-instance: every VPS gets a line item. The second is per-host. Atlantic.Net's dedicated host product is a clear example of the second model as it was documented in March 2023: you pay for the physical server, and launching instances on it, of any size, in any number, adds nothing to the bill. Windows Server licensing was the stated exception, because Microsoft requires licence affinity to the host. Under that model, the marginal cost of the tenth VPS is zero and the only question is whether the hardware fits.

Most dedicated server providers, including the one you are looking at, sell you a box and leave the virtualization to you, which is effectively per-host billing with you as the operator. That changes the comparison you should be running. Instead of asking "is this dedicated server cheaper than a VPS", divide the monthly host price by the number of instances you can reliably run on it, then compare that per-instance figure against what a comparable VPS costs elsewhere. A node at 7 Standard instances has a per-instance cost of one-seventh of the box. If that number is above the retail price of an equivalent VPS, you are buying control and isolation, not savings, and you should be honest with yourself about whether you need either.

## Pre-order checklist

| Item | What to ask | Why it matters |
| --- | --- | --- |
| Virtualization | Can I install KVM, Proxmox VE, or Xen? Is nested virtualization enabled? | Some providers block custom kernels or KVM modules |
| Console access | Is IPMI/KVM-over-IP included, and can I boot my own ISO? | Without it, a broken boot means a support ticket and a wait |
| IPv4 | How many included, cost per extra, block size available at order | Usually the binding constraint on instance count |
| Bandwidth | Port speed, monthly volume, overage rate | Overage bills are the classic surprise |
| Disk | Drive model, count, RAID controller, hot-swap bays | Decides IOPS and whether you can add drives later |
| Provisioning | Stated delivery time, and whether it is guaranteed | Ranges vary widely; GoDaddy's own hosting FAQ quotes "minutes, but up to 24 hours or more" |
| Contract | Minimum term, setup fee, refund window | Monthly with no setup fee is worth a premium when you are starting |
| Upgrades | Can RAM or disk be added without a reinstall or migration | Determines whether the box can grow with your customer count |

## Who should not buy a dedicated box for this

If you are running one website, a mail server, and a VPN, a managed VPS at a few dollars a month is the correct answer. You get someone else's patching, someone else's monitoring, and no hypervisor to maintain.

The break-even is not really about money. Running VPS on your own hardware means you own the boring parts: kernel updates on the host, backup verification, capacity monitoring, abuse complaints when a tenant spams, and the 3 a.m. restart. Three instances do not justify that. Ten paying customers do, and so does a workload that needs kernel-level control or hardware isolation you cannot get on shared infrastructure.

## FAQ

**Do I need to buy virtualization software?**
KVM and Proxmox VE carry no per-VM licence fee. Windows guests and commercial control panels do, and those are priced per instance, so a Windows VPS costs more to run than a Linux one on the same node. Budget for the guest licences, not the hypervisor.

**How long does delivery take?**
It varies enough that you should ask before paying if you have a launch date. Some providers provision in minutes automatically, others hand-build and quote up to a day or more. Confirm whether a setup fee applies, since that is often paired with manual provisioning.

**Can I add IPs after the server is live?**
Usually, but frequently with a justification requirement and a per-address monthly charge. Getting the block size right at order time is cheaper and faster than a ticket in month two.

**Can I upgrade RAM or disk later?**
On a rented dedicated server this often means a migration rather than a hot swap. Ask whether the provider can add drives to the existing chassis, and whether a RAM upgrade requires a new machine.

## How to decide

If you need three or fewer instances and no kernel control, buy a VPS and skip this entirely. If you need 5–10 small instances with room to grow, size on RAM first, confirm a /28 of IPv4, and accept a 2:1 vCPU overcommit only if your tenants are web-facing rather than CPU-bound. Above ten instances, plan for a second node rather than pushing one box to 3:1.

Two things to settle before you pay: the exact number of usable IPv4 addresses included, and what bandwidth overage costs. Everything else you can fix after delivery.

👉 [Compare SpinServers' current dedicated server configurations](https://bit.ly/Spinservers)
