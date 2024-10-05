# Timing Analysis

## Synchronous Design

- t~cq~ – clock to output: essentially a propagation delay 
- t~setup~ – setup time: the time the data needs to arrive before the clock 
- t~hold~ – hold time: the time the data has to be stable after the clock
- 

### t~cq~

**t~cq~就是时钟信号的上升沿到Q的上升沿或者下降沿的时间。**

![image-20241005103244611](https://gitee.com/yzhangkq/images/raw/master/202410051034543.png)







### t~setup~

**t~setup~就是D在上升时钟信号之前D必须保持的时间。**

![image-20241005103455303](https://gitee.com/yzhangkq/images/raw/master/202410051058433.png)







### t~hold~

**t~hold~就是上升时钟信号之后D必须保持的时间。**

![](https://gitee.com/yzhangkq/images/raw/master/202410051039245.png)





### Timing Constraints

- **Max Delay: The data doesn’t have enough time to pass from one register to the next before the next clock edge.**
-  **Min Delay: The data path is so short that it passes through several registers during the same clock cycle.**

Max delay也叫做“setup” path，是因为他是由于slow data path引起的，所以一般都是不符合setup的要求。

Min delay也叫做“hold” path，是因为他是由于是fast data path引起的，所以一般是不符合hold的要求。





### Setup (Max) Constraint

![image-20241005111156332](https://gitee.com/yzhangkq/images/raw/master/202410051112050.png)

看这张图就很清楚，其实max delay就是看两个clk之间的信号，例如在这里，第一个clock上升的信号就是叫做launch clock，过了一个一个clock period即使capture clock edge。

然后其中的时间看上图就是需要满足以下的式子：


![image-20241005111935101](https://gitee.com/yzhangkq/images/raw/master/202410051120558.png)





### Hold (Min) Constraint

![image-20241005112141333](https://gitee.com/yzhangkq/images/raw/master/202410051123750.png)

看这张图就很清楚，其实min delay就是要满足cq的时间加上logic上运行的时间需要比hold的时间多。因为如果少的话，那其实就是小于hold的时间了，这样子B的数据就不对了。

然后其中的时间看上图就是需要满足以下的式子：



![image-20241005112155963](https://gitee.com/yzhangkq/images/raw/master/202410051123005.png)



### Summary

![image-20241005113917367](https://gitee.com/yzhangkq/images/raw/master/202410051139992.png)



## Static Timing Analysis (STA)

### 主要特点

1. **时序路径分析**：
   - **启动时钟**和**捕获时钟**：确定信号从一个触发器出发到另一个触发器的路径。
   - **建立时间和保持时间检查**：确保信号在捕获时钟的建立时间和保持时间要求内到达。
2. **不依赖于输入矢量**：
   - STA通过分析所有可能的路径，不需要具体的输入测试向量。
3. **时钟约束**：
   - 分析时钟的偏移、抖动、占空比等特性，确保时钟网络的稳定性。
4. **工艺变化和温度影响**：
   - 考虑工艺变化、温度和电压对电路时序的影响。

### 优势

- **速度快**：相比动态时序分析，STA不需要仿真所有可能的输入组合，因此更快速。
- **覆盖全面**：能够检测到所有可能的最坏情况路径。

### 限制

- 不会电路的functionality。
- 不会考虑异步电路



### Node Oriented Timing Analysis

在静态时序分析（STA）中，节点导向时序分析（Node Oriented Timing Analysis）主要关注每个节点的时序特性。以下是其基本步骤：

### 1. 节点定义
- **识别电路中所有节点**：包括输入、输出、寄存器、逻辑门等。

### 2. 时钟传播
- **计算时钟到达时间**：从时钟源开始，沿时钟路径传播到每个节点。

### 3. 数据传播
- **计算数据到达时间**：从数据源节点出发，沿数据路径传播到目标节点。
  
### 4. 建立和保持时间检查
- **建立时间检查**：确保数据在捕获节点的建立时间窗口内到达。
- **保持时间检查**：确保数据在保持时间内稳定。

### 5. 计算裕量
- **裕量计算**：通过比较建立和保持时间与数据到达时间，确定时序裕量。

### 6. 时序报告

- **生成节点时序报告**：列出每个节点的时序信息，包括时钟和数据到达时间、裕量等。

### 7. 迭代优化
- **调整设计**：根据节点分析结果进行优化，如调整逻辑或添加缓冲器。

通过这种方法，可以细致地分析每个节点的时序特性，确保整个电路在设计要求下正确工作。





## Design Constraints

**How does the STA tool know what the required clock period is?**

我们需要告诉他，而这告诉的方式就是写在Synopsys Design Constraints（SDC）中。

SDC语法是TCL的子集。

- EDA tools sometimes use a different data structure called a “collection”

- A collection is similar to a TCL list, but: 
  - The value of a collection is not a string, but rather a pointer, and we need to use special functions to access its values. 
  - For example, if you were to run foreach on a collection, it would just have one element (the pointer to the collection). Instead, use foreach_in_collection.



### Design Objects

- Design: A circuit description that performs one or more logical functions (i.e Verilog module). 
- Cell: An instantiation of a design within another design (i.e Verilog instance).
  - Called an inst in Stylus Common UI. 
- Reference: The original design that a cell "points to" (i.e Verilog sub-module) 
  - Called a module in Stylus Common UI. 
- Port: The input, output or inout port of a Design. 
- Pin: The input, output or inout pin of a Cell in the Design. 
- Net: The wire that connects Ports to Pins and/or Pins to each other. 
- Clock: Port of a Design or Pin of a Cell explicitly defined as a clock source. 
  - Called a clock_tree in Stylus Common UI.

![image-20241005122354858](https://gitee.com/yzhangkq/images/raw/master/202410051224692.png)





### SDC实例

当然，这里是一个简单的SDC文件示例，展示如何定义一些基本的时序约束：

```sdc
# 定义主时钟，频率为100MHz，占空比50%
create_clock -name clk -period 10 [get_ports clk]

# 设置输入端口的输入延迟
set_input_delay 2.5 -clock clk [get_ports data_in]

# 设置输出端口的输出延迟
set_output_delay 2.5 -clock clk [get_ports data_out]

# 定义一个假路径
set_false_path -from [get_ports reset] -to [get_ports data_out]

# 定义一个多周期路径
set_multicycle_path 2 -from [get_registers reg1] -to [get_registers reg2]

# 设置负载
set_load 0.5 [get_ports data_out]

# 设置驱动单元
set_driving_cell -lib_cell INV_X1 [get_ports data_in]
```

### 解释

- **`create_clock`**：定义时钟信号`clk`，周期为10ns。
- **`set_input_delay`**：为`data_in`端口设置相对于`clk`的输入延迟。
- **`set_output_delay`**：为`data_out`端口设置相对于`clk`的输出延迟。
- **`set_false_path`**：指定从`reset`到`data_out`的路径为假路径。
- **`set_multicycle_path`**：指定从`reg1`到`reg2`的路径为多周期路径。
- **`set_load`**：为`data_out`端口设置负载。
- **`set_driving_cell`**：为`data_in`端口设置驱动单元。

这些约束帮助EDA工具进行时序分析和优化设计。



## Timing Report

### 在时序报告中，通常包含以下内容：

#### 1. **路径信息**

   - **起点和终点**：显示时序路径的起始和结束节点。
   - **路径类型**：如寄存器到寄存器、输入到寄存器等。

#### 2. **时序约束**

   - **时钟周期**：路径对应的时钟周期信息。
   - **建立和保持时间**：路径的建立时间和保持时间要求。

#### 3. **延迟分析**

   - **组合逻辑延迟**：路径中每个逻辑单元的延迟。
   - **互连延迟**：信号在线路上的延迟。

#### 4. **时序裕量**

   - **建立裕量**：建立时间要求减去实际到达时间。
   - **保持裕量**：实际到达时间减去保持时间要求。

#### 5. **时序例外**

   - **假路径**：标记为假路径的部分。
   - **多周期路径**：指定为多周期路径的部分。

#### 6. **其他信息**

   - **时钟抖动**：时钟信号的不确定性。
   - **数据到达和要求时间**：数据到达和所需的时间。

这些信息帮助设计者识别潜在的时序问题并进行优化。



### 实例

当然，这里是一个典型的时序报告示例：

```
Path: Register to Register
----------------------------------------------------------------
Startpoint: reg1 (rising edge-triggered flip-flop)
Endpoint: reg2 (rising edge-triggered flip-flop)
Path Group: clk
Path Type: Max

Clock: clk (rise edge)
  Arrival Time: 0.00
  Required Time: 10.00
  Data Arrival Time: 9.00
  Data Required Time: 9.50
  Slack: 0.50

+-----------------+---------------+----------------+--------+
| Instance        | Cell          | Type           | Delay  |
+-----------------+---------------+----------------+--------+
| reg1/Q          |               | Launch Clock   | 0.00   |
| logic1/Z        | AND2_X1       | Cell Delay     | 0.85   |
| net1            |               | Net Delay      | 0.25   |
| logic2/Z        | OR2_X1        | Cell Delay     | 0.90   |
| net2            |               | Net Delay      | 0.30   |
| reg2/D          |               | Arrival        | 9.00   |
+-----------------+---------------+----------------+--------+

Clock Uncertainty: 0.50
Setup Time: 0.50
```

### 解释

- **起点和终点**：路径从`reg1`到`reg2`。
- **时钟信息**：使用时钟`clk`，周期10ns。
- **延迟分析**：
  - **逻辑单元和互连延迟**：列出路径中每个单元及其延迟。
- **时序裕量**：
  - **Slack**：0.50ns，表示满足时序要求的余量。
- **不确定性**：
  - **时钟不确定性**：0.50ns，包括抖动和偏斜。

这个报告帮助设计者查看路径延迟和裕量，以确保设计满足时序要求。





## Multi-Mode Multi-Corner

Multi-Mode Multi-Corner（MMMC）分析用于在不同的工作模式和工艺条件下验证数字电路设计的时序和功耗。



### 具体步骤

进行Multi-Mode Multi-Corner（MMMC）分析的具体步骤如下：

#### 1. 定义模式和角

- **多模式定义**：
  - 确定设计需要支持的不同功能模式（如正常、测试、低功耗模式）。
  
- **多角定义**：
  - 确定分析的工艺角、温度和电压条件（如典型、快、慢角）。

#### 2. 设置约束

- **为每种模式创建SDC文件**：
  - 定义时钟、I/O延迟、时序例外等。
  
- **针对不同角设置库文件**：
  - 加载对应的工艺库和延迟模型。

#### 3. 分析准备

- **工具配置**：
  - 配置EDA工具以支持多模式和多角分析。

#### 4. 执行STA

- **运行时序分析**：
  - 在每个模式和角下执行静态时序分析。
  
- **检查路径**：
  - 分析关键路径，确保所有路径满足时序要求。

#### 5. 结果收集和优化

- **收集时序报告**：
  - 获取每个模式和角下的时序报告。
  
- **识别问题路径**：
  - 找出不满足时序要求的路径，进行优化。

#### 6. 迭代改进

- **设计调整**：
  - 根据分析结果调整设计，如修改电路或约束。
  
- **重新分析**：
  - 重复时序分析，验证调整效果。

通过这些步骤，确保设计在所有工作条件下的功能性和可靠性。