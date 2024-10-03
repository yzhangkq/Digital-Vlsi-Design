# Logic Synthesis

[TOC]



## Intro

### 逻辑综合是什么？

> Synthesis is the process that converts RTL into a technology-specific gate-level netlist, optimized for **a set of pre-defined constraints.**
>
> 逻辑综合其实就是一个把RTL转化成网表，而这个过程是有预先定义好的contraints的。

You start with: 

- A behavioral RTL design 
- A standard cell library 
- A set of design constraints

 You finish with: 

- A gate-level netlist, mapped to the standard cell library
- (For FPGAs: LUTs, flip-flops, and RAM blocks) 
- Hopefully, it’s also efficient in terms of speed, area, power, etc.

<img src="https://gitee.com/yzhangkq/images/raw/master/202410031942056.png" alt="image-20241003194245909" style="zoom: 50%;" />

上面这幅图就说的很清楚，其实就是把我们写好的RTL通过综合就产生了网表。



### 逻辑综合的目标



- 最小化面积 - 字面数量、单元数量、寄存器数量等。 - 最小化功耗 - 从单个门、停用电路块等开关活动的角度考虑。 - 性能最大化 - 同步系统的最大时钟频率、异步系统的吞吐量 - 以上各项的任意组合 - 结合不同的权重 - 表述为一个约束问题 - "时钟速度大于 300MHz 时面积最小" - 更全面的目标 - 布局反馈 - 实际物理尺寸、延迟、布局和布线

- **最小化面积**

  - In terms of literal count, cell count, register count, etc

    在这里literal指的是：

    （**逻辑表达式中的字面量**：在布尔逻辑中，字面量（literal）是指变量及其否定。例如，对于变量 A，A 和 ¬A 都是字面量。

    **电路复杂度**：literal count 可以用来衡量电路的复杂度，较高的字面量数量可能意味着更复杂的电路设计。

    **优化目标**：在电路优化过程中，设计师可能会试图减少字面量的数量，以降低功耗、提高速度或减少面积。）

- **最小化功耗**
  - In terms of switching activity in individual gates, deactivated circuit blocks, etc.
- **性能最大化**
  - In terms of maximal clock frequency of synchronous systems, throughput for asynchronous systems

- **更全面的目标**
  - Feedback from layout 
  - Actual physical sizes, delays, placement and routing

### 基本流程

![image-20241003201521040](https://gitee.com/yzhangkq/images/raw/master/202410032030754.png)

#### • Syntax Analysis:

- 这一步主要是读HDL文件，然后检查是不是有语法错误，即syntax errors
  - `read_hdl –verilog sourceCode/toplevel.v`

#### • Library Definition

- 提供标准单元库和IP库
  - `read_libs “/design/data/my_fab/digital/lib/TT1V25C.lib”`

#### • Elaboration and Binding

- 把RTL转化成布尔结构
- State reduction, encoding, register inferring. 
- 将所有的leaf cell和提供的库bind起来
  - `elaborate toplevel`

#### • Constraint Definition

- Define clock frequency and other design constraints.
  - `read_sdc sdc/constraints.sdc`

#### • Pre-mapping Optimization

- 映射 generic cells（逻辑单元） and perform additional heuristics.在逻辑电路映射到物理实现之前进行的优化过程。这一阶段的目标是提高电路的性能、降低功耗和减少芯片面积。

- **逻辑优化**：

  - 简化布尔表达式，减少逻辑门的数量和类型。
  - 合并逻辑功能，消除冗余的逻辑。

- **资源分配**：

  - 在逻辑层面上优化资源的使用，确保在映射过程中能够高效利用可用的逻辑单元。

- **延迟优化**：

  - 通过调整逻辑门的连接方式，减少信号传播延迟，从而提高整体电路的速度。

- **功耗优化**：

  - 通过选择低功耗的逻辑实现方式，减少动态和静态功耗。

- **面积优化**：

  - 尽量减少所需的硅面积，以降低成本和提高集成度

  - `syn_generic`

#### • Technology mapping

- Map generic logic to technology libraries
  - `syn_map`

#### • Post-mapping Optimization

- Iterate over design, changing gate sizes, Boolean literals, and architectural approaches to try and meet constraints.在逻辑综合和技术映射之后，对电路进行进一步优化的过程。具体而言，这个阶段主要包括以下几个方面：
- **调整门大小**：根据电路的性能要求，调整逻辑门的大小，以优化延迟和功耗。
- **布线优化**：改进信号之间的连接方式，以减少延迟和功耗，同时避免信号干扰。
- **逻辑重组**：重新安排逻辑门的连接，以提高电路的整体性能，尤其是在时序方面。
- **反馈迭代**：利用来自布局和布线的反馈信息，进一步优化电路。通过实际的布局数据，调整设计以满足物理约束。
- **功耗优化**：通过减少动态和静态功耗的策略（例如，通过关闭不活跃的电路路径）来优化功耗。
- **时序优化**：确保电路在所需的时钟频率下正常工作，可能需要通过增加缓冲区或重新安排信号路径来实现
  - `syn_opt`

#### • Report and export

- Report final results with an emphasis on timing reports.
  - `report timing –num paths 10 > reports/timing_reports.rpt`
- Export netlist and other results for further use. 
  - `write_hdl > export/netlist.v`

------



## Compilation

### compile和综合的区别：

#### Synthesis vs. Compilation: 

• Compiler 

​	• Recognizes all possible constructs in a formally defined program language 

​	• Translates them to a machine language representation of execution process 

• Synthesis 

​	• Recognizes a target dependent subset of a hardware description language 

​	• Maps to collection of concrete hardware resources

​	• Iterative tool in the design flow

**总而言之，其实compile就是把编程语言转换成机器语言，而synthesis就是把硬件描述语言转化成硬件源**

**在做语法检查的时候，其实是要先将HDL语言转换成高级软件语言，例如C或者Python，然后在这个level做checking：**

- To compile your Verilog code for syntax checking, use the NC-Verilog tool: 
  - `ncvlog <filename.v>`

-  This will quickly run compilation on your Verilog source code and point you to syntax errors. 

-  Alternatively, use the irun super command:
  - `irun -compile  <filename.v>`

------



## Library Definition

**库定义阶段告诉综合器在哪里寻找用于结合的leaf 单元和用于技术映射的目标库**



### 什么是库？

####  a standard cell library is delivered with a collection of files that provide all the information needed by the various EDA tools.（标准单元库中包含一系列文件，可提供各种 EDA 工具所需的全部信息。）



### 库里有哪些单元?

**• Combinational logic cells (NAND, NOR, INV, etc.):** 

-  Variety of drive strengths for all cells. 
- Complex cells (AOI【AND-OR INVERT 】, OAI, etc.) 
-  Fan-In <= 4
-  ECO Cells 

**• Buffers/Inverters**

- **功能**：缓冲器用于增强信号，提供更强的驱动能力。它能够将输入信号的幅度提高，以便驱动后续电路。
- 用途
  - 提高信号的驱动能力，以减少信号衰减。
  - 解决布线延迟问题，确保信号能够及时到达目标单元。
  - 在需要时提供隔离，避免相邻电路间的干扰。

-  Larger variety of drive strengths.
-  **“Clock cells” with balanced rise and fall delays.** 
- Delay cells 
-  Level Shifters 

**• Sequential Cells:**

-  Many types of flip flops: pos/negedge, set/reset, Q/QB, enable 
- Latches 
- Integrated Clock Gating cells 
- Scan enabled cells for ATPG.

 **• Physical Cells:** 

- Fillers, Tap cells, Antennas, DeCaps, EndCaps, Tie Cells



### Multiple Drive Strengths and VTs（多种驱动能力和threshold电压）

在标准单元库中，**Multiple Drive Strengths and VTs**（多驱动强度和多阈值）是指每个逻辑单元可以提供的不同输出驱动能力和阈值电压选项。这两者结合为设计师提供了更多的灵活性，以便在性能、功耗和面积之间进行优化。

1. **Multiple Drive Strengths（多驱动强度）**：
   - 每个逻辑单元可以有多个版本，通常标记为 X1、X2、X3 等，表示不同的驱动能力。
   - 较强的驱动能力（如 X3）适用于需要驱动较大负载的情况，而较弱的驱动能力（如 X1）则适合较小负载，通常功耗更低。

2. **Multiple Thresholds (VTs)（多阈值电压）**：
   - 每个单元可能具有不同的阈值电压（如 SVT、HVT、LVT），这影响了晶体管的开关特性。
   - **SVT**（标准阈值）适用于平衡性能和功耗，**HVT**（高阈值）适用于低功耗，但速度较慢，**LVT**（低阈值）则提供更快的开关速度，但功耗较高。

##### 优势：

- **灵活性**：设计师可以根据特定的应用需求选择合适的驱动强度和阈值电压，以优化性能和功耗。
- **性能优化**：在高频设计中，可以选择低阈值电压和高驱动强度的单元以提高速度；在低功耗应用中，则可以选择高阈值电压和低驱动强度的单元。
- **面积效率**：通过选择合适的驱动强度和阈值，设计师可以有效利用芯片面积，同时满足性能需求。

结合多驱动强度和多阈值电压，设计师能够在不同的设计约束下，灵活地调整电路特性，满足现代集成电路设计的复杂要求。



### Level Shifter

**Level Shifters**（电平转换器）在标准单元库中是一种特殊的逻辑单元，用于在不同电压域之间转换信号电平。它们在现代集成电路设计中非常重要，尤其是在多电压系统中。以下是关于电平转换器的详细信息：

##### 功能

- **电平转换**：电平转换器能够将输入信号从一个电压域（如高电压域）转换为另一个电压域（如低电压域），确保信号在不同电压环境中正确传输。
- **兼容性**：使得不同工作电压的电路模块能够互相通信，保证系统的正常工作。

##### 类型

1. **高到低电平转换（HL Shifter）**：
   - 将高电压信号（如 VDDH）转换为低电压信号（如 VDDL）。
   - 通常只需一个电源电压。
2. **低到高电平转换（LH Shifter）**：
   - 将低电压信号（如 VDDL）转换为高电压信号（如 VDDH）。
   - 通常需要两个不同的电源电压。

##### 应用场景

- **多电压域设计**：在芯片设计中，某些模块可能需要在较高电压下运行，而其他模块则使用较低电压。电平转换器确保这些模块能够有效地通信。
- **接口电路**：例如，与外部设备或传感器的接口，可能需要电平转换器来匹配不同的电压标准。

##### 设计考虑

- **延迟和功耗**：电平转换器的设计需要考虑转换延迟和功耗，以确保不会影响整个系统的性能。
- **电气特性**：电平转换器必须具备适当的输入和输出特性，以确保信号的完整性。

### Filler and Tap Cells

在标准单元库中，**Filler Cells** 和 **Tap Cells** 是两种特殊的单元，它们在集成电路设计中扮演着重要的角色，主要用于满足设计规则和确保电路的稳定性。以下是它们的具体功能和用途：

### Filler Cells（填充单元）

- **功能**：填充单元用于填补在布局中留下的空隙，以满足密度规则和制造工艺要求。这些空隙可能是由于标准单元之间的间隔或布局优化所留下的。
  
- **用途**：
  - **确保密度**：填充单元帮助维持芯片上材料的均匀分布，防止在制造过程中出现缺陷。
  - **提高工艺良率**：通过填补空隙，减少生产中的缺陷率，提高良率。

### Tap Cells（接地单元）

- **功能**：接地单元用于确保电路中适当的接地连接，防止出现潜在的寄生效应或锁存现象。

- **用途**：
  - **消除锁存现象**：通过将适当的接地连接到晶体管的衬底，防止因电压变化而引起的锁存现象。
  - **降低噪声**：在多电压或多层设计中，接地单元有助于降低噪声和电磁干扰。
  - **电源分布**：在大面积芯片中，确保电源和接地的有效分布，以提供稳定的电源。

#### 设计考虑

- **布局规则**：填充单元和接地单元的布局必须遵循特定的设计规则，以确保它们能够在制造过程中正常工作。
- **电气特性**：这些单元的电气性能（如功耗、延迟等）虽然不是主要考虑因素，但仍需确保不会对电路的整体性能产生不利影响。

#### 总结

Filler Cells 和 Tap Cells 在标准单元库中的作用是确保芯片设计的完整性和可靠性，帮助满足制造工艺要求，并提高电路的整体性能和良率。

------



### Engineering Change Order (ECO) Cells

在标准单元库中，**Engineering Change Order (ECO) Cells** 是用于处理设计变更的专用单元。这些单元的主要功能和用途如下：

#### 功能

1. **快速修改设计**：ECO Cells 允许在不重新进行整个设计流程的情况下，对已完成的设计进行小的修改。这种灵活性对于快速响应设计变更要求至关重要。

2. **局部更新**：可以在布局和布线完成后，针对特定功能或性能的需求进行小范围的调整，以修复错误或优化设计。

#### 用途

- **设计修正**：在芯片设计的后期阶段，可能会发现错误或性能问题，ECO Cells 可以用来快速修复这些问题，而无需重新设计整个电路。

- **增加功能**：在某些情况下，设计师可能需要添加小的功能变更，这些变更可以通过添加或替换 ECO Cells 来实现。

- **降低风险**：通过允许小范围的修改，ECO Cells 有助于降低重新设计的风险，避免可能的时间和成本浪费。

#### 设计考虑

- **集成**：ECO Cells 通常设计为能够无缝集成到现有布局中，确保不会干扰其他单元的功能。

- **电气特性**：ECO Cells 必须满足与原设计相同的电气特性，以确保在修改后电路的功能和性能保持一致。

#### 总结

ECO Cells 在标准单元库中的作用是为设计师提供一种灵活的方式，以便在设计的后期阶段进行必要的修改和优化，从而提高设计的效率和响应能力。它们帮助设计团队快速适应变化，确保产品的及时交付。



### ABSTRACTION(抽象化)

在逻辑综合和电路设计中，抽象化是一个重要的设计原则。通过简化所需的信息，设计师和工具可以更高效地工作，无需关注每一个细节。这有助于加速设计过程，并使得设计工具能够更好地协同工作。

- 通过抽象化，设计工具可以仅获取它们真正需要的数据。这种方法使得设计流程更简洁，计算更快速，减少了不必要的信息处理。
- 每个工具只关注其功能所需的信息，从而提高效率，降低复杂性。

**因此我们可以只提取库文件有用的信息，将他转换成lef！**

------



## Library Exchange Format (LEF)

-  Abstract description of the layout for P&R 

  -  Readable ASCII Format. 

  - Contains detailed PIN information for connecting. （**含有详细的pin的信息**）

  - Does not include front-end of the-line (poly, diffusion, etc.) data.（**不含有前端的线，例如poly，diffusion等**）

- **Abstract views only contain the following:** 
  - **Outline of the cell (size and shape)** 
  - **Pin locations and layers (usually on M1)** 
  - **Metal blockages (Areas in a cell where metal of a certain layer is being used, but is not a pin)**

下面的图很好的说明了LEF文件中到底存了哪些信息：

存了 PHYSICAL CELL SIZE;PIN; OBS;SITE;

![image-20241003213101397](https://gitee.com/yzhangkq/images/raw/master/202410032131434.png)

### **obstruction**（障碍物

**obstruction**（障碍物）是指不允许放置标准单元或其他电路元素的区域。这些区域通常是由于物理设计需求或制造工艺的限制而设置的。以下是关于 obstruction 的一些关键点：

#### 主要功能

1. **布局约束**：
   - Obstruction 用于定义在芯片布局中哪些区域不能放置单元。这对于确保电路的功能和制造工艺的完整性至关重要。
2. **保护重要区域**：
   - 在某些情况下，特定区域可能包含关键的电源或接地网络，或是其他敏感元件，设置 obstruction 可以避免在这些区域放置单元，防止潜在的干扰。
3. **优化布线**：
   - 通过定义不允许放置的区域，设计师可以更好地控制布线，从而提高布线的效率和质量。

#### 使用示例

- 定义方式

  ：

  - 在 LEF 文件中，obstruction 通常以多边形的形式定义，指定这些区域的几何形状和位置。

- 布线工具的参考

  ：

  - 布线工具在进行布局时，会参考这些 obstruction，以确保布局满足设计规则。

#### 总结

在 LEF 文件中，obstruction 是一种重要的设计元素，用于定义不允许放置标准单元的区域，从而帮助设计师控制布局和布线，确保电路的功能和性能符合要求。



### LEF文件code信息

![image-20241003213941412](https://gitee.com/yzhangkq/images/raw/master/202410032140344.png)

**• Technology LEF Files contain (simplified) information about the technology for use by the placer and router**: 

- Layers （版图层）
  - Name, such as M1, M2, etc.
  - Layer type, such as routing, cut (via)
  - Electrical properties (R, C) 
  -  Design Rules • Antenna data 
  - Preferred routing direction 
- SITE (x and y grid of the library) 
  - CORE sites are minimum standard cell size 
  - Can have the site for double-height cells! 
  - IOs have special SITE. 
- Via definitions 
- Units 
- Grids for layout and routing





### SITE

在 **LEF** 文件中，**SITE** 指的是库中标准单元的放置位置和布局信息。具体而言，SITE 定义了单元在芯片上的放置网格，包括其尺寸、对齐方式和其他相关特性。以下是关于 SITE 的一些关键点：

#### 主要功能

1. **放置网格**：
   - SITE 定义了标准单元在芯片上的放置位置网格，通常以 X 和 Y 方向的单位（如微米）表示。这有助于确保单元在布局时遵循一定的规则和对齐方式。
2. **单元尺寸**：
   - SITE 规定了标准单元的尺寸，包括宽度和高度。这些尺寸信息对于设计工具在布局时放置单元至关重要。
3. **对齐和间距**：
   - SITE 可能还会包含关于单元对齐和间距的规则，以确保在放置单元时不会违反设计规则。
4. **类型**：
   - LEF 文件中的 SITE 可能会定义不同类型的 SITE，例如用于特定功能的 SITE（如核心区域、边缘区域等），以便设计工具在布局时能够选择合适的 SITE。

#### 示例结构

在 LEF 文件中，SITE 的定义通常如下所示：

```
SITE my_site
  SIZE 1.0 BY 1.0 ;  # 单元的尺寸
  CLASS CORE ;        # SITE 类型
END my_site
```

#### 总结

在 LEF 文件中，SITE 是一个重要的定义元素，描述了标准单元的放置网格、尺寸和对齐规则。它为设计工具提供了布局标准，确保在集成电路设计过程中，单元的放置符合设计要求和制造工艺。



### Technology LEF

![image-20241003220907034](https://gitee.com/yzhangkq/images/raw/master/202410032209097.png)

![image-20241003221011561](https://gitee.com/yzhangkq/images/raw/master/202410032212589.png)

上图是一个8-track的cell，这个track是怎么看的呢，其实是这样A Track is one M1 pitch。这个cell horizontally可以容纳8条M1金属线。

**The more tracks, the wider the transistors, and the faster the cells.** 

- 7-8 low-track libraries for area efficiency 
- 11-12 tall-track libraries for performance, but have high leakage 
- 9-10 standard-track libraries for a reasonable area-performance tradeoff

------

## Liberty Timing Models (.lib)

**Liberty Timing Models**（.lib 文件）是用于描述标准单元库电气特性和时序的信息文件。它们在逻辑综合、时序分析和物理设计中起着至关重要的作用。以下是关于 .lib 文件的一些关键点：

### 主要功能

1. **时序特性**：
   - .lib 文件中包含每个逻辑单元的传播延迟（propagation delay）、上升时间（rise time）、下降时间（fall time）等时序参数。这些信息用于时序分析，确保电路在预定时钟频率下正常工作。
2. **功耗模型**：
   - 描述了单元在不同条件下的功耗特性，包括静态功耗和动态功耗。设计师可以利用这些信息来优化功耗。
3. **输入/输出特性**：
   - 包含每个引脚（pin）的电气特性，例如输入和输出阻抗、负载能力等。这些特性影响信号的完整性和电路的性能。
4. **多阈值电压和多驱动强度**：
   - 描述不同类型的阈值电压（高阈值、标准阈值、低阈值）和驱动强度的单元，允许设计师根据需求选择合适的单元。

### 文件结构示例

一个典型的 Liberty 文件结构如下：

```
library (example_lib) {
    cell (AND2) {
        pin (A) {
            direction : input;
            capacitance : 0.1;
        }
        pin (B) {
            direction : input;
            capacitance : 0.1;
        }
        pin (Y) {
            direction : output;
            capacitance : 0.2;
            timing () {
                (A, B) -> (Y) {
                    delay : (1.0, 1.5);
                }
            }
        }
    }
}
```

### 应用场景

- **逻辑综合**：在逻辑综合过程中，工具使用 .lib 文件中的信息来选择合适的单元，以满足设计的性能和功耗要求。
- **时序分析**：后端工具（如 STA，静态时序分析工具）使用这些时序参数来验证电路在工作频率下的时序是否满足约束。
- **功耗优化**：设计师利用这些功耗模型来进行电源管理和优化。

### 总结

Liberty Timing Models (.lib 文件) 是 VLSI 后端设计中不可或缺的部分，它们提供了关于标准单元的时序、功耗和电气特性的信息，帮助设计师和工具在逻辑综合和时序分析等过程中做出决策。