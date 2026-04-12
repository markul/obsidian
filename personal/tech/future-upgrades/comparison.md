# Hardware Comparison

| Item | optiplex.markul.net | alfa.markul.net | Current MacBook | Mac Studio (M4 Max 128 GB) | Custom Build (7950X) |
| --- | --- | --- | --- | --- | --- |
| Type | Physical machine | VM on Proxmox | Physical laptop | Physical desktop | Physical desktop |
| CPU | Intel Core<br>i5-13500T | 10 vCPU<br>`x86-64-v2-AES` | Apple M1 Pro | Apple M4 Max | AMD Ryzen 9<br>7950X |
| CPU topology | 20 threads<br>14 cores<br>1 socket | 10 vCPU<br>10 cores<br>1 socket | 8 CPU cores | 12 CPU cores<br>(8P + 4E) | 32 threads<br>16 cores<br>1 socket |
| RAM | 64 GB<br>physical | 16192 MiB max<br>balloon min 8192 MiB | 16 GB<br>unified memory | 128 GB<br>unified memory | 128 GB<br>physical |
| RAM type / speed | 2 x 32 GB<br>DDR4-3200 SODIMM<br>non-ECC<br>`JM3200HSE-32G` | QEMU virtual RAM<br>ECC-capable guest exposure<br>speed not exposed | LPDDR5 unified<br>Apple: 200 GB/s bandwidth | Unified memory<br>Apple: 546 GB/s bandwidth | User target:<br>128 GB RAM<br>speed TBD |
| Storage | 1 TB NVMe<br>1 TB SSD | 200 GB<br>virtual disk | 512 GB SSD | 1 TB SSD | 2 TB NVMe |
| Price<br>(Mar 29, 2026) | ~$500-650<br>used market estimate | N/A<br>VM on existing host | ~$750-900<br>used market estimate | ~$3,700 new<br>configured estimate | ~$1,700-2,200<br>estimated, no dGPU |
| Notes | Bare metal<br>highest local headroom | Backed by<br>`proxmox.markul.net` | `MacBookPro18,3` | M4 Max 16-core CPU<br>40-core GPU config | Proposed high-headroom<br>Docker / K8s dev box |
