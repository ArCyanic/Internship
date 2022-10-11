# 2022.9

新增两个页面，从时间空间两个维度分析仓库的包构建情况

- 空间：最近一次更新的两个仓库的包构建状态比较

  ![image-20221009004225278](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20221009004225278.png)

  比较结果根据以包为单位，分别为差异项，相同项，单独项。

- 时间：当前某个仓库中不同两个时间的包构建状态比较

  ![image-20221009004147372](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20221009004147372.png)

  新增包用绿色表示，删除了的包用红色表示，状态不发生变化不展示，发生变化以箭头表示变化方向。

# 2022.8

- Overview

  ![image-20220907215524587](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907215524587.png)

- 折线-条形图（左）与堆叠柱状图（右）

  ![image-20220907215856195](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907215856195.png)

  点击左侧的bar，即可更新右侧对应数据。

  左侧展示[OBS工程](https://build.tarsier-infra.com/project)包构建趋势，右侧展示各类别工程中各构建状态的数量，可切换为表格查看详细数据。

  <img src="https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907223321507.png" alt="image-20220907223321507" style="zoom:50%;" />

​	

- Diff界面：工程差异表格

  ![image-20220907233807349](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907233807349.png)

  红色的行表示相对于上一次commit已经删除了的行，绿色的行则表示增加。
  
- Package搜索

  ![image-20220907234043211](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907234043211.png)

  在搜索框内输入包名，点击搜索即可。
  
- 数据更新

  由于数据更新发生在后端而且有文件的保存，因此考虑将其设计为手动触发。

  目前考虑两种更新方式

  1. 拉取oerv_obsdata仓库数据，数据处理后保存中间数据文件
  2. 先更新oerv_obsdata仓库数据本身，再执行1

  要更新数据，点击页面左上角Brand处的文字/图标即可。

  Sidebar展开时Brand处的文字：

  ![image-20220907234403743](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907234403743.png)

  Sidebar缩略时Brand处的图标：

  ![image-20220907235005313](https://raw.githubusercontent.com/ArCyanic/Gener/master/image-20220907235005313.png)

  点击后会出现对应的提示信息。



# 2022.7

## Part I

网页总览图以及工程统计板块表格：

<img src="https://cdn.jsdelivr.net/gh/ArCyanic/Gener/20220810015940.png" style="zoom:50%;" />

该板块用于展示[OBS工程构建情况的统计信息](https://gitee.com/phoebe-xi/oerv_obsdata/blob/master/obsData/projectStatistics.txt)，目前只制作出表格，后续补全各类型可视化图表，增加日期选择器。

工程差异板块表格：

![](https://cdn.jsdelivr.net/gh/ArCyanic/Gener/20220810025315.png)

该板块对比的是存在commit记录的任意两天的OBS工程构建情况的数据，数据的改变用颜色和caret符号共同表示，受限于数据，上图没有展示出增加或者删除的数据，后续增加日期选择器。

## Part II

目前还有三个已完成图表绘制和数据模拟，但未对真实数据进行可视化的图表：

<img src="https://cdn.jsdelivr.net/gh/ArCyanic/Gener/20220810030259.jpg" style="zoom:50%;" />

点击某一个日期上的bar时或进入详细页面：

<img src="https://cdn.jsdelivr.net/gh/ArCyanic/Gener/20220810025821.jpg" style="zoom:50%;" />

同时可以切换成表格视图查看详细信息：

<img src="https://cdn.jsdelivr.net/gh/ArCyanic/Gener/20220810030354.jpg" style="zoom:50%;" />