# 什么是辅助电源？  
在电源设计中，除了主要功率电源，还需要与之隔离的电源为控制器、电压电流检测模块、驱动等各类模块进行供电。主电路电压可达几十甚至上百伏，但是一般IC供电电压为3.3V甚至更低，主电路电源无法直接进行供电。此外，由于主电路电压波动较大，而像处理器和ADC等精密器件对输入电压的稳定性有较高要求，需辅助电源为其提供稳定电压。
# 辅助电路设计需求
* ## 输入电压
    输入电压选择为10~15V，涵盖3s锂电池电压范围和驱动开关范围
* ## 输出电压
    输出电压3.3V，满足大部分处理器供电电压
* ## 最大输出电流
  处理器型号尚未确定，但目前笔者用过的高性能微控制器STM32H7系列在极限性能400MHz下，其输入电流也在100mA左右，考虑拓展性和其他IC供电以及裕量，输出电流为300mA
* ## 纹波
  越小越好
* ## 效率
  越高越好，辅助电源负载较轻，在低负载位置效率敏感
* ## 价格
  越低越好
# 电源方案
采用buck降压电路，将输入电压降至5V再通过线性稳压电源降至3.3V。开关电源采用开关器件，通过高频开关降低输出电压；而线性稳压电源则是利用三极管的线性工作区，通过改变三极管的等效电阻实现对输出电压的稳定。
![BUCK电路拓扑](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/cabb8d8cbc27ca56e23e43f0d7712ef3.jpg)
当开关S导通时，二极管反偏，电流通过电感L向负载供电；当开关S断开时，电感L续流，二极管导通产生续流回路，继续向负载供电。
![LDO电路拓扑](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/Screenshot%202024-04-29%20152309.png)
图中为NMOS LDO（low-dropout regulator），当输出电压偏低时，放大器反向端输入引脚电压降低，导致放大器输出增大，从而导致开关管$V_{GS}$增大，使得开关管等效电阻减小，最终导致输出电压的升高，实现了一个电压串联负反馈。当$V_{GS}$不断增加直至开关管饱和时，其等效电阻达到最小，但在开关管上仍存在压降，这便导致了***dropout***。此外，当输入电压降低时，$V_{GS}$随之降低，从而导致压降增大，可通过增加偏置电源$V_{BIAS}$解决上述问题。

通过上述对buck电路和线性稳压电路分析可以看出，buck电路通过开关器件降低电压，而线性稳压器则通过在开关管上消耗掉多余功率实现降压，这直接导致了两种电路的特点。buck电路效率较高，高速开关频率使得体积减小，变换范围大，但缺点就是输出噪声大，会有电磁干扰；而线性稳压器效率较低，这也导致其发热严重，为了散热需增大其体积甚至需要增加散热片，但是其输出稳定，噪声较小，且对于动态响应较好。

根据两种电源的特点采用相互配合的方式，通过buck电路将主电路12V电压降至5V，保证电源转换效率的同时降低电压等级，再通过线性稳压器件将5V电压转换至3.3V。根据LDO能量损耗公式(1)和(2)，可以看出LDO上的损耗与输入电压和输出电压有很大关系，在相同输出电流下，输入输出电压差值越大，LOD功耗就越高。式(1)中的$I_{ground}$很小，在带负载的情况下可以忽略不记。LDO的功耗越高，为了能够正常工作就需要更小的热阻来维持合适的工作温度，从而增大器件尺寸，降低系统集成度和效率。buck电路直接降压至3.3V会导致输出纹波较大，对于输入电源有较高要求的精密器件难以承受，增加滤波电路的设计难度。
$$P_D=[(V_{IN}-V_{OUT})*I_{OUT}]+(V_{IN}*I_{ground}) \tag{1}$$
$$T_J=T_A+(R_{\theta JA}*P_D) \tag{2}$$
# 电源参数设计
* ## BUCK电路
  考虑到日后利用分立式元件搭建buck电路，在辅助电源中buck电路尽可能采用模块化、高集成度的设计；输入电压为10~15V，输出电压为5V；为了增加拓展性，buck电路输出电流考虑为2A；价格在1\~2元
  在挑选器件过程中发现降压模块种类繁多，有集成开关管的还有外接开关管的，有隔离的还有非隔离的，同步的异步的，ECO和FCCM的等等。
  * 其中外接开关管的多适用于大功率场合，且集成度不高；
  * 隔离式电源拓扑有正激、反激等拓扑，非隔离型有buck、buck-boost等，隔离型电源成本高，结构复杂，效率低，而非隔离性则有高集成度，成本低的特点；
  * 同步整流则是将图中二极管替换为开关管，同步整流大大降低了损耗的同时提升了成本，此外在轻载情况下，异步整流无法提供电流逆向路径导致断流，而同步则避免了这种情况；
  ![异步](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/%E5%9B%BE1%EF%BC%9A%E5%BC%82%E6%AD%A5%E6%95%B4%E6%B5%81%E5%8E%9F%E7%90%86%E5%9B%BE.jpg)
  ![同步](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/%E5%9B%BE2%EF%BC%9A%E5%90%8C%E6%AD%A5%E6%95%B4%E6%B5%81%E5%8E%9F%E7%90%86%E5%9B%BE.jpg)
  ![轻载](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/%E5%9B%BE4%EF%BC%9A%E5%90%8C%E6%AD%A5%E6%95%B4%E6%B5%81%E5%BC%82%E6%AD%A5%E6%95%B4%E6%B5%81%E5%9C%A8%E8%BD%BB%E8%BD%BD%E5%B7%A5%E4%BD%9C%E6%97%B6%E7%9A%84%E7%8A%B6%E6%80%81.jpg)
  * FCCM为force continues conducting mode，其于ECO模式主要区别在轻载模式下。当开关电源工作在轻载模式下时电流效率降低，如图所示。ECO模式可以在轻载模式下降低开关频率，从而达到提高电源效率的目的，但是此时输出电压纹波增大；FCCM模式则是固定开关频率，使得输出纹波减小但是效率降低
  ![ECO效率](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/pastedimage1670383795123v1.png)
  ![FCCM效率](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/pastedimage1670383866684v2.png)
  ![ECO轻载](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/pastedimage1670384180420v3.png)
  ![FCCM轻载](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/pastedimage1670384205861v4.png-640x480.png)
  * 控制方法还有D-CAP和D-CAP2以及ACM/AECM

  在ti官网进行筛选比对，最终选择了TPS562202芯片。该芯片输入电压再4.3\~17V，输出电压再0.804\~7V，集成开关管，输出电流最大可达2A，有ECO模式，同步整流，D-CAP2控制、开关频率在580kHz，其封装为DRL(6)，尺寸仅为1.6\*1.6mm。
  该芯片的主要特性在其数据手册8.3章节有详细描述。其中对于ECO模式的启动电流是根据电感电流是否过零判断，当电感电流经过零点时，下开关管关断，电感保持断流状态，此时上开关管导通时间不变，下管关断时间增加从而达到开关频率降低的目的，ECO模式负载电流计算公式如式3：
$$I_{OUT(LL)}=\frac{1}{2Lf_{SW}}*\frac{(V_{IN}-V_{OUT})*V_{OUT}}{V_{IN}}  \tag{3}$$
  在此对上述公式做出解释：考虑电流连续工作模式临界点即电流最小值为0，假设输出电压稳定为$V_{OUT}$，则电流增长率为$\frac{V_{IN}-V_{OUT}}{L}$，电源周期为开关频率的倒数，占空比为$\frac{V_{OUT}}{V_{IN}}$，则上管导通时间为$\frac{V_{OUT}}{f_{SW}V_{IN}}$，乘以电流上升率并求其平均值可得式3。
  数据手册推荐设计电路图：
![推荐设计电路图](https://github.com/HUSTdzc/Auxiliary-SMPS/blob/main/IMAGE/Screenshot%202024-05-01%20135726.png)

  其中重要参数设计如下：
  * 输出分压电阻，芯片内部参考电压为0.804V，输出电压经过电阻分压后反馈至参考电压，当下电阻10kΩ时，上电阻选择52.19kΩ，采用最接近的1%标准电阻52.3kΩ。
  > 此时电阻选择既不能过大也不能过小，若电阻选择过小则会导致电阻之路电流较大，在其上产生损耗；而电阻过大则会导致电阻电流较小，容易受外界噪声干扰，并且芯片输入端电流无法忽略
  * 输出滤波电路设计，该滤波电路设计不仅考虑了芯片开关频率，还主要考虑了芯片内部D-CAP2控制方法所产生的零点，在此选择数据手册中的推荐值：电感4.7uH，电容44uF
  * 电感电流纹波计算如式4，与式3推导过程相同不再赘述，通过计算在理想满载状况下输出电感电流纹波峰峰值为1.15A，电流最大值峰值2.575A，电流有效值2.027A（该结果计算于开关频率540kHz，根据图示在12V输入5V输出情况下开关频率在540kHz附近），在普通断流模式下电感电流纹波减小，但是由于ECO模式导通时间不变，所以电感电流波动峰值不变。
  $$I_{P-P}=\frac{V_{OUT}}{V_{IN}}*\frac{V_{IN}-V_{OUT}}{L_O*f_{SW}} \tag{4}$$
  输出电压纹波计算可通过对电感纹波电流积分计算出电容两端电荷量变化，再通过电荷量变化计算电容两端电压变化，计算如下所示：
  $$\Delta Q=\frac{I_{P-P}}{8f_{SW}} \tag{5}$$
  $$V_{P-P}=2*\frac{\Delta Q}{C} =\frac{V_{OUT}(V_{IN}-V_{OUT})}{4V_{IN}L_Of_{SW}^2C} \tag{6}$$
  计算可得输出电压峰峰值在0.0121V，频率为开关频率540kHz。在轻载ECO模式下，电感纹波不变，负载电流却减小，这将导致涌入电容电荷增多，从而导致输出电压波动增加，当输出空载时最大波动为连续模式的四倍。
  > 上述对输出波动分析建立在理想器件情况下，实际情况下电感与电容均有等效电阻存在，应选择尽量理想器件。滤波电容可选择等效电阻较低的陶瓷电容，并且在输出通过并联电容的方式达到在增加电容容量的同时降低等效电阻。此外，在存在直流偏置的情况下电容容值也会有所下降，此部分将在LDO设计时进行描述。
  * 输入滤波电容设计，滤波电容主要是用来与前级电路解耦，减小输入电压纹波。此处数据手册建议使用10uF与0.1uF并联达到较好的滤波效果。
  > 实际电容存在等效电阻与等效电感，等效电感与电容形成谐振，较大的电容具有较低的谐振频率，对于高频信号表现为感性；而较小的电容谐振频率较高，但是其容值较小，截至频率较高，对于中低频抑制性能不如大电容。因此大小电容并联可使得滤波电路兼具低频和高频的良好滤波性能。
  * *BootStrap*电容选取0.1uF，BST引脚为上开关管输入引脚，用于自举驱动开关管。
  * $C_{FF}$为前馈电容，输出高频噪声通过该电容形成较小阻抗回路，使得输出高频噪声增益降为1，从而减小输出噪声。此外该电容还可使该系统引入零点，增加系统相位裕度，提高系统稳定性。再次选择推荐值22pF，具体设计可参考[德州仪器手册](https://www.ti.com/lit/an/sbva042/sbva042.pdf?ts=1714551248183)

* ## LDO电路
  LDO输入电压为5V，输出电压为3.3V，考虑裕量选取输出电流300mA，价格在1元以内。
