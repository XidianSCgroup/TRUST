# TRUST: A Toolkit for TEE-Assisted Secure Outsourced Computation over Integers

### Abstract
Secure outsourced computation (SOC) provides secure computing services by taking advantage of the computation power of cloud computing and the technology of privacy computing (e.g., homomorphic encryption). Expanding computational operations on encrypted data (e.g., enabling complex calculations directly over ciphertexts) and broadening the applicability of SOC across diverse use cases remain critical yet challenging research topics in the field. Nevertheless, previous SOC solutions frequently lack the computational efficiency and adaptability required to fully meet evolving demands. To this end, in this paper, we propose a toolkit for TEE-assisted (Trusted Execution Environment) SOC over integers, named TRUST. In terms of system architecture, TRUST falls in a single TEE-equipped cloud server only through seamlessly integrating the computation of REE (Rich Execution Environment) and TEE. In consideration of TEE being difficult to permanently store data and being vulnerable to attacks, we introduce a (2, 2)-threshold homomorphic cryptosystem to fit the hybrid computation between REE and TEE. Additionally, we carefully design a suite of SOC protocols supporting unary, binary and ternary operations. To achieve applications, we present SEAT, secure data trading based on TRUST. Security analysis demonstrates that TRUST enables SOC, avoids collusion attacks among multiple cloud servers, and mitigates potential secret leakage risks within TEE (e.g., from side-channel attacks). Experimental evaluations indicate that TRUST outperforms the state-of-the-art and requires no alignment of data as well as any network communications. Furthermore, SEAT is as effective as the Baseline without any data protection.

### Introduction

安全外包计算领域，基于门限 Paillier 加密算法（PaililerTD, FastPaiTD），涵盖了论文 [POCF](https://ieeexplore.ieee.org/abstract/document/7500106)、[SOCI](https://ieeexplore.ieee.org/abstract/document/9908577)、[SOCI+](https://ieeexplore.ieee.org/abstract/document/10531248) 和 [TRUST](https://www.sciencedirect.com/science/article/pii/S2667325825005163) 的实现。

## :memo: 文件概览

- #### **local_version**  
  
  - `Baseline`: mini-batch 梯度下降线性回归的 C++ 实现，底层训练，明文版本
  - `SDTE_C++`: mini-batch 梯度下降线性回归基于 `SDTE` 的 C++ 实现，支持 AES-GCM 加密，密文版本 
  - `SDTE_SGX`: mini-batch 梯度下降线性回归基于 `SDTE` 的 SGX 实现，结合 AES-GCM 加密与 TEE，密文版本
  - `SEAT_C++`: mini-batch 梯度下降线性回归基于 `SEAT` 的 C++ 实现，支持 FastPai 加密，密文版本
  - `POCF_C++`: `POCF` 协议的本地 C++ 可运行版本
  - `SOCI_C++`: `SOCI` 协议的本地 C++ 可运行版本
  - `SOCI+_C++`: 包含 `SOCI+` 和 `TRUST` 协议的本地 C++ 可运行版本，相当于  `SOCI+_C++` 和`TRUST_C++` 的实现，涵盖对比方案
  
- #### **socket_version**  
  
  - `POCF_socket` :  `POCF` 协议的 socket 可运行版本，需要开多端口
  - `SOCI_socket` :  `SOCI` 协议的 socket 可运行版本，需要开多端口
  - `SOCI+_socket` : 包含 `SOCI+` 和 `TRUST` 协议的 socket 可运行版本，相当于  `SOCI+_socket` 和`TRUST_socket` 的实现，涵盖对比方案，需要开多端口

- #### **sgx_version**

  - `TRUST_SGX` : `TRUST` 的 SGX 实现
  - `SEAT_SGX` : `SEAT` 的 SGX 实现

- #### **diy_version**

  - 做任何你想做的操作（e.g., 浮点数，负数）

- #### **simple_version**

  - `TRUST_C++`: `TRUST` 协议的本地 C++ 可运行版本，无对比方案

- #### **linux_version**

  - `Final_TRUST_socket` : 主动权在`CP.cpp`

- #### **window_version**

  - `Final_TRUST_socket` : 主动权在`CP.cpp`
  - `Final_TRUST_socket_v2` : 主动权在 `Client.cpp`

  
## :pencil2: Remark

一个值得思考的现象，当我们的实验将高精度计算（如 **GMP**）与网络通信（**Socket**）结合时，在Linux上的计算性能远低于Windows。
    - 在Windows上，网络版本的计算效率几乎与纯计算版本（无网络通信）持平。
    - 在Linux上，完全相同的代码其计算性能却会显著下降。

即便我们①切换网络模式（socket长连接、socket短连接、HTTP），②更换大数库（例如，从GMP替换为NTL）,依旧有如此现象。 
我们认为，性能差异的根源并非网络协议栈本身的速度，而在于网络I/O活动对CPU密集型计算任务产生的“副作用”，这种副作用因操作系统的调度策略不同而异。

在Linux环境下，即使是轻微的网络I/O活动，也会促使操作系统调度器更频繁地中断计算线程，并将其在不同CPU核心间迁移。每一次迁移都可能导致昂贵的CPU缓存失效 (Cache Miss)，迫使数据从主内存重新加载，从而严重拖慢了GMP的高精度计算速度。相比之下，Windows的调度策略在这种混合负载下表现得更为稳定，使得计算线程免受此类干扰，因而维持了接近纯计算版本的性能。

Window `simple_version/TRUST_C++` - Windows Performance Toolkit

Window `socket_version/SOCI+_socket` - Windows Performance Toolkit

Linux `simple_version/TRUST_C++` - perf report
  - 733 context-switches                                                                        
  - 0 cpu-migrations

Linux  `socket_version/SOCI+_socket`
  - 906 context-switches                                                                                                                 
  - 68 cpu-migrations


相同代码（`Final_TRUST_socket`）在Windows/Linux的时间开销测试。

#### Window

```shell
// Final_TRUST_socket
[epoch = 80] --- m_1 = 363, m_2 = 19, m_3 = 217
[epoch = 81] --- m_1 = -613, m_2 = -297, m_3 = 254
[epoch = 82] --- m_1 = 1000, m_2 = 383, m_3 = 203
[epoch = 83] --- m_1 = -999, m_2 = -455, m_3 = 252
[epoch = 84] --- m_1 = 292, m_2 = 317, m_3 = 204
[epoch = 85] --- m_1 = 416, m_2 = -862, m_3 = 201
[epoch = 86] --- m_1 = 50, m_2 = 556, m_3 = 180
[epoch = 87] --- m_1 = -595, m_2 = 524, m_3 = 158
[epoch = 88] --- m_1 = 974, m_2 = 612, m_3 = 142
[epoch = 89] --- m_1 = -21, m_2 = 100, m_3 = 217
[epoch = 90] --- m_1 = -255, m_2 = 692, m_3 = 243
[epoch = 91] --- m_1 = -725, m_2 = 746, m_3 = 135
[epoch = 92] --- m_1 = 547, m_2 = -392, m_3 = 205
[epoch = 93] --- m_1 = 504, m_2 = -277, m_3 = 165
[epoch = 94] --- m_1 = 187, m_2 = -350, m_3 = 150
[epoch = 95] --- m_1 = -596, m_2 = 696, m_3 = 182
[epoch = 96] --- m_1 = -246, m_2 = 498, m_3 = 202
[epoch = 97] --- m_1 = -542, m_2 = -165, m_3 = 153
[epoch = 98] --- m_1 = 555, m_2 = 378, m_3 = 225
[epoch = 99] --- m_1 = 900, m_2 = 798, m_3 = 195
[epoch = 100] --- m_1 = -10, m_2 = -106, m_3 = 140
Trust_FMUL (100 average) time is  ------  14.620004 ms
Trust_FCMP (100 average) time is  ------  14.590008 ms      
Trust_FEQL (100 average) time is  ------  29.439998 ms      
Trust_FABS (100 average) time is  ------  14.040010 ms      
Trust_FTRN (100 average) time is  ------  27.969992 ms
```

#### Linux

```shell
// Final_TRUST_socket
[epoch = 80] --- m_1 = 363, m_2 = 19, m_3 = 217
[epoch = 81] --- m_1 = -613, m_2 = -297, m_3 = 254
[epoch = 82] --- m_1 = 1000, m_2 = 383, m_3 = 203
[epoch = 83] --- m_1 = -999, m_2 = -455, m_3 = 252
[epoch = 84] --- m_1 = 292, m_2 = 317, m_3 = 204
[epoch = 85] --- m_1 = 416, m_2 = -862, m_3 = 201
[epoch = 86] --- m_1 = 50, m_2 = 556, m_3 = 180
[epoch = 87] --- m_1 = -595, m_2 = 524, m_3 = 158
[epoch = 88] --- m_1 = 974, m_2 = 612, m_3 = 142
[epoch = 89] --- m_1 = -21, m_2 = 100, m_3 = 217
[epoch = 90] --- m_1 = -255, m_2 = 692, m_3 = 243
[epoch = 91] --- m_1 = -725, m_2 = 746, m_3 = 135
[epoch = 92] --- m_1 = 547, m_2 = -392, m_3 = 205
[epoch = 93] --- m_1 = 504, m_2 = -277, m_3 = 165
[epoch = 94] --- m_1 = 187, m_2 = -350, m_3 = 150
[epoch = 95] --- m_1 = -596, m_2 = 696, m_3 = 182
[epoch = 96] --- m_1 = -246, m_2 = 498, m_3 = 202
[epoch = 97] --- m_1 = -542, m_2 = -165, m_3 = 153
[epoch = 98] --- m_1 = 555, m_2 = 378, m_3 = 225
[epoch = 99] --- m_1 = 900, m_2 = 798, m_3 = 195
[epoch = 100] --- m_1 = -10, m_2 = -106, m_3 = 140
Trust_FMUL (100 average) time is  ------  25.115513 ms
Trust_FCMP (100 average) time is  ------  25.743598 ms
Trust_FEQL (100 average) time is  ------  38.334907 ms
Trust_FABS (100 average) time is  ------  24.601104 ms
Trust_FTRN (100 average) time is  ------  36.389111 ms
```



## License

如果学到了，**不要忘记点个Star** :sparkling_heart:

Copyright :copyright:2024 [Aptx4869AC](https://github.com/Aptx4869AC)
