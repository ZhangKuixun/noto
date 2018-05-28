
bolag:
https://blog.csdn.net/u014769864/article/details/72723180 放大后下标和背景一起移动
https://blog.csdn.net/u012123938/article/details/80344318 详细介绍
https://jitpack.io/com/github/PhilJay/MPAndroidChart/v3.0.2/javadoc/index.html?help-doc.html MPChartLib v3.0.2 API
https://realm.io/cn/docs/java/latest/   realm数据库

1. 点击屏幕上的某项的执行流程  有两种情况，一种是时间很短，一种时间稍长
时间很短：onDown--------》onSingleTapUp--------》onSingleTapConfirmed
时间稍长：onDown--------》onShowPress------》onSingleTapUp--------》onSingleTapConfirmed
2. 长按事件
onDown--------》onShowPress------》onLongPress
3.抛：手指触动屏幕后，稍微滑动后立即松开
onDown-----》onScroll----》onScroll----》onScroll----》………----->onFling
4.拖动
onDown------》onScroll----》onScroll------》onFiling
注意：有的时候会触发onFiling，但是有的时候不会触发，个人理解是人的动作不标准所致。


刷新
invalidate() : 在chart中调用会使其刷新重绘
notifyDataSetChanged() : 让chart知道它依赖的基础数据已经改变，并执行所有必要的重新计算（比如偏移量，legend，最大值，最小值 …）。在动态添加数据时需要用到。

打印日志
etLogEnabled(boolean enabled) : 设置为true将激活chart的logcat输出。但这不利于性能，如果不是必要的，应保持禁用。

setBackgroundColor(int color) : 设置背景颜色，将覆盖整个图表视图。 此外，背景颜色可以在布局文件 .xml 中进行设置。注意：设置颜色时要ARGB完整的八位（如 0xff00ff00）
setDescription(String desc) : 设置图表的描述文字，会显示在图表的右下角。
setDescriptionColor(int color) : 设置描述文字的颜色。 
setDescriptionPosition(float x, float y) : 自定义描述文字在屏幕上的位置（单位是像素）。
setDescriptionTypeface(Typeface t) : 设置描述文字的 Typeface。
setDescriptionTextSize(float size) : 设置以像素为单位的描述文字，最小6f，最大16f。 
setNoDataTextDescription(String desc) : 设置当 chart 为空时显示的描述文字。 
setDrawGridBackground(boolean enabled) : 如果启用，chart 绘图区后面的背景矩形将绘制。 
setGridBackgroundColor(int color) : 设置网格背景应与绘制的颜色。 
setDrawBorders(boolean enabled) : 启用/禁用绘制图表边框（chart周围的线）。
setBorderColor(int color) : 设置 chart 边框线的颜色。
setBorderWidth(float width) : 设置 chart 边界线的宽度，单位 dp。
setMaxVisibleValueCount(int count) : 设置最大可见绘制的 chart count 的数量。 只在 setDrawValues() 设置为 true 时有效。

启用/ 禁止 手势交互
setTouchEnabled(boolean enabled) : 启用/禁用与图表的所有可能的触摸交互。
setDragEnabled(boolean enabled) : 启用/禁用拖动（平移）图表。
setScaleEnabled(boolean enabled) : 启用/禁用缩放图表上的两个轴。
setScaleXEnabled(boolean enabled) : 启用/禁用缩放在x轴上。
setScaleYEnabled(boolean enabled) : 启用/禁用缩放在y轴。
setPinchZoom(boolean enabled) : 如果设置为true，捏缩放功能。 如果false，x轴和y轴可分别放大。
setDoubleTapToZoomEnabled(boolean enabled) : 设置为false以禁止通过在其上双击缩放图表。
setHighlightPerDragEnabled(boolean enabled) : 设置为true，允许每个图表表面拖过，当它完全缩小突出。 默认值：true
setHighlightPerTapEnabled(boolean enabled) : 设置为false，以防止值由敲击姿态被突出显示。 值仍然可以通过拖动或编程方式突出显示。 默认值：true

图表的 抛掷/减速
setDragDecelerationEnabled(boolean enabled) : 如果设置为true，手指滑动抛掷图表后继续减速滚动。 默认值：true。
setDragDecelerationFrictionCoef(float coef) : 减速的摩擦系数在[0; 1]区间，数值越高表示速度会缓慢下降，例如，如果将其设置为0，将立即停止。 1是一个无效的值，会自动转换至0.9999。

高亮
highlightValues(Highlight[] highs) : 高亮显示值，高亮显示的点击的位置在数据集中的值。 设置null或空数组则撤消所有高亮。
highlightValue(int xIndex, int dataSetIndex) : 高亮给定xIndex在数据集的值。 设置xIndex或dataSetIndex为-1撤消所有高亮。
getHighlighted() : 返回一个 Highlight[] 其中包含所有高亮对象的信息， xIndex 和 dataSetIndex。
以java编程方式使得值高亮不会回调 OnChartValueSelectedListener .

mLineChart.setDrawGridBackground(false);是否绘制背景颜色,如果是true,setGridBackgroundColor 和 setBackgroundColor 无效
mLineChart.setGridBackgroundColor(Color.CYAN);

XAxis 
X轴标签设置。只使用setter方法来修改它，不要直接访问公共变量。请注意，not all features the XLabels class provides are suitable for the RadarChart（对于RadarChart蜘蛛网状图不是所有的Xlabels 都适用）。

YAxis 
Y轴标签设置和它的条目。只使用setter方法来修改它，不要直接访问公共变量。请注意，not all features the XLabels class provides are suitable for the RadarChart（对于RadarChart蜘蛛网状图不是所有的Ylabels 都适用）。在为 chart 设置 data 之前，影响轴的值范围的 Customizations 需要先被应用。

“轴”类允许特定的Style，由以下 components/parts 组成（可以包含）：
轴的标签（y轴垂直绘制 或 x轴水平取向），contain 轴的描述值。
所谓 axis-line 被直接绘制在便签旁且平行。
grid-lines 在水平方向，且源自每一个轴标签。
LimitLines 允许呈现的特别信息，如边界或限制。

setEnabled(boolean enabled) : 设置轴启用或禁用。如果false，该轴的任何部分都不会被绘制（不绘制坐标轴/便签等）
setDrawGridLines(boolean enabled) : 设置为true，则绘制网格线。 
setDrawAxisLine(boolean enabled) : 设置为true，则绘制该行旁边的轴线（axis-line）.
setDrawLabels(boolean enabled) : 设置为true，则绘制轴的标签。 

Styling / 修改轴　
setTextColor(int color) : 设置轴标签的颜色。
setTextSize(float size) : 设置轴标签的文字大小。
setTypeface(Typeface tf) : 设置轴标签的 Typeface。
setGridColor(int color) : 设置该轴的网格线颜色。
setGridLineWidth(float width) : 设置该轴网格线的宽度。
setAxisLineColor(int color) : 设置轴线的轴的颜色。
setAxisLineWidth(float width) : 设置该轴轴行的宽度。 
enableGridDashedLine(float lineLength, float spaceLength, float phase) : 启用网格线的虚线模式中得出，比如像这样“ - - - - - - ”。
“lineLength”控制虚线段的长度
“spaceLength”控制线之间的空间
“phase”controls the starting point.

限制线
两个轴支持 LimitLines 来呈现特定信息，如边界或限制线。LimitLines 加入到 YAxis 在水平方向上绘制，添加到 XAxis 在垂直方向绘制。 如何通过给定的轴添加和删除 LimitLines：
addLimitLine(LimitLine l) : 给该轴添加一个新的 LimitLine 。
removeLimitLine(LimitLine l) : 从该轴删除指定 LimitLine 。
还有其他的方法进行 添加/删除 操作。 
setDrawLimitLinesBehindData(boolean enabled) : 控制 LimitLines 与 actual data 之间的 z-order 。 如果设置为 true，LimitLines 绘制在 actual data 的后面，否则在其前面。 默认值：false 


X坐标轴
XAxis 类是 AxisBase 的一个子类。 
XAxis 类是所有与水平轴相关的 “数据和信息容器”。 
每个 Line-, Bar-, Scatter-, CandleStick- and RadarChart 都有一个 XAxis 对象。 XAxis 对象展示了以 ArrayList<String> 或 String[] ("xVals") 形式递交给 ChartData 对象的数据。
XAxis xAxis = chart.getXAxis();获得 XAxis 类的实例
setSpaceBetweenLabels(int characters) : 设置标签字符间的空隙，默认characters间隔是4 。 
setLabelsToSkip(int count) : 设置在”绘制下一个标签”时，要忽略的标签数。
resetLabelsToSkip() : 调用这个方法将使得通过 setLabelsToSkip(...) 的“忽略效果”失效 while drawing the x-axis. 
setAvoidFirstLastClipping(boolean enabled) : 如果设置为true，则在绘制时会避免“剪掉”在x轴上的图表或屏幕边缘的第一个和最后一个坐标轴标签项。
setPosition(XAxisPosition pos) : 设置XAxis出现的位置。TOP，BOTTOM，TOP_INSIDE，BOTTOM_INSIDE 或 BOTH_SIDED。 
setValueFormatter(XAxisValueFormatter formatter)设置自定义格式，在绘制之前动态调整x的值。 

l.setForm(Legend.LegendForm.DEFAULT); //设置图例的形状  
l.setFormSize(10);                    //设置图例的大小  
l.setFormToTextSpace(10f);            //设置每个图例实体中标签和形状之间的间距  
l.setDrawInside(false);  
l.setWordWrapEnabled(true);           //设置图列换行(注意使用影响性能,仅适用legend位于图表下面)  
l.setXEntrySpace(10f);                //设置图例实体之间延X轴的间距（setOrientation = HORIZONTAL有效）  
l.setYEntrySpace(8f);                 //设置图例实体之间延Y轴的间距（setOrientation = VERTICAL 有效）  
l.setYOffset(0f);                     //设置比例块Y轴偏移量  
l.setTextSize(14f);                   //设置图例标签文本的大小  
l.setTextColor(Color.parseColor("#ff9933"));//设置图例标签文本的颜色  

Legend
 setXOffset(floatxOffset)：设置在X轴方向的偏移量
 setYOffset(floatyOffset)：设置在Y轴方向的偏移量
 setTypeface(Typefacetf)： 设置文本的字体
 setTextSize(floatsize)：设置文本字体大小
 setTextColor(intcolor)：设置文本颜色
 setEnabled(booleanenabled)：设置是否可用（简单理解为是否显示）
 setEntries(List<LegendEntry> entries);           设置图例，传LegendEntry的集合
 setExtra(LegendEntry[]entries)    设置图例，传LegendEntry数组
 setExtra(List<Integer>colors, List<String> labels) 设置图例，传color的集合和  LegendEntry的集合
 setExtra(int[]colors, String[] labels) 设置图例，传color的数字和LegendEntry数组
  setCustom(LegendEntry[] entries)      设置图例，灰色图标不可见
  setCustom(List<LegendEntry> entries)   设置图例，灰色图标不可见
  setHorizontalAlignment(LegendHorizontalAlignment value)  设置水平对齐方式
  setVerticalAlignment(LegendVerticalAlignment value)  设置垂直对齐方式
  setOrientation(LegendOrientation value)   设置方向
  setDrawInside(boolean value)    设置是否画在图表里
  setDirection(LegendDirection pos)    设置文字的方向
  setForm(LegendForm shape)    设置形状
  setFormSize(float size)    设置形状大小
  setFormLineWidth(float size)    设置线条宽度（形状为线状时）
 setFormLineDashEffect(DashPathEffect dashPathEffect) 设置线状轨迹
  setXEntrySpace(float space)   设置图例水平方向的间距
 setYEntrySpace(float space)    设置图例垂直方向的间距
  setFormToTextSpace(float space)  设置图例和文字的间距
  setWordWrapEnabled(boolean enabled)  设置图例是否重新创建一行


Highlight[] getHighlighted()：返回当前所有的高亮集合，通过Highlight.getXPx()方法可以获取到高亮点的X坐标；
setDragEnabled()：设置视口可以被拖拽；
setHighlightPerDragEnabled():设置高亮线可以被拖拽；



