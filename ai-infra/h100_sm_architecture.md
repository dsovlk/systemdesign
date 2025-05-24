# NVIDIA H100 Streaming Multiprocessor Architecture

This diagram illustrates the internal architecture of a single Streaming Multiprocessor (SM) in the NVIDIA H100 GPU, showing the horizontal data flow and parallel processing pipeline.

```mermaid
flowchart LR
    subgraph "H100 Streaming Multiprocessor (SM)"
        direction LR
        
        %% Left Layer - Control
        subgraph "Control Layer"
            direction TB
            WS1[Warp Scheduler 1]
            WS2[Warp Scheduler 2]
            WS3[Warp Scheduler 3]
            WS4[Warp Scheduler 4]
        end

        %% Middle Layer - Execution
        subgraph "Execution Layer"
            direction TB
            subgraph "General Purpose"
                direction TB
                CC1["CUDA Core Block (4x Total)"]
            end
            
            subgraph "Specialized"
                direction TB
                TC1["Tensor Core Block (4x Total)"]
                SFU1[Special Function Unit 1]
                SFU2[Special Function Unit 2]
            end
        end

        %% Right Layer - Memory
        subgraph "Memory Layer"
            direction TB
            subgraph "Fast Memory"
                direction TB
                RF["Register File (256KB)"]
                L1["L1 Data Cache (192KB)"]
            end
            
            subgraph "Shared Resources"
                direction TB
                SM["Shared Memory (256KB)"]
                LSU[Load/Store Unit]
            end
        end

        %% Control to Execution Connections
        WS1 --> CC1 & TC1 & SFU1 & LSU
        WS2 --> CC1 & TC1 & SFU2 & LSU
        WS3 --> CC1 & TC1 & LSU
        WS4 --> CC1 & TC1 & LSU

        %% Execution to Memory Connections
        CC1 --> RF
        TC1 --> RF
        SFU1 & SFU2 --> RF
        LSU --> RF

        %% Memory Hierarchy Connections
        RF --> L1
        L1 --> SM
    end

    %% External Memory
    subgraph "GPU Memory Hierarchy"
        direction TB
        HBM["HBM Memory (80GB, 3TB/s)"]
        SDRAM["SDRAM (2TB, 200GB/s)"]
    end

    %% Connect SM to External Memory
    SM -.->|Memory Controller| HBM
    HBM -.->|PCIe| SDRAM

    %% Style Definitions
    classDef gpu fill:#76b900,stroke:#333,stroke-width:2px,color:#fff,font-weight:bold
    classDef cpu fill:#ff9900,stroke:#333,stroke-width:2px,color:#fff,font-weight:bold
    classDef memory fill:#66ccff,stroke:#333,stroke-width:2px,color:#000,font-weight:bold
    classDef cache fill:#ff9966,stroke:#333,stroke-width:2px,color:#000,font-weight:bold
    classDef storage fill:#663399,stroke:#333,stroke-width:2px,color:#fff,font-weight:bold
    classDef external fill:#0066cc,stroke:#333,stroke-width:2px,color:#fff,font-weight:bold

    %% Apply styles
    class WS1,WS2,WS3,WS4 cpu
    class CC1,TC1,SFU1,SFU2,LSU gpu
    class RF,SM memory
    class L1 cache
    class HBM,SDRAM external
```

## Architecture Overview

### Control Layer
- **Warp Schedulers (4x)**
  - Orchestrates thread execution
  - Issues instructions to execution units
  - Manages thread scheduling and switching
  - All schedulers have access to Load/Store Unit for memory operations

### Execution Layer
- **General Purpose Computing**
  - CUDA Core Blocks (4x)
  - Integer and floating-point operations
  - Basic arithmetic and logic operations
  - Each block contains multiple CUDA cores for parallel processing
  - Optimized for general-purpose computing tasks
  - Supports FP32, FP64, and integer operations

- **Specialized Computing**
  - Tensor Core Blocks (4x)
    - Specialized for matrix operations
    - Optimized for AI and deep learning workloads
    - Supports mixed-precision operations (FP16, BF16, FP8)
    - Performs matrix multiply-accumulate operations
    - 4x faster than FP32 for matrix operations
    - Essential for transformer models and neural networks

  - Special Function Units (2x)
    - Handle transcendental functions
    - Process special mathematical operations
    - Support for:
      - Trigonometric functions (sin, cos, tan)
      - Exponential and logarithmic functions
      - Power functions
      - Other complex mathematical operations
    - Optimized for scientific computing

### Memory Layer
- **Fast Memory**
  - Register File (256KB)
    - Fastest memory access
    - Dedicated to each thread
    - Used for frequently accessed data
    - Zero latency access

  - L1 Data Cache (192KB)
    - Shared between L1 cache and shared memory
    - Configurable split between cache and shared memory
    - Reduces memory access latency
    - Caches frequently used data

- **Shared Resources**
  - Shared Memory (256KB)
    - Low-latency memory shared between threads
    - Configurable with L1 cache
    - Used for thread communication
    - Enables efficient data sharing

  - Load/Store Unit
    - Manages memory operations
    - Handles data movement between memory levels
    - Coordinates memory access patterns
    - Optimizes memory bandwidth usage

### External Memory
- **HBM Memory**
  - 80GB capacity
  - 3TB/s bandwidth
  - High Bandwidth Memory for GPU
  - Connected via Memory Controller
  - Optimized for high-throughput data access

- **System Memory (SDRAM)**
  - CPU's main system memory
  - Typically 256GB-1TB capacity
  - Bandwidth: 100-200 GB/s (DDR4/DDR5)
  - Connected via PCIe to GPU
  - Used for CPU-GPU communication
  - Larger capacity but lower bandwidth than HBM
  - Stores data not actively being processed by GPU
  - Acts as a bridge between CPU and GPU memory systems

## Data Flow
1. Warp Schedulers dispatch instructions to execution units
2. Execution units process data using registers
3. Memory hierarchy manages data movement
4. Load/Store Unit handles memory operations for all warps
5. Shared Memory enables thread communication
6. Data flows between SM and HBM via Memory Controller
7. HBM communicates with System Memory via PCIe

## Performance Characteristics
- Up to 2048 concurrent threads per SM
- 64 thread groups (warps) of 32 threads each
- Warp switching in single clock cycle
- Efficient latency hiding through massive parallelism
- High throughput for parallel workloads
- 3TB/s memory bandwidth with HBM
- Optimized for both general-purpose and specialized computing
- Excellent performance for AI and scientific workloads 