---
title: "My Proxmox Odyssey"
date: 2025-11-03
draft: false
---

## From QEMU Quirks to Virtualized Nirvana

For years, I've been a devotee of virtualization, bouncing between bare metal, QEMU, and everything in between. Recent success with Proxmox has truly unlocked a level of performance and convenience I hadn’t thought possible. This post details my journey to building a robust, virtualized environment centered around Proxmox, and the lessons I learned along the way.

### The QEMU Struggle: A Foundation of Pain

My initial foray into GPU passthrough started with QEMU. After months of troubleshooting, compiling kernels from source, and battling secure boot, I achieved near-native benchmarks (and in some benchmarks, I even _out-performed_ bare metal!), but stability proved elusive. The GPU would bind and unbind, but at some point a full system reboot was often required. While I achieved respectable performance, it felt like magic, not a repeatable solution.

Even though there was plenty of pain, I learned _so much_ about the inner workings of `qemu`, driver blacklisting, and virtualization in general. Compiling from source became almost a default for me, as I loved being able to configure exactly how programs were compiled, and ensuring I was getting the best performance for my specific hardware.

### Proxmox: The Game Changer

I knew of Proxmox for a long while before actually implementing it on my own hardware. Once I finally installed it, I saw that it provided a more elegant solution, and the brilliant devs that make Proxmox happen somehow found a way to allow for seamless binding/unbinding of the GPU.

That was the part that I got so close to achieving, but never quite got there with `qemu`. I can now seamlessly switch between a Windows 10 gaming VM and a Debian LLM server, and I have not _once_ run into an issue where I needed to do a full system reboot to regain access to the GPU.

### The Setup

My current Proxmox setup consists of:

| Component                                                   | Key Features                                                                                                                                                                                                                                                                                                                                                                       |
| ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Host**<br>Debian 12<br>_Proxmox Virtualization Host_      | <ul><li>i5-12600K</li><ul><li>Intel IOMMU passthrough</li><li>[SR-IOV](https://en.wikipedia.org/wiki/Single-root_input/output_virtualization)</li></ul><li>2x 16GB DDR5 RAM</li><li>Nvidia 3090FE (Founders Edition)</li><ul><li>nouveau and nvidia drivers blacklisted on boot, for full VM passthrough capability</li></ul><li>750W Platinum+ PSU</li><li>1TB NVMe SSD</li></ul> |
| **VM1 (Gaming)**<br>Windows 10<br>_High-Performance Gaming_ | <ul><li>12 i5 cores (the P-cores) with `host` passthrough</li><li>28GB RAM</li><li>Full PCIe NVMe SSD controller passthrough</li><li>Full VFIO Passthrough of 3090FE</li><li>[smbios settings](https://forum.proxmox.com/threads/windows-11-vm-for-gaming-setup-guide.137718/) to avoid issues with anti-cheat kernel implementations</li></ul>                                    |
| **VM2 (LLM)**<br>Debian 12<br>_LLM Serving & Inference_     | <ul><li>Debian 12 (for best current compatibility with TensorRT builds)</li><li>12 i5 cores (the P-cores) with `host` passthrough</li><li>28GB RAM passed through (w/1GB [hugepages](https://www.netdata.cloud/blog/understanding-huge-pages/))</li><li>Full VFIO Passthrough of 3090FE</li><li>28GB RAM</li><li>dedicated to hosting, serving, and optimizing LLMs.</li></ul>     |

_Although, this does not include the Proxmox cluster I've set up, of which this is only one node. More on that in the future!_

### Key Benefits

- **It's all so easy!:** The 3090FE seamlessly attaches/detaches between the VMs, enabling both high-performance gaming and efficient LLM inference. Sure, I _could_ compile `qemu` from source again and try to make this happen, but Proxmox does all that for me!
- **Performance:** solid 240 FPS in Rocket League and 45 t/s with Gemma 3B 27B with a 131K context window.
- **Reliability:** After years of troubleshooting and compiling `qemu` from source, this setup is _stable_.

One of my main goals was to not sacrifice _anything_ in the setup of the LLM, so I could get all those benefits that were bragged about in the [initial release announcement](https://developers.googleblog.com/en/introducing-gemma3/). I remember reading about "largest model to fit in a single consumer GPU!!" and "enormous 131K context window!" - but when I first set it up on `ollama`, it would crash nearly instantly with a 131K context window.

- I want the full 27B model on my gpu
- I want the _full_ 131,072 token context window
- **AND** I want incredible performance, with GPU vRAM to spare

That led to my relentless pursuit of optimized perfection, building `llama.cpp` from source and reading the `man` pages (well, the `llama-server --help`) to ensure I was running with all the optimal settings. This also taught me about all the different architectural aspects of these models, leading me to make more informed decisions about which models I wanted to incorporate into my workflow.

### Future Plans

- Explore TensorRT optimization for the LLM VM.
- Automate the deployment with a more robust configuration management system.
- Incorporate a custom vector database for enhanced [RAG](https://en.wikipedia.org/wiki/Retrieval-augmented_generation) performance.

---
