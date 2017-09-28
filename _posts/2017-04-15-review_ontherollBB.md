---
layout: page
title: Paper Review - On the Role of Burst Buffers in Leadership-Class Storage Systems
---
- 최근의 연구에서는 HPC Application이 Bursty한 I/O패턴 겪게된다고 함.
- Checkpointing을 위해 60TiB/s 스토리지 대욕역폭 필요.
- 이러한 Bursty I/O requirement는 시스템 설계자에게 I/O에 소요되는 시간을 최소화함과 동시에 system failure에 대한 대비책도 고려하게 한다.
- 현재 대규모 HPC 시스템들은 다양한 응용을 위해 설계된 Parallel FS 이용(HPC 시스템에 ad-hoc하지 않다. 따라서 HPC시스템과 PFS사이에 I/O middle-ware 레이어를 추가 구성하여, I/O를 보다 잘 핸들링하도록 한다. 이 레이어는 HPC I/O 스택 관점에서는 충분히 낮아서, 모든 응용의 I/O 액세스를 캡쳐 할 수 있고, 기존 스토리지 시스템 설계를 보완하기 위한 스택 관점에서는 충분히 높은 위치에 있다.

 
