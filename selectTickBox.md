### 实现selectTickBox

##### 一、基础知识

###### 1. 获取元素的宽高

clientHeight = height + padding

offsetHeight = height + padding + border

scrollHeight = height + padding



**clientHeight和offsetHeight的注意点**

​		当元素设置为display:none;之后，clientHeight和offsetHeight的高度均变为0，因为浏览器不会对display:none的元素进行渲染
​		当父元素没有明确高度，而是由子盒子撑开时，clientHeight和offsetHeight计算高度时，不会计算设置了绝对定位（absolute）或者固定定位（fixed）的元素的宽高，只会对子元素中的标准流元素和清除了浮动的浮动元素高度进行累加得到父元素的实际高度。



###### 2. 获取偏移量

clientLeft：该元素对象的左边框宽度

offsetLeft：该元素左外边框至窗口左边界的距离，返回当前元素的相对水平偏移位置

scrollLeft：页面利用滚动条滚动到右侧时，隐藏在滚动条左侧的页面的宽度

（滚动条在父元素身上，所以判断滚动的距离应该在父元素上判断）



###### 3. 获取鼠标点击位置

clientX：clientX  以**浏览器窗口左上顶角**为原点，定位 x 轴坐标  所有浏览器，不兼容 Safari

offsetX：当事件被触发时鼠标指针相对于**所触发的标签元素**的左**内边框**的水平坐标。所有浏览器，不兼容 Mozilla

pageX： 以 document 对象（即文档窗口）左上顶角为原点，定位 x 轴坐标  所有浏览器，不兼容 IE

screenX：计算机屏幕左上顶角为原点，定位 x 轴坐标  所有浏览器

layerX：最近的绝对定位的父元素（如果没有，则为 document 对象）左上顶角为元素，定位 x 轴坐标  Mozilla 和 Safari



##### 二、实现

分析：当在container区域点击鼠标左键键并移动的时候，绘制一个矩形框，松开时矩形框消失，在矩形框中的选择框被勾选

流程：

​		鼠标按下：绘制一个框选mask，绘制的位置以container的左上角为基准，因此需要计算鼠标点击的位置距离基准点的距离

​		鼠标移动：鼠标移动时，根据当前鼠标所在位置（endX、endY）和初始鼠标按下的位置（startX、startY）显示框选的mask，计算出mask的宽度高度以及mask的top和left值进行定位，定位时如果始终以初始位置设置top和left值，则只能向右下方拖动，因此需要比较end和start的值，选出较小的值作为top和left值。

​		鼠标弹起：隐藏mask，遍历每个勾选框，判断勾选框是否在mask中，判断方式是获取勾选框的边界以及mask的边界值并进行比较，如果在mask中就让其勾选



##### 三、问题及解决方案

1.绘制mask框时，绘制范围会超出container盒子

​	解决：设置`overflow:hidden`对其进行隐藏，但是当mask有边框时，就会出现部分边框看不见的情况，因此通过判断点击的位置将mask的绘制范围设置在盒子内部



2.鼠标左键弹起后，盒子没有隐藏并且继续随着鼠标的位置移动

​	解决：鼠标弹起后会触发mouseup事件，此时会隐藏元素，但是随后移动鼠标又会触发mousemove事件，因此设置一个标记值hasMouseDown，只有当该值为true的时候，表明之前已经触发了mousedown事件，才会接着触发mousemove事件，在mouseup事件中将其值设置为false，这样鼠标弹起后再移动不会触发mouseMoveEvent。



3.当鼠标左键在container外的右下区域弹起时，遮罩mask不消失

​	解决：由于事件绑定在body上，而container外的右下区域不在body内部，不会触发mouseup事件，因此使用home盒子把body撑开



4.窗口缩小时，绘制盒子的位置和点击的位置出现了偏移

​	解决：由于窗口缩小，会出现滚动条，导致clientX和clientY发生改变，因此在判断点击的位置时候出现位置偏差，需要在原有的top和left值上加上滚动的距离



5.不断改变样式，引起回流重绘

​	解决：设置定时器使其每50ms触发一次



