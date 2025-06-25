# UE4PlanarShadow
基于UE4.27.2引擎源码 添加了PlanarShaow Pass 
![1750834782935](https://github.com/user-attachments/assets/25f46131-14ee-41a5-9c57-1ead6416f53c)
其实是一种Trick的动态阴影做法,原理是多渲染一遍Mesh并压缩顶点到一个平面上

优点
- 高性能, 相比CSM在大视角多物体的情况下有绝对性能优势,因为其没有常规动态阴影的渲染开销
- 精度高, 没有CSM阴影的边缘锯齿和贴图精度低问题
  
缺点:
- 仅在一个平面,不能投射到其它物体上
- 没有软阴影的柔和边缘

