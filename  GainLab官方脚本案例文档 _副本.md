# GainLab Script 脚本案例

```
//@name=MA
//@title=MovingAverage 移动平均线
//@desc = ## MovingAverage 移动平均线
//作为衡量主力成本重要的参照指标，用以观察价格变动趋势和起到测试压力和支撑的作用。
//当作为趋势判断的时候，周期越长、越有效。
//另一方面，需要注意的是，当均线形成多头排列后，短期均线才是作为持仓的依据。
//#### 应用法则：
//- 平均线从下降逐渐走平，当价格从平均线的下方突破平均线时是为买进信号。
//- 价格连续上升远离平均线之上，价格突然下跌，但未跌破上升的平均线，价格又再度上升时，可以加码买进。
//- 价格虽一时跌至平均线之下，但平均线仍在上扬且价格不久马上又恢复到平均线之上时，仍为买进信号。
//- 价格跌破平均线之下，突然连续暴跌，远离平均线时，很可能再向平均线弹升，这也是买进信号。
//- 价格急速上升远超过上升的平均线时，将出现短线的回跌，再趋向于平均线，这是卖出信号。
//- 平均线走势从上升逐渐走平转弯下跌，而价格从平均线的上方，往下跌破平均线时，应是卖出信号。
//- 价格跌落于平均线之下，然后向平均线弹升，但未突破平均线即又告回落，仍是卖出信号。
//@position=main
//@version=1

period1 = I.int(5, title="周期1", min=1) 
period2 = I.int(20, title="周期2", min=1)
period3 = I.int(50, title="周期3", min=1) 
period4 = I.int(100, title="周期4", min=1) 
period5 = I.int(200, title="周期5", min=1)  
source = I.select('close', title="数据源", tip='用来计算的数据', options=SOURCE)

maStyle1 = S.line('#F00',1, 'solid', title="MA1样式") 
maStyle2 = S.line('#FF0',1, 'solid', title="MA2样式") 
maStyle3 = S.line('#0FF',1, 'solid',false, title="MA3样式")
maStyle4 = S.line('#F0F',1, 'solid',false, title="MA4样式") 
maStyle5 = S.line('#0F0',1, 'solid',false, title="MA5样式") 
ma1 = F.ma(dataList, period1, source)
ma2 = F.ma(dataList, period2, source)
ma3 = F.ma(dataList, period3, source)
ma4 = F.ma(dataList, period4, source)
ma5 = F.ma(dataList, period5, source)
D.line(ma1,maStyle1)
D.line(ma2,maStyle2)
D.line(ma3,maStyle3)
D.line(ma4,maStyle4)
D.line(ma5,maStyle5)
setPrecision('price')
O.tools('MA'+period1,ma1,maStyle1) 
O.tools('MA'+period2,ma2,maStyle2) 
O.tools('MA'+period3,ma3,maStyle3) 
O.tools('MA'+period4,ma4,maStyle4) 
O.tools('MA'+period5,ma5,maStyle5)
```

```
//@name=FVG
//@title=Fair Value Gap
//@desc=## FVG (Fair Value Gap) 公允价值缺口指标
//FVG是一种用于识别价格图表中缺口区域的技术分析工具。当价格快速移动时，会在图表上留下"公允价值缺口"，这些缺口通常会被后续价格回补。
//指标识别两种类型的FVG：
//1. **看涨FVG（绿色）**：当当前K线的最低价高于前两根K线的最高价时形成，表示价格向上跳跃留下的缺口
//2. **看跌FVG（红色）：当当前K线的最高价低于前两根K线的最低价时形成，表示价格向下跳跃留下的缺口
//#### 应用法则：
//- FVG区域通常会作为支撑或阻力位
//- 价格经常会回到FVG区域进行回补
//- 看涨FVG可作为潜在的支撑区域
//- 看跌FVG可作为潜在的阻力区域
//- 当价格完全填补FVG时，该缺口失效
//@position=main
//@version=1
minGapPercent = I.float(0.1,title='最小缺口百分比')
maxFVGBoxes = I.int(20,title='最大显示FVG数量',max=200,min=1)
maxFVGScope = I.int(200,title='FVG作用域',tip='最大K线数量',max=999,min=1)
bullishFVG = S.rect('rgba(0, 255, 0, 0.3)','fill',title='看涨FVG')
bearishFVG = S.rect('rgba(255, 0, 0, 0.3)',1,'fill','solid',title='看跌FVG')
bullishFilledFVG = S.rect('rgba(0, 255, 0, 0.1)',1,'fill','solid',title='已填补看涨FVG')
bearishFilledFVG = S.rect('rgba(255, 0, 0, 0.1)',1,'fill','solid',title='已填补看跌FVG')
fvgLabel = S.label('rgb(255, 255, 255)',12,'left','value',title='FVG标签')
fvgLabelBg=S.labelbg('rgba(0, 0, 0,0.6)','fill',title='标签背景')
filledFVGLabel = S.shape('check','#FF0',12,title='已填补记号')
let fvgList = []
let activeFVGs=[]
dataList.forEach((kData,i)=>{
    let fvg = {...kData}
    if(i > 1){        
        let prev2 = dataList[i - 2]
        let endIndex = maxFVGScope + i - 2
        if(endIndex > dataList.length){
            endIndex = dataList.length
        }
        if (kData.low > prev2.high) {
            const gapSize = kData.low - prev2.high
            const gapPercent = (gapSize / kData.close) * 100
            if (gapPercent >= minGapPercent) {	
                fvg.type = 'buy'
                fvg.start = kData.low
                fvg.end =prev2.high
                fvg.startIndex = i - 2
                fvg.endIndex = endIndex
                fvg.filled = false
                activeFVGs.push(i)
            }
        }
        if (kData.high < prev2.low) {
            const gapSize = prev2.low - kData.high
            const gapPercent = (gapSize / kData.close) * 100
            if (gapPercent >= minGapPercent) {				
                fvg.type = 'sell'
                fvg.start = prev2.low
                fvg.end = kData.high
                fvg.startIndex =  i - 2
                fvg.endIndex = endIndex
                fvg.filled = false
                activeFVGs.push(i)
            }
        }
        for (let j = 0; j < activeFVGs.length; j++) {
            if(activeFVGs[j] < i){
                const aFvg = fvgList[activeFVGs[j]]
                if (!aFvg.filled) {
                    let isFilled = false
                    if (aFvg.type === 'buy') {
                        isFilled = kData.low <= aFvg.end
                    } else {
                        isFilled = kData.high >= aFvg.start
                    }
                    if (isFilled) {
                        aFvg.filled = true
                        aFvg.endIndex = i
                    }
                }
            }			
        }		
    }
    fvgList.push(fvg)
})
labelList = []
filledList = []
fvgList.forEach((item,index)=>{
    labelList.push(null)
    filledList.push(null)
    if(item.start){
        labelList[index - 2] = {value:item.start,text:item.type==='buy'?'看涨':'看跌'}
        if(item.filled){
            filledList[index - 2] = item.start
        }
    }	
})
D.rect(fvgList,({value})=>{
    if(value.type=== 'buy' ){
        return value.filled ? bullishFilledFVG : bullishFVG
    }else{
        return value.filled ? bearishFilledFVG : bearishFVG
    }
})
fvgLabel.y = fvgLabel.size
fvgLabel.x = 5
filledFVGLabel.y = fvgLabel.size + filledFVGLabel.size
filledFVGLabel.x =  filledFVGLabel.size
D.label(labelList,fvgLabel,fvgLabelBg)
D.shape(filledList,filledFVGLabel)
```

```
//@name=SuperTrend
//@title=SuperTrend 超级趋势
//@desc=## SuperTrend 超级趋势指标
//SuperTrend是一个基于ATR（平均真实范围）的趋势跟踪指标，能够有效识别趋势方向和买卖信号。
//#### 主要特点：
//1. 基于ATR计算动态支撑阻力位
//2. 清晰的趋势方向指示
//3. 明确的买卖信号
//4. 适用于各种时间周期
//#### 应用法则：
//- 绿色线表示上涨趋势，红色线表示下跌趋势
//- 价格突破SuperTrend线时产生交易信号
//- 上涨趋势中，SuperTrend线作为动态支撑
//- 下跌趋势中，SuperTrend线作为动态阻力
//@position=main
//@version=1

length = I.int(10, title="ATR长度", min=1, max=50)
multiplier = I.float(3.0, title="ATR倍数", min=0.1, max=10.0, step=0.1)
source = I.select('hl2', title="数据源", options=SOURCE)

trendUpLine = S.line('#00FF00', 2,"solid",title="上涨趋势")
trendDownLine = S.line('#FF0000',  2,"solid",title="下跌趋势")
trendUpFill = S.area('rgba(0, 255, 0, 0.1)', title="上涨趋势填充")
trendDownFill = S.area('rgba(255, 0, 0, 0.1)', title="下跌趋势填充")

atr = F.atr(dataList, length)
sourceValues = F.attr(dataList, source)

superTrendValues = []
directionValues = []
upperBandValues = []
lowerBandValues = []
superTrendUpValues = []  
superTrendDownValues = []
superTrend = 0
direction = 1
upperBand = 0
lowerBand = 0
highList = F.attr(dataList,'high')
lowList = F.attr(dataList,'low')
for (let i = 0; i < dataList.length; i++) {
    if (i >= length && atr[i]) {
        // 计算基本上下轨
        const hl2 = sourceValues[i]
        const atrValue = atr[i]
        
        upperBand = hl2 + (multiplier * atrValue)
        lowerBand = hl2 - (multiplier * atrValue)
        
        // 计算最终上下轨
        const prevUpperBand = i > 0 ? (upperBandValues[i - 1] || upperBand) : upperBand
        const prevLowerBand = i > 0 ? (lowerBandValues[i - 1] || lowerBand) : lowerBand
        
        upperBand = upperBand < prevUpperBand || dataList[i - 1].close > prevUpperBand ? upperBand : prevUpperBand
        lowerBand = lowerBand > prevLowerBand || dataList[i - 1].close < prevLowerBand ? lowerBand : prevLowerBand
        
        // 计算SuperTrend方向
        const prevSuperTrend = i > 0 ? (superTrendValues[i - 1] || lowerBand) : lowerBand
        const prevDirection = i > 0 ? (directionValues[i - 1] || 1) : 1
        
        if (prevDirection === 1) {
            if (dataList[i].close < lowerBand) {
                direction = -1
                superTrend = upperBand
            } else {
                direction = 1
                superTrend = lowerBand
            }
        } else {
            if (dataList[i].close > upperBand) {
                direction = 1
                superTrend = lowerBand
            } else {
                direction = -1
                superTrend = upperBand
            }
        }
    } else {
        superTrend = 0
        direction = 1
        upperBand = 0
        lowerBand = 0
    }
    
    superTrendValues.push(superTrend)
    directionValues.push(direction)
    upperBandValues.push(upperBand)
    lowerBandValues.push(lowerBand)
    if (direction === 1) {
        superTrendUpValues.push(superTrend)
        superTrendDownValues.push(null)
    } else {
        superTrendUpValues.push(null)
        superTrendDownValues.push(superTrend)
    }
}

D.area(superTrendUpValues, highList, trendUpFill)
D.area(superTrendDownValues, lowList, trendDownFill)
D.line(superTrendUpValues, trendUpLine)
D.line(superTrendDownValues, trendDownLine)



// 添加工具提示
O.tools('SuperTrend', superTrendValues, trendUpLine)

setPrecision('price')
```

```
//@name=Smoothed NASI
//@title=SuperTrend Smoothed Heiken Ashi
//@desc=## Smoothed Heiken Ashi 平滑平均K线指标
//这是一个基于平均K线的改进版本，通过在计算平均K线之前和之后都应用平滑处理，来减少噪音并提供更清晰的趋势信号。
//#### 指标特点：
//- 前置平滑：在计算平均K线之前对OHLC数据进行平滑处理
//- 后置平滑：对计算出的平均K线数据再次平滑
//- 多种平滑方式：支持20多种移动平均线类型
//- 趋势识别：通过颜色变化清晰显示上升和下降趋势
//#### 应用法则：
//- 绿色蜡烛表示上升趋势，红色蜡烛表示下降趋势。
//- 相比传统平均K线，平滑版本能更好地过滤市场噪音，提供更可靠的趋势信号。
//@position=main
//@version=1

haSmootLength = I.int(10, title="前置平滑周期", min=1)
haSmoothMaType = I.select('EMA', title="前置平滑类型", options=['SMA', 'EMA', 'WMA', 'RMA', 'DEMA', 'TEMA', 'HMA','LSMA', 'JMA'])
haAfterSmoothLength = I.int(10, title="后置平滑周期", min=1)
haAfterSmoothMaType = I.select('EMA', title="后置平滑类型", options=['SMA', 'EMA', 'WMA', 'RMA', 'DEMA', 'TEMA', 'HMA','LSMA', 'JMA'])

upCandle = S.candle('#00FF41', 'fill', 1, title="上升")
downCandle = S.candle('#FF3333', 'fill', 1, title="下降")


smoothedOpen = F.call(haSmoothMaType,dataList,haSmootLength,'open')
smoothedHigh = F.call(haSmoothMaType,dataList,haSmootLength,'high')
smoothedLow = F.call(haSmoothMaType,dataList,haSmootLength,'low')
smoothedClose = F.call(haSmoothMaType,dataList,haSmootLength,'close')

heikenAshiData = []
prevHaOpen = null
prevHaClose = null
dataList.forEach((kData,index)=>{
    let _smoothedOpen = smoothedOpen[index]
    let _smoothedHigh = smoothedHigh[index]
    let _smoothedLow = smoothedLow[index]
    let _smoothedClose = smoothedClose[index]
    
    if(!_smoothedOpen || !_smoothedHigh || !_smoothedLow || !_smoothedClose){
        heikenAshiData.push({ open: null, high: null, low: null, close: null })
    }else{
        let haClose = (_smoothedOpen + _smoothedHigh + _smoothedLow + _smoothedClose) / 4
        let haOpen = prevHaOpen !== null && prevHaClose !== null ? (prevHaOpen + prevHaClose) / 2  : (_smoothedOpen + _smoothedClose) / 2
        let haHigh = Math.max(_smoothedHigh, haOpen, haClose)
        let haLow = Math.min(_smoothedLow, haOpen, haClose)
        heikenAshiData.push({open: haOpen,high: haHigh,low: haLow,close: haClose})
        prevHaOpen = haOpen
        prevHaClose = haClose
    }
    
})
finalOpen = F.call(haAfterSmoothMaType,heikenAshiData,haAfterSmoothLength,'open')
finalHigh = F.call(haAfterSmoothMaType,heikenAshiData,haAfterSmoothLength,'high')
finalLow = F.call(haAfterSmoothMaType,heikenAshiData,haAfterSmoothLength,'low')
finalClose = F.call(haAfterSmoothMaType,heikenAshiData,haAfterSmoothLength,'close')
smoothedHeikenAshi = []
dataList.forEach((kData,index)=>{
    let open = finalOpen[index]
    let high = finalHigh[index]
    let low = finalLow[index]
    let close = finalClose[index]
    if(open && high && low && close){
        smoothedHeikenAshi.push({open,high,low,close})
    }else{
        smoothedHeikenAshi.push(kData)
    }
})

D.candle(smoothedHeikenAshi, ({value})=>{
    return value.close < value.open ? downCandle : upCandle
})
O.tools('open',F.attr(smoothedHeikenAshi,'open'),upCandle)
O.tools('high',F.attr(smoothedHeikenAshi,'high'),upCandle)
O.tools('low',F.attr(smoothedHeikenAshi,'low'),upCandle)
O.tools('close',F.attr(smoothedHeikenAshi,'close'),upCandle)
setPrecision('price')
```

```
//@name=GFO
//@title=GainLab Fluctuating orbit
//@desc=## GainLab Fluctuating orbit 波动轨
//GainLab 波动轨是在Keltner Channel（肯特纳通道）的基础上，增加了更精确的参数与计算方式，提供了更精确的支撑阻力判断。
//### 本指标集成了多种功能：
//1. 双重Keltner通道：内外两层通道，提供更精确的支撑阻力判断
//2. 动态支撑阻力：自动识别并标记关键支撑阻力位
//3. WaveTrend信号：集成波浪趋势指标，提供买卖信号
//4. 多种移动平均：支持SMA、EMA、WMA、DEMA、TEMA、TRIMA、KAMA、MAMA、T3
//#### 应用法则：
//- 价格突破上轨可能表示上涨趋势，突破下轨可能表示下跌趋势。
//- 价格在通道内震荡时，上下轨可作为支撑阻力参考。
//- 通道收窄往往预示着即将出现较大的价格变动。
//- 红色圆点标记阻力位，黑色圆点标记支撑位。
//- 绿色三角形向上表示买入信号，红色三角形向下表示卖出信号。
//@vip=1
//@position=main
//@version=1

source = I.select('close', title="数据源", tip='用来计算的数据', options=SOURCE)
maLength = I.int(34, title="MA周期", min=1)
maType = I.select('MA',title='MA类型',options=['MA','SMA','EMA', 'WMA', 'DEMA', 'TEMA', 'TRIMA', 'KAMA', 'MAMA', 'T3'])
atrMultiplierMin = I.float(1.5,title='MIN',tip='ATR倍数最小值')
atrMultiplierMax = I.float(2.5,title='MAX',tip='ATR倍数最大值')
atrType = I.select('EMA',title='ATR平滑类型',options=['MA', 'EMA', 'WMA', 'DEMA', 'TEMA', 'TRIMA', 'KAMA', 'MAMA', 'T3'])
atrLength = I.int(88,title='ATR平滑长度')
keltnerLength = I.int(34,title='Keltner长度')
keltnerMultiplier = I.float(2.0,title='Keltner倍数')
overbought = I.int(1,title='超买阈值')
oversold = I.int(0,title='超卖阈值')
wtChannelLen = I.int(10,title='WT通道长度')
wtMALen = I.int(3,title='WT移动平均长度')
wtSmoothLen = I.int(3,title='WT平滑长度')
wtOverbought = I.int(90,title='WT超买阈值')
wtOversold = I.int(-90,title='WT超卖阈值')
upperMax =  S.line('rgb(249, 87, 249)',1,'dashed',title='上轨最大')
upper = S.line('rgb(249, 87, 249)',1,'dashed',title='上轨最小')
middle = S.line('#FF9900',1,'dashed',title='中轨')
lower = S.line('rgb(249, 87, 249)',1,'dashed',title='下轨最小')
lowerMax = S.line('rgb(249, 87, 249)',1,'dashed',title='下轨最大')
upperChannelFill = S.area('rgba(0, 255, 0, 0.1)',false, title="上轨通道填充")
lowerChannelFill = S.area('rgba(0, 255, 0, 0.1)',false, title="下轨通道填充")
resistance = S.shape('circle','#F00',4,'fill',title='阻力位')
support = S.shape('circle','#888',4,'fill',title='支撑位')
buySignal = S.shape('arrowUp','#00FF00',12,'fill','lowDown',title='买入信号')
sellSignal = S.shape('arrowDown','#FF0000',12,'fill','highUp',title='卖出信号')
sourceValues = F.attr(dataList,source)
hlcValues = F.attr(dataList,'hlc3')

middleValues = F.call(maType,sourceValues,maLength)
trValues = F.tr(dataList)
atrValues = F.call(atrType,trValues,atrLength)
const stdevValues = sourceValues.map((_, i) => {
    if (i < keltnerLength) return 0
    const slice = sourceValues.slice(i - keltnerLength + 1, i + 1)
    const mean = slice.reduce((a, b) => a + b) / slice.length
    const variance = slice.reduce((a, b) => a + Math.pow(b - mean, 2), 0) / slice.length
    return Math.sqrt(variance)
})
const wtMovingAverage = F.ema(hlcValues, wtChannelLen)
const wtMAChannel = F.ema(
    hlcValues.map((v, i) => Math.abs(v - wtMovingAverage[i])),
    wtChannelLen
)
const wtChannelIndex = hlcValues.map((v, i) =>
    wtMAChannel[i] !== 0 ? (v - wtMovingAverage[i]) / (0.015 * wtMAChannel[i]) : 0
)
const wtFast = F.ema(wtChannelIndex, wtMALen)
const wtSlow = F.ma(wtFast, wtSmoothLen)
const upperMaxData = []
const upperData = []
const middleData = []
const lowerData = []
const lowerMaxData=[]
const resistanceData = []
const supportData = []
const buySignalData = []
const sellSignalData = []
let pintarojo = 0.0
let pintaverde = 0.0
dataList.forEach((kData,i)=>{
    if (i >= Math.max(maLength, atrLength, keltnerLength)) {
        const _middleValue = middleValues[i]
        const _atrValue = atrValues[i]
        middleData.push(_middleValue)
        upperData.push(_middleValue + _atrValue * atrMultiplierMin)
        lowerData.push(_middleValue - _atrValue * atrMultiplierMin)
        upperMaxData.push(_middleValue + _atrValue * atrMultiplierMax)
        lowerMaxData.push(_middleValue - _atrValue * atrMultiplierMax)
        const dev = keltnerMultiplier * stdevValues[i]
        const upper = _middleValue + dev
        const lower = _middleValue - dev
        const bbr = (upper - lower) !== 0 ? (sourceValues[i] - lower) / (upper - lower) : 0
        if (i > 0) {
            const prevBasis = middleValues[i - 1]
            const prevDev = keltnerMultiplier * stdevValues[i - 1]
            const prevUpper = prevBasis + prevDev
            const prevLower = prevBasis - prevDev
            const prevBBR = (prevUpper - prevLower) !== 0 ? (sourceValues[i - 1] - prevLower) / (prevUpper - prevLower) : 0
            if (prevBBR > overbought && bbr < overbought) {
                pintarojo = dataList[i - 1].high
            }			
            if (prevBBR < oversold && bbr > oversold) {
                pintaverde = dataList[i - 1].low
            }
            const prevPintarojo = resistanceData[i-1] || 0
            const prevPintaverde = supportData[i-1] || 0
            buySignalData.push(prevPintaverde > 0 && pintaverde !== prevPintaverde ? true : null)
            sellSignalData.push(prevPintarojo > 0 && pintarojo !== prevPintarojo ? true : null)
            resistanceData.push(pintarojo > 0 ? pintarojo : null)
            supportData.push(pintaverde > 0 ? pintaverde : null)
        }else{
            resistanceData.push(null)
            supportData.push(null)
            buySignalData.push(null)
            sellSignalData.push(null)
        }
    }else{
        middleData.push(null)
        upperData.push(null)
        lowerData.push(null)
        upperMaxData.push(null)
        lowerMaxData.push(null)
        resistanceData.push(null)
        supportData.push(null)
        buySignalData.push(null)
        sellSignalData.push(null)
    }
})
D.area(upperMaxData,upperData,upperChannelFill)
D.area(lowerMaxData,lowerData,lowerChannelFill)
D.line(upperMaxData,upperMax)
D.line(upperData,upper)
D.line(middleData,middle)
D.line(lowerData,lower)
D.line(lowerMaxData,lowerMax)
D.shape(resistanceData,resistance)
D.shape(supportData,support)
D.shape(buySignalData,buySignal)
D.shape(sellSignalData,sellSignal)
```

```
//@name=PVT
//@title=Price Volume Trend 价量趋势
//@desc=PVT（Price Volume Trend）价量趋势指标，是一种将价格变化幅度与成交量相结合的累积型技术指标，用于分析价格趋势的强度和可持续性。
//#### 应用法则：
//- 当PVT持续上升时，表示价格上涨得到成交量的有效支撑，上涨趋势较为健康；当PVT持续下降时，表示价格下跌伴随放量，下跌压力较大。
//- PVT与价格走势的背离是重要信号：价格创新高而PVT未创新高时，出现顶背离，预示上涨动能减弱；价格创新低而PVT未创新低时，出现底背离，预示下跌动能减弱。
//- PVT穿越零轴具有重要意义：向上突破零轴表示多头力量占优；向下跌破零轴表示空头力量占优。
//- 结合平滑线使用：当PVT上穿平滑线时为买入信号，当PVT下穿平滑线时为卖出信号。
//#### 与OBV指标对比：
//- OBV只考虑价格方向（上涨+1，下跌-1），而PVT考虑价格变化的具体幅度，因此PVT对价格变化更加敏感，能更好地反映趋势的强度变化。
//- PVT指标特别适用于中长期趋势分析，当PVT与价格同步上升且成交量放大时，往往预示着强势趋势的延续。
//- 在使用PVT时需要注意价量配合的质量，单纯的价格变化而无成交量支撑的走势往往不可持续，有效的PVT信号应当伴随合理的成交量变化。
//@position=vice
//@version=1
smoothType = I.select('None', title="平滑类型", options=['None', 'SMA', 'EMA', 'WMA'])
smoothPeriod = I.int(14, title="平滑周期", min=1)
pvtStyle = S.line('#FF6D00',1, 'solid', title="PVT") 
maStyle = S.line('#2196F3',1, 'solid', title="平滑线") 
let sum = 0
let pvt = dataList.map((kData,i)=>{
        if(i === 0){
        return sum
        }
        let prev = dataList[i - 1]
        if(!prev.close || prev.close  === 0){
        return sum
        }
        const rate = (kData.close - prev.close) / prev.close
        let x = rate * kData.volume
        sum += x
        return sum
})
let ma = []
D.line(pvt,pvtStyle)
setPrecision(4)
O.tools('PVT',pvt,pvtStyle) 
if(smoothType !== 'None'){
    ma = F.call(smoothType,pvt,smoothPeriod)
    D.line(ma,maStyle)
    O.tools('PVTMA',ma,maStyle) 
}
```

```
//@name=QQE_MOD
//@title=Quantitative Qualitative Estimation Modified
//@desc=QQE MOD指标是一个基于RSI的高级技术分析工具，结合了布林带和双QQE系统。
//QQE指标通过平滑RSI并应用动态ATR带来减少噪音，提供更可靠的交易信号。
//#### 应用法则：
//- 主QQE线：基于RSI的平滑趋势线，用于判断主要趋势方向。
//- 次QQE线：用于信号确认，减少假信号。
//- 次QQE线：用于信号确认，减少假信号。
//- 买入信号：次RSI > 阈值 且 主RSI > 布林带上轨。
//- 号：次RSI < -阈值 且 主RSI < 布林带下轨。
//- 零线：50是多空分界线，上方为多头市场，下方为空头市场。
//@position=vice
//@version=1.0
rsiLengthPrimary = I.int(6, title='主RSI周期', min=1, max=50, tip='主要RSI计算周期')
rsiSmoothingPrimary = I.int(5, title='主RSI平滑', min=1, max=20, tip='主要RSI平滑周期')
qqeFactorPrimary = I.float(3.0, title='主QQE因子', min=0.1, max=10, step=0.1, tip='主要QQE系统的因子')
rsiLengthSecondary = I.int(6, title='次RSI周期', min=1, max=50, tip='次要RSI计算周期')
rsiSmoothingSecondary = I.int(5, title='次RSI平滑', min=1, max=20, tip='次要RSI平滑周期')
qqeFactorSecondary = I.float(1.61, title='次QQE因子', min=0.1, max=10, step=0.01, tip='次要QQE系统的因子')
threshold = I.float(3.0, title='信号阈值', min=0.1, max=10, step=0.1, tip='次要系统的信号阈值')
bollingerLength = I.int(50, title='布林带周期', min=10, max=200, tip='布林带计算周期')
bollingerMultiplier = I.float(0.35, title='布林带倍数', min=0.1, max=2, step=0.05, tip='布林带宽度倍数')
primaryLineStyle = S.line('#FFFFFF', 2, solid, title='主QQE线')
secondaryLineStyle = S.line('#FF9900', 2, solid,false, title='次QQE线')
bollingerUpperStyle = S.line('#888888',1,'dashed',false,title='布林带上轨')
bollingerLowerStyle = S.line('#888888',1,'dashed',false,title='布林带下轨')
zeroLineStyle = S.line('#ffffff', 1, dotted, title='零轴线')
upSignalStyle = S.bar('#00c3ff', fill, title='看涨信号')
downSignalStyle = S.bar('#ff0062', fill, title='看跌信号')
neutralStyle = S.bar('#707070', fill, title='中性区域')
function calculateRSI(dataList, period) {
    const rsiValues = []

    for (let i = 0; i < dataList.length; i++) {
        if (i < period) {
            rsiValues.push(50)
            continue
        }

        let gains = 0
        let losses = 0

        for (let j = i - period + 1; j <= i; j++) {
            const change = dataList[j].close - dataList[j - 1].close
            if (change > 0) {
                gains += change
            } else {
                losses += Math.abs(change)
            }
        }

        const avgGain = gains / period
        const avgLoss = losses / period

        if (avgLoss === 0) {
            rsiValues.push(100)
        } else {
            const rs = avgGain / avgLoss
            const rsi = 100 - (100 / (1 + rs))
            rsiValues.push(rsi)
        }
    }

    return rsiValues
}

// 计算EMA的函数  
function calculateEMA(values, period) {
    const ema = []
    const multiplier = 2 / (period + 1)

    for (let i = 0; i < values.length; i++) {
        if (i === 0) {
            ema.push(values[i])
        } else {
            ema.push((values[i] * multiplier) + (ema[i - 1] * (1 - multiplier)))
        }
    }

    return ema
}
function calculateQQE(dataList, rsiLength, smoothingFactor, qqeFactor) {
    const wildersLength = rsiLength * 2 - 1
    const rsiValues = calculateRSI(dataList, rsiLength)
    const smoothedRsi = calculateEMA(rsiValues, smoothingFactor)
    const atrRsi = []
    for (let i = 0; i < smoothedRsi.length; i++) {
        if (i === 0) {
            atrRsi.push(0)
        } else {
            atrRsi.push(Math.abs(smoothedRsi[i] - smoothedRsi[i - 1]))
        }
    }
    const smoothedAtrRsi = calculateEMA(atrRsi, wildersLength)
    const qqeTrendLine = []
    const longBand = []
    const shortBand = []
    let trendDirection = []
    for (let i = 0; i < smoothedRsi.length; i++) {
        const dynamicAtrRsi = smoothedAtrRsi[i] * qqeFactor
        const newLongBand = smoothedRsi[i] - dynamicAtrRsi
        const newShortBand = smoothedRsi[i] + dynamicAtrRsi
        if (i === 0) {
            longBand.push(newLongBand)
            shortBand.push(newShortBand)
            trendDirection.push(1)
        } else {
            if (smoothedRsi[i - 1] > longBand[i - 1] && smoothedRsi[i] > longBand[i - 1]) {
                longBand.push(Math.max(longBand[i - 1], newLongBand))
            } else {
                longBand.push(newLongBand)
            }
            if (smoothedRsi[i - 1] < shortBand[i - 1] && smoothedRsi[i] < shortBand[i - 1]) {
                shortBand.push(Math.min(shortBand[i - 1], newShortBand))
            } else {
                shortBand.push(newShortBand)
            }
            if (smoothedRsi[i] > shortBand[i - 1] && smoothedRsi[i - 1] <= shortBand[i - 1]) {
                trendDirection.push(1)
            } else if (smoothedRsi[i] < longBand[i - 1] && smoothedRsi[i - 1] >= longBand[i - 1]) {
                trendDirection.push(-1)
            } else {
                trendDirection.push(trendDirection[i - 1])
            }
        } 
        qqeTrendLine.push(trendDirection[i] === 1 ? longBand[i] : shortBand[i])
    }
    return {
        qqeTrendLine,
        smoothedRsi
    }
}
const primaryQQE = calculateQQE(dataList, rsiLengthPrimary, rsiSmoothingPrimary, qqeFactorPrimary)
const secondaryQQE = calculateQQE(dataList, rsiLengthSecondary, rsiSmoothingSecondary, qqeFactorSecondary)
const bollingerBasis = []
const bollingerDeviation = []
const bollingerUpper = []
const bollingerLower = []
dataList.forEach((kData,i)=>{
    if (i < bollingerLength - 1) {
        bollingerBasis.push(0)
        bollingerDeviation.push(0)
        bollingerUpper.push(0)
        bollingerLower.push(0)
    }else{
        let sum = 0
        for (let j = i - bollingerLength + 1; j <= i; j++) {
            sum += (primaryQQE.qqeTrendLine[j] - 50)
        }
        const basis = sum / bollingerLength
        bollingerBasis.push(basis)
        let variance = 0
        for (let j = i - bollingerLength + 1; j <= i; j++) {
            variance += Math.pow((primaryQQE.qqeTrendLine[j] - 50) - basis, 2)
        }
        const stdev = Math.sqrt(variance / bollingerLength)
        const deviation = bollingerMultiplier * stdev
        bollingerDeviation.push(deviation)
        bollingerUpper.push(basis + deviation)
        bollingerLower.push(basis - deviation)
    }
})
let primaryQQETrendLine = []
let primaryRSIArr = []
let secondaryQQETrendLine = []
let secondaryRSIArr = []
let upSignal = []
let downSignal = []
let noSignal = []
dataList.forEach((kData,i)=>{
    let _primaryRSI = primaryQQE.smoothedRsi[i] - 50
    let _secondaryRSI = secondaryQQE.smoothedRsi[i] - 50
    let _primaryQQETrendLine = primaryQQE.qqeTrendLine[i] - 50
    let _secondaryQQETrendLine = secondaryQQE.qqeTrendLine[i] - 50
    let _upSignal = null
    let _downSignal = null
    let _noSignal = null
    if (_secondaryRSI > threshold && _primaryRSI > bollingerUpper[i]) {
        _upSignal = _secondaryRSI
    }else	if (_secondaryRSI < -threshold && _primaryRSI < bollingerLower[i]) {
        _downSignal = _secondaryRSI
    }else{
        _noSignal = _secondaryRSI
    }
    primaryQQETrendLine.push(_primaryQQETrendLine)
    primaryRSIArr.push(_primaryRSI)
    secondaryQQETrendLine.push(_secondaryQQETrendLine)
    secondaryRSIArr.push(_secondaryRSI)
    upSignal.push(_upSignal)
    downSignal.push(_downSignal)
    noSignal.push(_noSignal)
})
D.hline(0, zeroLineStyle)
D.bar(upSignal, 0, upSignalStyle)
D.bar(downSignal, 0, downSignalStyle)
D.bar(noSignal, 0, neutralStyle)
D.line(primaryQQETrendLine,primaryLineStyle)
D.line(secondaryQQETrendLine, secondaryLineStyle)
D.line(bollingerUpper, bollingerUpperStyle)
D.line(bollingerLower, bollingerLowerStyle)
O.tools('主QQE', primaryQQETrendLine, primaryLineStyle)
O.tools('次QQE', secondaryQQETrendLine, secondaryLineStyle)
O.tools('布林上轨', bollingerUpper, bollingerUpperStyle)
O.tools('布林下轨', bollingerLower, bollingerLowerStyle)
O.tools('主RSI', primaryRSIArr, primaryLineStyle)
O.tools('次RSI', secondaryRSIArr,secondaryLineStyle)

// 设置精度
setPrecision(2)
```

```
//@name=TD
//@title=Tom DeMark Sequential
//@desc=TD Sequential（TD序列指标）是由著名技术分析师汤姆·德马克开发的技术分析工具，通过严格的计数规则识别市场转折点。
//#### 应用法则：
//- 当买入设置完成（标记为绿色数字9）时，表示下跌趋势可能结束，是潜在的买入信号。
//- 当卖出设置完成（标记为红色数字9）时，表示上涨趋势可能结束，是潜在的卖出信号。
//- TD倒计时：如果设置完成后市场未反转，则启动倒计时阶段，寻找13个符合条件的交易日，完成后产生更强的反转信号。
//- 信号确认：第9根K线收盘价应突破第6根或第7根K线的高低点，增加信号可靠性。
//- 风险控制：买入信号的止损位设在第9根K线最低价下方，卖出信号的止损位设在第9根K线最高价上方。
//- 适用范围：TD指标适用于所有时间框架和金融品种，在日线图表中表现最佳，特别适合趋势市场中寻找反转机会。
//- 使用要点：必须严格按照规则计数，不能中断；建议结合其他技术指标确认；注意资金管理和风险控制。
//@position=main
//@version=1
period = I.int(4, title="周期", min=1)
buySignal = S.shape('triangleUp','#4CAF50',8,'fill','lowDown',title='多箭头')
sellSignal = S.shape('triangleDown','#F44336',8,'fill','highUp',title='卖箭头')
textBuyStyle = S.label('#fff', 12, 'center','lowDown', title='数字多')
textBuyBgStyle = S.labelbg('#4CAF50','fill',title='数字背景多')
textSellStyle = S.label('#fff', 12, 'center','highUp', title='数字空')
textSellBgStyle = S.labelbg('#F44336','fill',title='数字背景空')
let tdState = {type:'',count:0}
const tdArr = dataList.map((item,index)=>{
    let td = {}
    if (index >= period){
        const oclose = dataList[index - period].close
        if (item.close > oclose) {
            if (tdState.type === 'up') {
                tdState.count++
            } else {
                tdState.type = 'up'
                tdState.count = 1
            }
        } else if (item.close < oclose) {
            if (tdState.type === 'down') {
                tdState.count++
            } else {
                tdState.type = 'down'
                tdState.count = 1
            }
        } else {
            tdState.type = ''
            tdState.count = 0
        }
        if (tdState.type !== '') {
            td.type = tdState.type
            td.count = tdState.count
        }
    }
    return td
})
let buyData = []
let sellData = []
tdArr.forEach(item=>{
    if(item && item.count && (item.count === 9 || item.count  === 13)){
        if(item.type === 'up'){
            sellData.push({text:item.count})
            buyData.push(null)
        }else{
            sellData.push(null)
            buyData.push({text:item.count})
        }
    }else{
        buyData.push(null)
        sellData.push(null)
    }
})
D.shape(buyData,buySignal)
D.shape(sellData,sellSignal)
textBuyStyle.y = 8
textSellStyle.y = -6
textBuyBgStyle.padding = 8
textSellBgStyle.padding = 8
D.label(buyData,textBuyStyle,textBuyBgStyle)
D.label(sellData,textSellStyle,textSellBgStyle)
```

```
//@name=BOLL
//@title=Bollinger Bands 布林线
//@desc=BOLL(Bollinger Bands)通过计算价格的"标准差"，再求价格的"信赖区间"，是一个路径型指标
// 该指标在图形上画出三条线，上下两条线可以分别看成是价格的压力线和支撑线，中间为价格平均线。
// 价格波动在上限和下限的区间之内，价格涨跌幅度加大时，带状区会变宽，涨跌幅度狭小盘整时，带状区会变窄。价超越上限时，代表超买，价格超越下限时，代表超卖。
//#### 应用法则：
//- 当布林线的带状区呈水平方向移动时，可以视为处于"常态范围"。
//- 此时，价格向上穿越"上限"时，将形成短期回档，为短线的卖出信号；价格向下穿越"下限"时，将形成短期反弹，为短线的买进时机。
//- 当带状区朝右上方和右下方移动时，则属于脱离常态。
//- 当价格连续穿越"上限"，暗示价格将朝上涨方向前进；当价格连续穿越"下限"，暗示价格将朝下跌方向前进。
//@position=main
//@version=1
setPrecision('price')
n = I.int(20, title="N", tip="中轨线周期")
k = I.int(2, title="K", tip="标准差倍数")
ubStyle = S.line('rgb(255, 0, 255)', 1, 'solid', title="上轨")
midStyle = S.line('rgb(30, 144, 255)', 1, 'solid', title="中轨")
lbStyle = S.line('rgb(255, 0, 255)', 1, 'solid', title="下轨")
fullStyle = S.area('rgba(255, 0, 255, 0.1)',false, title="填充")
const boll = F.boll(dataList, n, k)
const ubData = F.attr(boll,'ub')
const midData = F.attr(boll,'mid')
const lbData = F.attr(boll,'lb')
D.area(ubData,lbData,fullStyle)
D.line(ubData, ubStyle)
D.line(midData, midStyle)
D.line(lbData, lbStyle)
O.tools('UB', ubData, ubStyle)
O.tools('MID', midData, midStyle)
O.tools('LB', lbData, lbStyle)
```

```
//@name=MACD
//@title=指数平滑异同移动平均线
//@desc=## MACD 指数平滑异同移动平均线
//MACD（Moving Average Convergence and Divergence）属于大势趋势类指标
//它由长期均线DEA、短期的DIF、能量柱、0轴（多空分界线）组成，可以去除掉移动平均线频繁的假讯号缺陷又能确保移动平均线最大的战果。
//### 应用法则
//1. MACD在0界限以上为多头市场，反之为空头市场。
//2. DIF、DEA均为正，DIF向上突破DEA，买入信号。
//3. DIF、DEA均为负，DIF向下跌破DEA，卖出信号。
//4. 分析MACD柱状线，由红变绿(正变负)，卖出信号；由绿变红，买入信号。
//5. DEA线与K线发生背离，行情反转信号。
//6. 日线、周线、月线、分钟线配合运用效果会更好。
//@position=vice
//@version=1
// 定义可设置参数
n = I.int(12, title="快线周期", tip="计算的快线周期", min=1)
k = I.int(26, title="慢线周期", tip="计算的慢线周期", min=1)
m = I.int(9, title="DEA周期", tip="计算的DEA周期", min=1)
// 定义可设置样式
difStyle = S.line('#FF9900', 1, 'solid', title="DIF样式")
deaStyle = S.line('#0066CC', 1, 'solid', title="DEA样式")
zeroStyle = S.line('#999', 1, 'solid', title="0轴样式")
buyMacdUp=S.bar('#2DC08E','stroke',title='多头增加')
buyMacdDown=S.bar('#2DC08E','fill',title='多头减少')
sellMacdUp=S.bar('#F92855','stroke',title='空头增加')
sellMacdDown=S.bar('#F92855','fill',title='空头减少')
// 计算数据
macdResult = F.macd(dataList, n, k, m)
difData = F.attr(macdResult,'dif')
deaData = F.attr(macdResult,'dea')
macdData = F.attr(macdResult,'macd')
// 绘制0轴
D.hline(0, zeroStyle)
// 绘制MACD柱状图（动态颜色）
D.bar(macdData, 0, ({value, prev}) => {
    if (value >= 0) {
    // 多头
    return prev < value ? buyMacdUp : buyMacdDown
    } else {
    // 空头
    return prev < value ? sellMacdUp : sellMacdDown
    }
})
// 绘制DIF线
D.line(difData, difStyle)
// 绘制DEA线
D.line(deaData, deaStyle)
// 设置精度
setPrecision(4)
// 输出工具栏提示数据
O.tools('DIF',difData,difStyle)
O.tools('DEA',deaData,deaStyle)
O.tools('MACD',macdData,{color: '#666666'})
```

```
//@name=MTM
//@title=Momentum 动量指标
//@desc=MTM（Momentum）动量指标，又称动力指标，是通过比较当前价格与N日前价格的差值来衡量价格变动速度和力度的技术分析工具。
//#### 应用法则：
//- MTM指标以0为中轴线，当MTM大于0时表示价格上涨动量，市场偏向多头；当MTM小于0时表示价格下跌动量，市场偏向空头。
//- 当MTM从底部向上突破0轴或其移动平均线时，为买入信号；当MTM从顶部向下跌破0轴或其移动平均线时，为卖出信号。
//- 观察MTM的数值变化趋势，当MTM数值开始缩小时，表明当前趋势的动量正在减弱，可能面临趋势转折。
//- 当价格创新高而MTM指标没有创新高时，形成顶背离，往往是趋势转折的信号；当价格创新低而MTM指标没有创新低时，形成底背离。
//- MTM指标对价格变动非常敏感，适合用于捕捉短期价格动向和确认趋势强度，与其移动平均线配合使用可以过滤假信号。
//- 建议结合成交量指标观察，当MTM向上突破且成交量放大时，买入信号更加可靠。
//- 不同周期的MTM可以反映不同时间框架的动量变化，短周期MTM更敏感，长周期MTM更稳定。
//@position=vice
//@version=1
period = I.int(12, title="MTM周期", min=1)
maPeriod = I.int(6, title="MTM均线周期", min=1)
mtmStyle = S.line('#FF6600',1, 'solid', title="MTM") 
maStyle = S.line('#00CC99',1, 'solid', title="MTMMA") 
let mtm = F.mtm(dataList, maPeriod)
let ma = F.ma(mtm, period)
D.line(mtm,mtmStyle)
D.line(ma,maStyle)
setPrecision(4)
O.tools('MTM',mtm,mtmStyle) 
O.tools('MTMMA',ma,maStyle)
```

```
//@name=SAR
//@title=SAR 停损点转向指标
//@desc=## SAR 停损点转向指标
//SAR(Stop and Reverse)是应买进或抛售价位的转向点，此种技术分析工具与移动平均线原理颇为相似，属于价格与时间并重的分析工具。
//由于组成该线的点以弧形的方式移动，故称抛物转向。
//### 应用法则：
//- 价格曲线在SAR曲线之上时，为多头市场。
//- 价格曲线在SAR曲线之下时，为空头市场。
//- 价格曲线由上向下跌破SAR曲线时，为卖出信号并应同时放空。
//- 价格曲线由下向上穿破SAR曲线时，为买入信号并应同时补空。
//- 专门使用SAR操作时，不必参考其他指标，放弃自己人性的研判，采取突破买进与跌破卖出的手法。
//- SAR不求买在最低价，  卖在最高价，却注重一段时期内的绝对获利率。
//- 注意，SAR 在盘整时失效。
//@position=main
//@version=1

startAf = I.int(2,title='开始') 
step = I.int(2,title='增量') 
maxAf = I.int(20,title='极限') 
upStyle = S.shape('circle','#339933',6,title='多方')
downStyle = S.shape('circle','#FF0033',6,title='空方')

sar = F.sar(dataList,startAf,step,maxAf)
D.shape(sar,({value,index})=>{
    return (value < (dataList[index].high + dataList[index].low) / 2) ? downStyle : upStyle
})
O.tools('SAR',sar,upStyle)
setPrecision('price')
```

```
//@name=VOL
//@title=VOLUME 成交量
//@desc = ## VOLUME 成交量
//成交量是指在某一时段内具体的交易数，将一定时期内的成交量相加后平均，在成交量的柱条图中形成较为平滑的曲线，即均量线。
//均量线反映的仅是市场交投的主要趋向，对未来价格变动的大势起着辅助指标的作用。
//#### 应用法则：
//- 价格随成交量的递增而上涨，为市场行情的正常特性，此种量增价涨的关系，表示价格将继续上升。
//- 价格下跌，向下跌破价格形态、趋势线、移动平均线，同时出现大成交量是价格将深幅下跌的信号，强调趋势的反转。
//- 价格随着缓慢递增的成交量而逐渐上涨，渐渐的走势突然成为垂直上升的爆发行情，成交量急剧增加，价格爆涨，紧接着，成交量大幅萎缩，价格急剧下跌，表示涨势已到末期，有转势可能。
//- 温和放量。成交量在前期持续低迷之后，出现连续温和放量形态，一般可以证明有实力资金在介入。但这并不意味着投资者就可以马上介入，在底部出现温和放量之后，价格会随量上升，量缩时价格会适量调整。当持续一段时间后，价格的上涨会逐步加快。
//- 突放巨量。这其中可能存在多种情况，如果价格经历了较长时间的上涨过程后放巨量，通常表明多空分歧加大，有实力资金开始派发，后市继续上涨将面临一定困难。
//- 而经历了深幅下跌后的巨量一般多为空方力量的最后一次集中释放，后市继续深跌的可能性很小。
//- 如果市场整体下跌，而某个币种逆势放量，在市场一片喊空声之时放量上攻，往往持续时间不长，随后反而加速下跌。
//- 成交量也有形态，当成交量构筑圆弧底，而价格也形成圆弧底时，往往表明后市将出现较大上涨机会。
//@position=vice
//@version=1

period1 = I.int(5, title="平均线周期1", min=1) 
period2 = I.int(10, title="平均线周期2", min=1)
period3 = I.int(20, title="平均线周期3", min=1) 
maStyle1 = S.line('#F00',1, 'solid', title="MA1样式") 
maStyle2 = S.line('#FF0',1, 'solid', title="MA2样式") 
maStyle3 = S.line('#0FF',1, 'solid',false, title="MA3样式")
buyStyle = S.bar('#2DC08E', 'stroke', title="多柱")
sellStyle = S.bar('#F92855', 'fill', title="空柱")

volumeData = F.attr(dataList, 'volume')
ma1 = F.ma(volumeData, period1)
ma2 = F.ma(volumeData, period2)
ma3 = F.ma(volumeData, period3)


D.bar(volumeData,0, ({value,prev})=>{
    return value > prev ? buyStyle : sellStyle
})
D.line(ma1,maStyle1)
D.line(ma2,maStyle2)
D.line(ma3,maStyle3)
O.tools('MA'+period1,ma1,maStyle1) 
O.tools('MA'+period2,ma2,maStyle2) 
O.tools('MA'+period3,ma3,maStyle3) 
O.tools('VOL',volumeData,buyStyle) 
setPrecision('volume')
setMin(0)
```

```
//@name=ZigZag
//@title=ZigZag指标
//@desc=ZigZag 指标用于识别价格重要转折点。
//Depth: 寻找极值的深度
//Deviation: 最小偏差点数
//Backstep: 回退步数。
//@position=main
//@version=1

// 设置精度
setPrecision('price')

// 输入参数
depth = I.int(12, title="Depth", tip="寻找极值的深度")
deviation = I.int(5, title="Deviation", tip="最小偏差点数")
backstep = I.int(2, title="Backstep", tip="回退步数")

buyStyle = S.line('#00FF00', 2, 'solid', true, title="多头样式")
sellStyle = S.line('#FF0000', 2, 'solid', true, title="空头样式")
buyStyle.continuous = true
sellStyle.continuous = true
function calculateZigZag(dataList, depth, deviation, backstep) {
    const len = dataList.length
    const zigzagBuffer = new Array(len).fill(0)
    const highBuffer = new Array(len).fill(0)
    const lowBuffer = new Array(len).fill(0)
    
    for (let i = depth; i < len - depth; i++) {
    const currentHigh = dataList[i].high
    let isHighPoint = true
    for (let j = i - depth; j <= i + depth; j++) {
        if (j !== i && dataList[j].high >= currentHigh) {
        isHighPoint = false
        break
        }
    }
    if (isHighPoint) highBuffer[i] = currentHigh
    
    const currentLow = dataList[i].low
    let isLowPoint = true
    for (let j = i - depth; j <= i + depth; j++) {
        if (j !== i && dataList[j].low <= currentLow) {
        isLowPoint = false
        break
        }
    }
    if (isLowPoint) lowBuffer[i] = currentLow
    }
    let whatlookfor = 0, back = 0, pos = 0, val = 0, res = 0, extreme = 0
    let foundPoints = 0
    
    for (let i = 0; i < len; i++) {
    if (whatlookfor === 0) {
        if (highBuffer[i] > 0) {
        val = highBuffer[i]
        if (val === dataList[i].high) {
            zigzagBuffer[i] = dataList[i].high
            whatlookfor = -1
            pos = i
            extreme = val
            foundPoints++
        }
        }
        if (lowBuffer[i] > 0) {
        val = lowBuffer[i]
        if (val === dataList[i].low) {
            zigzagBuffer[i] = dataList[i].low
            whatlookfor = 1
            pos = i
            extreme = val
            foundPoints++
        }
        }
    } else if (whatlookfor === 1) {
        if (highBuffer[i] > 0) {
        val = highBuffer[i]
        if (val > extreme && val === dataList[i].high) {
            zigzagBuffer[i] = dataList[i].high
            whatlookfor = -1
            pos = i
            extreme = val
            foundPoints++
        }
        }
        if (lowBuffer[i] > 0) {
        val = lowBuffer[i]
        if (val < extreme && val === dataList[i].low) {
            back = pos
            pos = i
            extreme = val
            res = val
            if (i - back >= backstep) {
            zigzagBuffer[i] = dataList[i].low
            whatlookfor = 1
            foundPoints++
            } else {
            }
        }
        }
    } else if (whatlookfor === -1) {
        if (lowBuffer[i] > 0) {
        val = lowBuffer[i]
        if (val < extreme && val === dataList[i].low) {
            // 修复：不清除之前的点，直接设置新点
            zigzagBuffer[i] = dataList[i].low
            whatlookfor = 1
            pos = i
            extreme = val
            foundPoints++
        }
        }
        if (highBuffer[i] > 0) {
        val = highBuffer[i]
        if (val > extreme && val === dataList[i].high) {
            back = pos
            pos = i
            extreme = val
            res = val
            if (i - back >= backstep) {
            zigzagBuffer[i] = dataList[i].high
            whatlookfor = -1
            foundPoints++
            } else {
            }
        }
        }
    }
    }
    const result = zigzagBuffer.map(val => val !== 0 ? val : null)
    return result
}
zigzagData = calculateZigZag(dataList, depth, deviation, backstep)
D.line(zigzagData, ({ value, prevValid }) => {
    if (prevValid !== null) {
    if (value > prevValid) {
        return buyStyle
    } else if (value < prevValid) {
        return sellStyle 
    }
    }
    
    return buyStyle 
})
```

```
//@name=TB-Trend
//@title=TB-Trend 趋势指标
//@desc = ## TB-Trend 趋势指标
//基于快线和慢线的双重趋势带指标，通过计算最高价和最低价的简单移动平均线来构建趋势通道。
//#### 应用法则：
//- 价格突破快线上轨时，快线通道显示看涨信号
//- 价格跌破快线下轨时，快线通道显示看跌信号
//- 慢线通道提供主要趋势方向确认
//- 快慢线配合使用，可以识别趋势的强度和持续性
//@position=main
//@version=1

period1 = I.int(20, title="快线", min=1) 
period2 = I.int(180, title="慢线", min=1)
fastUpper = S.line('rgb(0, 195, 255)',1, 'solid', title="快线上轨") 
fastLower = S.line('hsl(194, 100%, 50%)',1, 'solid', title="快线下轨") 
slowUpper = S.line('rgb(255, 107, 53)',1, 'solid', title="慢线上轨") 
slowLower = S.line('#FF6B35',1, 'solid', title="慢线下轨") 
fastFill = S.area('rgba(0, 195, 255, 0.15)', title="快线通道") 
slowFill = S.area('rgba(255, 107, 53, 0.15)', title="慢线通道") 
fastHighMA = F.ma(dataList, period1, 'high')
fastLowMA = F.ma(dataList, period1, 'low')
slowHighMA = F.ma(dataList, period2, 'high')
slowLowMA = F.ma(dataList, period2, 'low')
D.area(fastHighMA,fastLowMA,fastFill)
D.area(slowHighMA,slowLowMA,slowFill)
D.line(fastHighMA,fastUpper)
D.line(fastLowMA,fastLower)
D.line(slowHighMA,slowUpper)
D.line(slowLowMA,slowLower)

O.tools('快线上轨',fastHighMA,fastUpper)
O.tools('快线下轨',fastLowMA,fastLower)
O.tools('慢线上轨',slowHighMA,slowUpper)
O.tools('慢线下轨',slowLowMA,slowLower)
setPrecision('price')
```

```
//@name=Donchian Channels
//@title=Donchian Channels 唐奇安通道
//@desc=## Donchian Channels 唐奇安通道
// 唐奇安通道（Donchian Channel）是由技术分析大师理查德·唐奇安（Richard Donchian）开发的一个经典技术指标，被誉为"趋势跟踪之父"的代表作。
//它是一个基于价格突破的趋势跟踪工具。
//#### 趋势识别
//- 价格在通道上方运行 → 上升趋势
//- 价格在通道下方运行 → 下降趋势
//- 价格在通道内波动 → 震荡行情
//#### 交易信号
//- 买入信号：价格突破上轨线
//- 卖出信号：价格跌破下轨线
//- 止损参考：以对向通道线作为止损位
//#### 支撑阻力
//- 上轨线作为压力位
//- 下轨线作为支撑位
//- 中轨线提供动态支撑或阻力
//@position=main
//@version=1

length = I.int(20,title='周期')
upperStyle = S.line('#2962FF',title='上通道')
basisStyle = S.line('#FF6D00',title='基线')
lowerStyle = S.line('#2962FF',title='下通道')
lower =  F.llv(dataList,length,'low')
upper =  F.hhv(dataList,length,'high')

basis =  upper.map((val,index)=>{
    let val2 = lower[index]
    if(U.isValidNumber(val) && U.isValidNumber(val2)){
    return (val + val2) / 2
    }else{
    return null
    }

})
D.line(upper,upperStyle)
D.line(basis,basisStyle)
D.line(lower,lowerStyle)
```

