## 郊狼脉冲波形

本文我们将关于郊狼脉冲主机的波形协议进行部分解释，帮助广大郊狼爱好者更好理解脉冲波形的数据原理。

### 概念解释

#### 波形频率

郊狼的程序把每一秒分割成1000毫秒，在每个毫秒内都可以产生一次脉冲。我们通过控制脉冲和间隔来编码脉冲产生的规律。

波形频率表示一个输出单元的时长(输出单元由X个脉冲和Y个间隔组成)，单位是ms。

所以如果您希望输出一个1Hz的波形，实际上波形频率=1000ms；若输出50Hz的波形，波形频率=20ms。

|  脉冲频率  |  波形频率  |
|:------:|:------:|
|  1Hz   | 1000ms |
|  5Hz   | 200ms  |
|  10Hz  | 100ms  |
|  50Hz  |  20ms  |
| 100Hz  |  10ms  |
| 500Hz  |  2ms   |
| 1000Hz |  1ms   |

在郊狼V2 协议中，波形频率由[X 和 Y](/coyote/v2/README_V2.md)共同决定，其中X代表连续X毫秒发出X个脉冲，Y表示X个脉冲过后会间隔Y毫秒再发出X个脉冲并循环。

> e.g<br/>
参数【1,9】代表每隔9ms发出1个脉冲，总共耗时10ms，也就是脉冲频率为100hz，波形频率为10ms。参数【5,95】代表每隔95ms发出5个脉冲，总共耗时100ms，由于这五个脉冲连在一起并且持续时间仅为5ms，因此使用者只会感受到一次（五合一）脉冲，因此在使用者的体感中脉冲频率为10hz，波形频率为100ms

在郊狼V3 协议中，我们只提供了波形频率值的输入，而波形频率中脉冲(X)和间隔(Y)的分配由V3协议中的 [频率平衡参数1](/coyote/v3/README_V3.md) 来决定。

V3 协议中，波形频率的输入还需经过一次压缩转化，若用户想要波形频率线性变化，请参考以下例子。

>e.g 1<br/>1. 确定要线性变化的脉冲频率，举例：1Hz，2Hz，3Hz，4Hz，5Hz，6Hz，7Hz，8Hz，9Hz，10Hz<br/>2. 转换为波形频率值: 1000ms,500ms,333ms,250ms,200ms,166ms,142ms,125ms,111ms,100ms<br/>3. 根据V3 协议的转化公式，将波形频率转化为实际输入值：240，180，146，130,120,113,108,105,102,100<br/>4. 根据协议4个一组，不足的0补，每100ms向设备输入

>e.g 2<br/>1. 确定要线性变化的波形频率值，举例：100ms，200ms，300ms，400ms，500ms，600ms，700ms，800ms，900ms，1000ms<br/>2. 根据V3 协议的转化共识，将波形频率转化为实际输入值：100，120，140，160，180，200，210，220，230，240<br/>3. 根据协议4个一组，不足的0补，每100ms向设备输入

#### 波形频率与实际输入值的转化

由于人体对于频率的细微变化感知并不敏感，波形频率(输出单元)越大，则波形频率(输出单元时长)变化相对应的脉冲频率变化越细微。另外这个转化可以压缩数据长度，可以以更短的数据进行交互

| 波形频率  | 输出值 |  脉冲频率  |
|:-----:|:---:|:------:|
| 10ms  | 10  | 100Hz  |
| 20ms  | 20  |  50Hz  |
| 50ms  | 50  |  20Hz  |
| 100ms | 100 |  10Hz  |
| 110ms | 102 |  9Hz   |
| 150ms | 110 | 6.6Hz  |
| 650ms | 204 | 1.53Hz |
| 680ms | 208 | 1.47Hz |
| 750ms | 215 | 1.33Hz |

#### 波形强度

一次脉冲由两个对称的正负单极性脉冲组成，两个单极性脉冲的高度(电压)由这个通道的强度决定。我们通过控制脉冲宽度的方式控制脉冲带来的感受的强弱。脉冲越宽，感受就越强，反之脉冲越窄感受约越弱。脉冲宽度的节奏性变化可以创造出不同的脉冲感受。

在郊狼V2 协议中，波形强度由[Z](/coyote/v2/README_V2.md)决定,官方APP的值范围是(0 ~ 20)，波形强度是一个相对值，所以并没有实际的单位来表示

在郊狼V3 协议中，波形强度的值范围是(0 ~ 100)，波形强度是一个相对值，所以并没有实际的单位来表示

V2的波形强度与V3的波形强度的映射关系: (V2 协议中 波形强度 20) ≈ (V3 协议中 波形强度 100)

### 输出窗口

V2 协议的输出窗口为100ms，V3 协议的输出窗口为25ms，但每次设置的数据为4组，所以依然可以认为是100ms。

很多人对于波形频率值和输出窗口值的冲突表示困惑，波形频率时长是大于输出窗口的，那么对于下一个周期输入的数据设备将如何处理？

我们的设备内部对于这一情况有一套相对复杂的处理方式，处理细节暂不描述，但给出几个建议：

1. 若希望稳定输出某个波形频率，建议稳定输入该波形频率值
2. 若波形频率值大于输出窗口，下一周期输入的波形频率有所变化，那么真正的输出脉冲可能并非输入值所对应的效果，而是经过复杂方式处理后的效果

### V2协议波形 转化为 V3协议波形

V3波形频率 = V2 (X + Y) 后，执行(10 ~ 1000) -> (10 ~ 240)的转化

V3波形强度 = V2 (Z * 5)