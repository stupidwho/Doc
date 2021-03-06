# Dialog

**Dialog作为一个常用的控件，和Activity、Fragment的显示有所不同，是另一种特殊情况下的展示界面，主要用于在使用过程中，想要轻量级弹出消息，但同时又是一种会阻塞用户交互的存在，占据整个界面，用户必须做出反馈才能进行下一步操作**

### 如何自定义界面

#### 基本内容

* 一般从Dialog继承，不会有暗色背景，如需阴影背景，则应该继承AlertDialog
* 实现和Activity类似的onCreate方法，实现界面的修改

#### 特点

由于界面管理是和Activity不一样的，UI上和Activity属于平级的内容，因此Activity的主题内容并不会影响到Dialog，或者说只是一个默认值获取的地方，Dialog会单独有自己的对Window的管理，因此在考虑Dialog的显示的时候，可以将Activity相关的内容（主要是主题相关）平移到Dialog上，同样是Title、状态栏、界面透明等等需要考虑的内容，而不是以View的方式来考虑，就像本人工作中曾经遇到的修改Dialog的界面内容（状态栏），应该像考虑Activity一样可以从设置主题入手