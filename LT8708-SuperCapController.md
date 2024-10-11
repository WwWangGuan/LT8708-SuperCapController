---
title: LT8708-SuperCapController
date: 2024-9-24 15:00:00
mathjax: true
catgories: 电子电力
cover: true
tags:
  - 超级电容
  - 电子电力技术
  - 电子元件原神
  - 硬件组
  - RoboMaster
---
# Design Requirement
 电容侧电压欠压保护
 底盘侧电压过压保护
## LT8708
### Chip Diagram
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241010182741.png)
### Typical Application
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240923104201.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240923104211.png)
#### For SuperCap
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240929142259.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240929142311.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240929142323.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240929142330.png)
### Monitor Pin
| 引脚        | 功能                    | 描述                                                                                                         | 备注  |
| --------- | --------------------- | ---------------------------------------------------------------------------------------------------------- | --- |
| VOUTLOMON | $V_{OUT}$欠压监测引脚       | 在$V_{OUT}$、VOUTLOMON 和 GND 之间连接一个 ±1% 电阻分压器，以设置$V_{OUT}$上的欠压电平。当$V_{OUT}$低于该电平时，反向导通被禁用，以防止从$V_{OUT}$汲取电流。 |     |
| VINHIMON  | $V_{IN}$过压监测引脚        | 在$V_{IN}$、VINHIMON 和 GND 之间连接一个 ±1% 的电阻分压器，以设置 VIN 上的过压电平。当 VIN 高于该电平时，反向导通被禁用，以防止电流流入$V_{IN}$。            |     |
| ICP       | 正向$V_{OUT}$电流监测引脚     | 从该引脚流出的电流为 20μA 加上与正平均$V_{OUT}$电流成比例的电流。                                                                   |     |
| ICN       | 反向$V_{OUT}$电流监测引脚     | 从该引脚流出的电流为 20μA 加上与负平均$V_{OUT}$电流成比例的电流。                                                                   |     |
| IMON_INP  | 正向$V_{IN}$ 电流监视器和限制引脚 | 从该引脚流出的电流为 20μA 加上与正平均 VIN 电流成比例的电流。 $IMON\_INP$  还连接到误差放大器 EA5，可用于限制最大正$V_{IN}$电流。                        |     |
| IMON_INN  | 反向$V_{IN}$ 电流监视器和限制引脚 | 从该引脚流出的电流为 20μA 加上与负平均 VIN 电流成比例的电流。 $IMON\_INN$ 还连接到误差放大器 EA1，可用于限制最大负$V_{IN}$电流。                         |     |
| IIMON_OP  | 正 VOUT 电流监视器和限制引脚。    | 从该引脚流出的电流为 20μA 加上与正平均 VOUT 电流成比例的电流。 IMON_OP 还连接到误差放大器 EA6，可用于限制最大正 VOUT 电流。                              |     |
| IMON_ON   | 负 VOUT 电流监视器和限制引脚     | 从该引脚流出的电流为 20μA 加上与负平均 VOUT 电流成比例的电流。 IMON_ON 还连接到误差放大器 EA2，可用于限制最大负 VOUT 电流。                              |     |

$ICP$和$ICN$是附加状态监测引脚

### Controller
#### Start-Up Sequence

![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240924145904.png)

$\overline{SHDN}$是LT8708的主关断引脚，当此引脚上的电压低于0.35V时整个芯片会被强制关闭（$\textcolor{red}{CHIP \quad OFF}$），此时静态电流最小，只有当电压提升到1.221V以上后，$INTV_{CC}$和$LDO33$启用之后，芯片才会启动（$\textcolor{red}{SWITCHER \quad OFF \quad 1}$） 
~~傻逼MathJax这latex语法都认不出来？蛤？~~
当$\overline{SHDN}$为高电平之后，$SWEN$用来控制启动芯片的开关稳压器（~~芝士什么~~），低于0.8V时无法启动，必须通过电阻拉高至1.208V以上才可启动，否则芯片电流检测电路将被禁用。 

$SWEN$也为高电平之后进入$\textcolor{red}{INITIALIZE}$状态，此时$SS$将被强制拉低，为软起动做准备。在此状态下，随着$SS$逐渐上升，软启动电路会在适当的方向上提供 $V_C$ 和电感器电流的逐渐斜坡。这可以防止电感电流突然浪涌，并有助于输出电压平稳地斜坡调节。 

$SS$内部有180K上拉电阻，和外部电容组成$RC$电路，当$SS$引脚电压到达1.8V时退出软起动状态开始进入正常操作状态。 
#### Power Switch Control
##### Error Amplifiers
LT8708一共有六个误差放大器
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240924163550.png)
误差放大器优先级
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240924173611.png)
在某些情况下某些误差放大器不可用
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240924173837.png)
##### Power Switch Contorl
![请注意，本图所述情况为反向传导环节.pdf](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241011163748.png)
LT8708比较有意思，由于IC内部自带了电荷泵升压，因此可以做到上管常开
###### Switch Control: Buck Region (VIN >> VOUT)
这部分很常规，上管常开之后就跟普通的Buck Controller差不多。
随着 $V_{IN}$ 和 $V_{OUT}$ 彼此接近，占空比减小，直到降压区域中转换器的最小占空比达到$DC_{(ABSMIN,M2,BUCK)}$。如果占空比低于 $DC_{(ABSMIN,M2,BUCK)}$，该区域转移至Buck-Boost区域。
$$
DC_{(ABSMIN,M2,BUCK)}=t_{on(M2,MIN)}\cdot f \cdot 100 \%
$$
其中，$t_{on(M2,MIN)}$是降压操作中同步开关的最短导通时间（典型值为 200ns，详见Electrical Characteristics）
###### Switch Control: Buck-Boost (VIN ≅ VOUT)
四开关交替导通，没啥好说的
###### Switch Control: Boost Region (VIN << VOUT)
和Buck Reigon一样，就一个公式。
随着$V_{IN}$和$V_{OUT}$彼此接近，占空比减小，直到升压区域中转换器的最小占空比达到$DC_{(ABSMIN,M3,BOOST)}$。如果占空比低于 $DC_{(ABSMIN,M3,BOOST)}$，该区域转移至Buck-Boost区域。
$$
DC_{(ABSMIN,M3,BOOST)}=t_{on(M3,MIN)} \cdot f \cdot 100 \%
$$
其中，$DC_{(ABSMIN,M3,BOOST)}$是升压操作中同步开关的最短导通时间（典型值为 200ns，详见Electrical Characteristics）
##### Uni and Bidirecional Conduction
双向模式下是CCM，单向模式下是HCM（混合电流模式和突发模式操作）和DCM 
~~（HCM？....没听过  
 没事了，datasheet后面有）~~ 
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240924172325.png)
###### CCM
伟大，无需多言
###### DCM
#### Current Monitoring and Limiting
![输入电流检测与限制](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241009180756.png)
![输出电流检测与限制](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241009180808.png)
输入和输出的电流检测是完全独立的，两边的结构都是相似的，检流电阻的压差差分信号经过一个跨导放大器连接到内部的误差放大器，并且输出到外部的检测引脚，由于引脚输出的是电流（内部会在跨导放大器的基础上加上一个$20\mu A$的电流），因此接上一个电阻就可以将其转换成电压信号，这样就可以用ADC采集到电流信息。电容仅仅是为了滤波和稳定环路（忘了在哪看到的了），几个$nF$就足够了
##### Monitors 
检测引脚的电压由以下公式决定 
$$
\begin{equation}\begin{split}
V_{IMON\_INP}=(1m\frac{A}{V} \cdot R_{Sense1}\cdot I_{IN}+20\mu A)
\cdot R_{IMON\_INP} \\
V_{IMON\_INN}=(-1m\frac{A}{V} \cdot R_{Sense1}\cdot I_{IN}+20\mu A)
\cdot R_{IMON\_INN} \\ 
\end{split}\end{equation}
$$

##### Current Limiting
电流限制的思路比较简单，注意到检流电阻的信号是加上一个偏置电流输出到外部引脚，引脚连接电阻就会有电压值，这个电压值和误差放大器的基准电压进行比较，当这个电压大于基准电压的时候误差放大器就会输出低电平到补偿环路，进而使得PWM占空比减小，也就达到了控制电流的目的。
我们以控制正向的输入电流为例（$IMON\_INP$）,内部结构见$Figure 15$，其基准电压为$1.209V$,那么$R_{IMON\_INP}$、$R_{SENSE1}$和正向输入电流$I_{IN,FWD,LIMIT}$有如下关系：
$$
R_{IMON\_INP}=\frac{1.209}{I_{IN,FWD,LIMIT}\cdot1m\frac{A}{V}\cdot R_{SENSE1}+20\mu A}
$$
并没有看到有关限制$R_{SENSE}$相关的字样，那看来是随便取，datasheet给出了一个例子。
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241010181501.png)
### Frequency
经典$RT$引脚配置频率，btw，$SYNC$可用作温度监测
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240923105631.png)
### Inductor Current Sense(Pin: CSP/CSN)
$LT8708$使用电感电流模式控制，其测量升压区域中电感器电流波形的峰值和降压区域中电感器电流波形的谷值，在任何给定周期内，电感电流的峰值（升压区域）或谷值（降压区域）由 VC 引脚电压控制。
>![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240923110607.png)
#### Slope Compensation
datasheet上有，与硬件设计无关（大雾
### IC Junction Temperature Measurement
LT8708的芯片温度会对$CLKOUT$引脚输出信号的占空比产生影响
$$
T_J \approx \frac{DC_{CLKOUT}- 34.4\%}{0.325\%} \  ^\circ C
$$
计算出来的温度有大概±10℃的误差，~~NTC算出来的误差大概多少？
感觉还是上个NTC吧，误差有点离谱了，不能芯片本身制动还要靠这个吧（）~~
### Component Selection
#### $R_{SENSE}$  Selection
这里的检流电阻特指电感电流的检流电阻，输入和输出的检流电阻不在本小节的讨论范围内，请右转Current Monitoring and Limiting。
需要计算四个区域的最大$R_{SENSE}$阻值，正向升压，反向升压，正向降压，反向降压，最后的结果小于四者的最小值，并建议流出建议留出$20\%$至$30\%$的余量。如果有一个区域没有用到时，此区域的$R_{SENSE}=\infty$
还有一部分详见datasheet P31右侧。
![会有用的.jpg](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241011105512.png)
##### Max $R_{SENSE}$ in the Boost Reigon
###### Forward Conduction
在升压区域，当$VIN$最低且$VOUT$最高时，最大正$VOUT$电流能力最低。~~（说实话没看懂啥意思）~~
此时的M3占空比
$$
DC_{MAX,M3,BOOST}\approx(1-\frac{V_{IN(MIN,BOOST)}}{V_{OUT(MAX,BOOST)}})\cdot 100\%
$$
计算出占空比后需要去看一下最大电感电流检测电压图，找到此时$R_{SENSE}$两侧的电压$V_{R_{SENSE}}$
下一级计算正向升压区域的电感纹波电流(Inductor ripple Current)，此时分两种情况
如果尚未确定此时的电感感值，取最大$V_{OUT}$负载电流的30%-50%，因此纹波电流为：
$$
\Delta I_{L(MAX,BOOST)}=\frac{V_{OUT(MAX.BOOST)}\cdot I_{OUT(MAX,FWD)}}{V_{IN(MIN,BOOST)}\cdot (\frac{100 \%}{ripple}-0.5)}A
$$
若此时电感感值已确定，那么有：

$$
\Delta I_{L(MAX,M3,BOOST)}=\frac{(\frac{DC_{(MAX,M3,BOOST)}}{100\%})\cdot V_{IN(MIN,BOOST)}}{f\cdot L}A
$$

在最大纹波电流确定后，本区域内的最大$R_{SENSE}$计算公式如下：

$$
R_{SENSE(MAX,BOOST,FWD)}=\frac{2\cdot V_{R_{SENSE(MAX,BOOST,MAXDC)}}\cdot V_{IN(MIN,BOOST)}}{(2\cdot I_{OUT(MAX,FWD)}\cdot V_{OUT(MAX,BOOST)}+(\Delta I_{L(MAX,BOOST)}\cdot V_{IN(MIN,BOOST)}))} 
$$

###### Reverse Conduction
在反向升压区域，当以最小占空比工作时，最大反向$VIN$电流最低
在计算此区域的$R_{SENSE}$大小时，首先需要确定电感纹波电流，还是两种情况。 
如果电感尚未确定，则可以通过选择$\Delta I_{L(MIN,BOOST)}$ 为升压区域中最小峰值电感电流的 10% 来估计纹波电流 $\Delta I_{L(MIN,BOOST)}$，如下所示:

$$
\Delta I_{L(MIN,BOOST)}=\frac{I_{IN(MAX,RVS)}}{\frac{100\%}{10\%}-0.5}A
$$ 
其中,$I_{IN(MAX,RVS)}$ 是反向升压区域所需的最大$VIN$负载电流.
如果电感已选型，那么可根据如下公式计算：
$$
\Delta I_{L(MIN,BOOST)}=\frac{\frac{DC_{(ADBSMIN,M3,BOOST)}}{100\%}\cdot V_{IN(MIN,BOOST)}}{f \cdot L}
$$
其中，$DC_{(ADBSMIN,M3,BOOST)}$是升压区域的最小占空比。
在得到电感纹波电流之后，本区域的$R_{SENSE}$可根据如下公式计算：
$$
R_{SENSE(MAX.BOOST,RVS)}=\frac{2\cdot |V_{RSENSE(MIN,BOOST,MINDC)}|}{(2\cdot I_{IN(MAX,RVS)})-\Delta I_{L(MIN,BOOST)}}
$$
其中，$V_{RSENSE(MIN,BOOST,MINDC)}$是在升压区域下最小占空比时的$R_{SENSE}$两侧电压（最小电感电流检测电压），典型值为$-93mV$。
若上式的计算结果为负，则说明任何阻值都可满足要求，将本区域内的阻值视为$\infty$并进入下一部分计算
##### Max $R_{SENSE}$ in the Buck Reigon
###### Forward Conduction
在降压区域中，当以最小占空比运行时，最大$VOUT$电流能力最低。
和升压区域中的反向导通一样，仍然需要首先计算电感纹波电流$\Delta I_{L(MIN,BUCK)}$，两种情况：
如果电感感值未知，纹波电流$\Delta I_{L(MIN,BUCK)}$可以通过选择$\Delta I_{L(MIN,BUCK)}$为降压区域中最大峰值电感电流的 10% 来估算~~(这句话真的好怪，翻译出来是这样但是你看公式又不是那味)~~，如下所示
$$
\Delta I_{L(MIN,BUCK)}=\frac{I_{OUT(MAX,FWD)}}{\frac{100\%}{10\%}-0.5}A
$$
其中，$I_{OUT(MAX,FWD)}$是正向降压区域所需的最大$VOUT$负载电流。
如果电感感值已知，那么可根据下式计算：
$$
\Delta I_{L(MIN,BUCK)}=\frac{\frac{DC_{ABISMIN,M2,BUCK}}{100\%}\cdot V_{OUT(MIN,BUCK)}}{f\cdot L}A
$$
其中，$DC_{ABISMIN,M2,BUCK}$是先前计算的降压区域的最小占空比。
在得到电感纹波电流之后，本区域的$R_{SENSE}$计算公式为：
$$
R_{SENSE(MAX,BUCK,FWD)}=\frac{2\cdot V_{R_{SENSE(MAX,BUCK,MINDC)}}}{2\cdot I_{OUT(MAX,FWD)}-\Delta I_{L(MIN,BUCK)}} \ohm
$$
其中，$V_{R_{SENSE(MAX,BUCK,MINDC)}}$是在最小占空比时的最大电感电流检测电压，典型值为$82mV$。
若上式的计算结果为负，则说明任何阻值都可满足要求，将本区域内的阻值视为$\infty$并进入下一部分计算
###### Reverse Forward
在降压区域，当$VIN$处于最大值并且$VOUT$处于降压操作的最小值时，最大反向$VIN$电流能力最小。
首先找到$VIN$最小且$VOUT$最大时的降压区域占空比
$$
DC_{(MAX,M2,BUCK)}=(1-\frac{V_{OUT(MIN,BUCK)}}{V_{IN(MAX,BUCK)}})\cdot 100\%
$$
然后计算电感纹波电流，如果电感未知，那么最大纹波电流 $\Delta I_{L(MAX,BUCK)}$ 可以通过选择$\Delta I_{L(MAX,BUCK)}$为降压区域最大峰值电感电流的$30\%$至$50\%$来估算，如下所示:
$$
\Delta I_{L(MAX,BUCK)}= \frac{V_{IN(MAX,BUCK)}\cdot I_{IN(MAX,RVS)}}{V_{OUT(MIN,BUCK)}\cdot (\frac{100\%}{Ripple}-0.5)}A 
$$
如果电感已知，那么可根据下式计算的更精确一点：
$$
\Delta I_{L(MAX,BUCK)}=\frac{\frac{DC_{(MAX,M2,BUCK)}}{100\%}\cdot V_{OUT(MIN,BUCK)}}{f\cdot L}A
$$
其中，$DC_{(MAX,M2,BUCK)}$是先前计算的降压区域的最大占空比。
那么，最终本区域的$R_{SENSE}$据下式计算：
$$
R_{SENSE(MAX,BUCK,RVS)}=\frac{2\cdot |V_{R_{SENSE()MIN,BUCK,MAXDC}}|\cdot V_{OUT(MIN,BUCK)}}{(2\cdot |I_{IN(MAX,RVS)}|\cdot V_{IN(MAX,BUCK)})+(\Delta I_{L(MAX,BUCK)}\cdot V_{OUT(MIN,BUCK)})} \ohm
$$
其中，$V_{R_{SENSE()MIN,BUCK,MAXDC}}$是最大占空比时的最小电感电流检测电压。该值的确定方式与之前在$R_{SENSE}$ Selction Boost  Reigon部分中的最大$R_{SENSE}$中讨论的$V_{R_{SENSE(MAX,BOOST,MAXDC)}}$ 类似
##### $R_{SENSE}$ Filtering
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241011162638.png)
10(a)和10(b)都可以，但是10(b)的接法会在电流经过电容时引入接地走线噪声（？从何而来的结论）

#### Mosfet Selection
##### Dissipation Calculate

## INA226
INA并不直接测量电流，它测量检流电阻两侧的差分电压得到电流值（~~what can i say.jpg~~）
貌似用不到了，直接用ADC加个OPAMP采一下监控引脚就能拿到电压电流值了，非常的nice。
## SuperCap
Viashy的[220 EDLC ENYCAP™ Energy Storage Capacitors | Vishay](https://www.vishay.com/zh/product/28421/),2.7V 50F
手册的最小放电电压为1.5V，最大充放电电流为4.4A
11串电容单体，电压为$$2.7\times11=29.7V$$
此时能量
$$
E = \frac{1}{2}CU^2=2004.75J
$$
被动均衡，充电至2.65V
## 裁判系统要求
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240927164538.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240926024221.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240926024232.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240926024241.png)
![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20240926024247.png)
## BOM
# 杂记
## 参考链接和不认识的玩应
1. as intended 按预期
2. methodical $adj.$ 有条理的
3. criteria $n.$ 标准
4. optimal $adj.$ 最合适的
5. Subharmonic $n.$ 次谐波
6. dissipate $v.$ 消失
7. Potentiometer $n.$ 电位计
8. illustrate $v.$ 说明，阐明
9. respective $adj.$ 分别
10. be proportional to 正比于
11. Amps. 安培
12. significantly $adj.$ 极大的，显著的
13. product $n.$ 乘积
14. trip $v.$ 跳闸（？我觉得说成跳变也许更好）![](https://raw.githubusercontent.com/WwWangGuan/pics/main/20241011171250.png)
15. [LT8708 Datasheets and Production Info | Analog Devices](https://www.analog.com/en/products/lt8708.html)
16. [Latex符号大全](https://zhuanlan.zhihu.com/p/472919794)
17. [LaTex中输入空格以及换行\_ctex换行-CSDN博客](https://blog.csdn.net/luolang_103/article/details/81289529)
18. [OB0207 obsidian 自动获取url链接：auto-link-title插件使用\_auto link title-CSDN博客](https://blog.csdn.net/qq_41520353/article/details/128675834)
19. [Latex中如何设置字体颜色（3种方式） - Tsingke - 博客园](https://www.cnblogs.com/tsingke/p/7457236.html)
20. [latex 写作常见问题记录\_overleaf正文有百分数后面的内容不显示了-CSDN博客](https://blog.csdn.net/JM1307hhh/article/details/128448860?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522416F45B3-C90F-4F11-A77D-1A1E76672115%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=416F45B3-C90F-4F11-A77D-1A1E76672115&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-128448860-null-null.142^v100^pc_search_result_base3&utm_term=latex%20%E7%99%BE%E5%88%86%E5%8F%B7%E6%80%8E%E4%B9%88%E6%89%93&spm=1018.2226.3001.4187)
21. [在LaTeX中如何输入摄氏度的符号\_latex 摄氏度-CSDN博客](https://blog.csdn.net/namishizi321/article/details/128028757)
22. [工程师指南：如何动态调整合适的输出电压 | Analog Devices](https://www.analog.com/cn/resources/technical-articles/dynamically-adjust-the-right-output-voltage.html)
23. [具有电压输出智能 DAC 的电压裕量和调节电路 (Rev. A)](https://www.ti.com.cn/cn/lit/an/zhcadz3a/zhcadz3a.pdf?ts=1727229600659)
24. [DAC基本架构II：二进制DAC](https://www.analog.com/media/cn/training-seminars/tutorials/mt-015_cn.pdf)
25. [如何快速安全地为超级电容器充电](https://www.ti.com.cn/cn/lit/an/zhcacg4/zhcacg4.pdf?ts=1727247745601&ref_url=https%253A%252F%252Fwww.ti.com.cn%252Fproduct%252Fcn%252FBQ25713)
26. [超级电容器的熟练使用方法](https://www.chemi-con.co.jp/download/pdf/dl-technote-c.pdf)
27. [ENYCAPTM Instruction Manual](https://www.vishay.com/docs/28495/enycapinstrmanual.pdf)
## 电压裕度
没事了，自己百度吧，是我多虑了