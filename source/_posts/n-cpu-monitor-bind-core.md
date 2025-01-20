---
title: 【学习笔记】CPU 利用率、绑核
tags:
  - C++
toc: true
languages:
  - zh-CN
categories:
  - 学习笔记
comments: false
cover: false
date: 2025-01-21 01:43:48
---

CPU 利用率，以及绑核操作示例代码。

<!-- more -->

## CPU 利用率统计

与 `top` 命令结果一致。

```cpp
inline double calculate_cpu_utilization(uint64_t &last_total_time, uint64_t &last_process_time)
{
    std::ifstream proc_stat("/proc/stat");
    std::ifstream self_stat("/proc/self/stat");

    if (!proc_stat.is_open() || !self_stat.is_open())
    {
        return -1.0; // Error opening files
    }

    proc_stat.seekg(0);
    self_stat.seekg(0);

    std::string line;
    uint64_t user, nice, system, idle, iowait, irq, softirq, steal;
    uint64_t utime, stime;

    if (std::getline(proc_stat, line))
    {
        std::istringstream iss(line);
        iss.ignore(5, ' '); // Skip "cpu"
        iss >> user >> nice >> system >> idle >> iowait >> irq >> softirq >> steal;
    }
    else
    {
        return -1.0; // Error reading /proc/stat
    }

    if (std::getline(self_stat, line))
    {
        std::istringstream iss(line);
        for (int i = 0; i < 13; ++i)
            {
                iss.ignore(std::numeric_limits<std::streamsize>::max(), ' ');
            }
        iss >> utime >> stime;
    }
    else
    {
        return -1.0; // Error reading /proc/self/stat
    }

    uint64_t total_time = user + nice + system + idle + iowait + irq + softirq + steal;
    uint64_t process_time = utime + stime;

    double utilization = 0.0;
    if (last_total_time > 0 && last_process_time > 0)
    {
        uint64_t total_delta = total_time - last_total_time;
        uint64_t process_delta = process_time - last_process_time;
        if (total_delta > 0)
        {
            int num_cores = std::thread::hardware_concurrency();
            utilization = 100.0 * process_delta / (total_delta / num_cores);
        }
    }

    last_total_time = total_time;
    last_process_time = process_time;

    return utilization;
}
```

---

## 绑核

```cpp
void BindThreadToCore(int thread_id)
{
    cpu_set_t cpu_set;
    CPU_ZERO(&cpu_set);
    // Get the core from the CPU mask by checking the bit at the thread_id position
    int core_id = GetCoreIdFromMask(thread_id);
    CPU_SET(core_id, &cpu_set);
    if (pthread_setaffinity_np(pthread_self(), sizeof(cpu_set_t), &cpu_set) != 0)
    {
        std::cerr << "Error binding thread to CPU core " << core_id << std::endl;
    }
    else
    {
        std::cout << "Server thread " << std::this_thread::get_id() << " bound to core " << core_id << std::endl;
    }
}

// Function to get the core ID from the mask, rotating through available cores
int GetCoreIdFromMask(int thread_id)
{
    int available_core_count = cpu_mask_.count();
    if (available_core_count == 0)
    {
        throw std::runtime_error("No available CPU cores in the mask");
    }
    // Find the core by checking the bits in the CPU mask
    int core_id = -1;
    int count = 0;
    for (int i = 0; i < 128; ++i)
    {
        if (cpu_mask_.test(i))
        {
            if (count == thread_id)
            {
                core_id = i;
                break;
            }
            ++count;
        }
    }
    // Not enough cores
    if (core_id == -1)
    {
        throw std::runtime_error("The allocated cpu cores are less than the number of threads");
    }
    return core_id;
}
```