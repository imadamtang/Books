# Nand Flash 结构



## 背景

Nand Flash是一种非易失性存储器，生活中常见的u盘、ssd盘、sd卡等器件都有包含。

> The two main types of flash memory, NOR flash and NAND flash, are named for the NOR and NAND logic gates. Both use the same cell design, consisting of floating-gate MOSFETs. **They differ at the circuit level depending on whether the state of the bit line or word lines is pulled high or low:** in NAND flash, the relationship between the bit line and the word lines resembles a NAND gate; in NOR flash, it resembles a NOR gate.

根据维基百科关于的介绍，我们可以看到Nand Flash是Flash Memory中的一种，其中Nand为NOT AND（与非）缩写，另一种是Nor Flash，Nor为NOT OR（或非）缩写。之所以这么叫，是因为这两种Flash架构与逻辑电路中的与非门、或非门类似，这个地方做一个初步了解即可。

<img src="https://s21.ax1x.com/2025/01/05/pE9AuwT.png" width=30%/>



## 浮栅MOS管

Nand Flash为什么能存储数据，这就牵涉到一个叫MOS管的东西。认识MOS管之前，先了解下什么是PN结。

PN结是由N型材料和P型材料组合而成的。N型材料包含较多电子，带负电；P型材料包含较多空穴，带正电。当两种材料结合时，N型材料中的电子天然地跑向P型材料，在PN材料结合处，会形成一个过渡层（也叫耗尽层），这就叫PN结。

<img src="https://s21.ax1x.com/2025/01/05/pE9EIUO.png" width=40%/>

由于PN结中电子从N向P扩散，导致靠N区一侧失去电子带正电，靠P区一侧获得电子带负电，形成N区指向P区的电场，即PN结内电场。也正是由于这电场的存在，N区电子无法继续扩散，使得PN结整体呈电中性。而当N区一侧接负极，P区一侧接正极时，外部正向电压形成的电场方向与PN结内电场方向相反，使得内电场被削弱，从而使得电子能够从N区顺利扩散到P区，PN结导通。反之，内电场扩大，电子更加无法扩散，PN无法导通。（反向电压足够大时，可能造成击穿而急剧漏电）。

总之，结论就是：**PN材料结合，形成一个单向导通的特性，即电子只能从N型材料流向P型材料。**



ok，后来有一个聪明人，利用PN结的特性，设计了下图的MOS管结构。其中，黑色为金属级，用于导电，蓝色为N型材料，绿色为P型材料。

<img src="https://s21.ax1x.com/2025/01/05/pE9VqFU.png" width=50%/>

当给源极和漏极接通电源时，由于P型材料含有较少电子，源极和漏极之间不能形成导电沟道，电路无法导通。



<img src="https://s21.ax1x.com/2025/01/05/pE9VLYF.png" width=50%/>

当给栅极接通正极时，栅极到P型材料之间形成正电压，栅极下是二氧化硅，是绝缘的。所以P型材料中的电子会在电场力的作用下，聚集到绝缘层下面。当聚集的电子数足够多时，便在源极和漏极之间形成导电沟道，使得源极和漏极可以导通。

所以，MOS管的特性总结下来就是：**给栅极高电平，源极和漏极就能导通；给低电平，源极和漏极就无法导通。**



在MOS管中的绝缘层，增加一个可以导电的浮栅层，就形成了浮栅MOS管，如下图所示。

<img src="https://s21.ax1x.com/2025/01/05/pE9ZFYD.png" width=50%/>

给栅极接高电压，P型材料的衬底接低电压，电子会聚集在绝缘层附近，但由于绝缘层中间增加了浮栅层，绝缘层非常薄，电子在高压的作用下通过隧穿效应，穿过绝缘层，进入浮栅层。这就是Nand写入的过程。当栅极电压消失后，由于绝缘层的存在，电子就被锁在浮栅层中（也就得到了非易失性的存储特性）。反之，秤底增加高压，栅极增加低压，电子又会隧穿回P型材料，浮栅层内电子流失，这就是擦除的过程。

所以，浮栅MOS管，就是通过：**通过浮栅层中是否有电子，来表示0或者1，从而储存信息。**

> 浮栅层中是否有电子，表示0或1，这就是Nand中的SLC。同时，当电子足够多时，我们还可以按照有电子的多少，来表示更多bit的信息。MLC就是2bit，TLC就是3bit，QLC就是4bit。每n个bit需要 $2^{n}$ 个电子数量来表示。



## Nand结构

### 2D Nand

把浮栅MOS管按照下图方式组合起来，形成的Nand就是2D Nand。

<img src="https://s21.ax1x.com/2025/01/05/pE9ZNmq.png" width=70%/>

其中，横着共用栅极的，叫做Word Line（字线）；竖着源极和漏极串联起来的是Bit Line（位线）。

<img src="https://s21.ax1x.com/2025/01/05/pE9Zyc9.png"/>

其中，为了成本考虑，某一块所有字线和位线的浮栅MOS管是共用衬底的，所以擦除是以block为单位进行的擦除。

> 在一个位线中，当需要给其中的某一个浮栅MOS管做操作时，由于所有浮栅MOS管的源极和漏极是串联起来的，所以需要将位线中的其他浮栅MOS管置于导通状态，所以其他浮栅MOS管所处的字线需要加一个 $V_{pass}$ 导通电压。



### 3D Nand

2D Nand为了增加单位体积内的容量，只能不断缩小浮栅MOS管的体积，但随着体积的缩小，带来了很多问题。比如氧化层变薄后，隧穿次数稍微多一些，氧化层就容易失效，导致无法锁住电子，存储单元就失效了。所以，后面人们就发明了3D Nand 结构，通过堆叠的方式，能更加有效地增加单位体积内的存储容量。

如下图，就是3D Nand结构示意图。

<img src="https://s21.ax1x.com/2025/01/05/pE9Z4hD.png" width=50%/>

