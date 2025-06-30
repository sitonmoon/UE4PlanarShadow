# UE4PlanarShadow 平面阴影!
一种很省的阴影做法.基于UE4.27.2引擎源码对StaticMesh和SkeletalMesh添加了PlanarShaow Pass 
![1750834782935](https://github.com/user-attachments/assets/25f46131-14ee-41a5-9c57-1ead6416f53c)
其实是一种Trick的动态阴影做法,原理是多渲染一遍Mesh并压缩顶点到一个平面上

**优点**
- 高性能, 相比CSM在大视角多物体的情况下有绝对性能优势,因为其没有常规动态阴影的渲染开销
- 精度高, 没有CSM阴影的边缘锯齿和贴图精度低问题,因为它根本都没有阴影贴图
  
**缺点**
- 阴影仅在一个平面,不能投射到其它物体上
- 仅适用于平坦地表,有起伏的地表处阴影会被裁切,原因同上
- 没有软阴影的柔和边缘

**使用说明**  
将代码覆盖引擎源码,并重新编译引擎.  
注意,如果你的代码并非原始源代码,为了避免覆盖掉你自己的修改内容,请参考此链接:[Commit Diff from UE4.27.2 source](https://github.com/sitonmoon/UE4PlanarShadow/commit/53ed3fc473e650fd4b080e8b8f37e3347fe3db16)手动编辑文件改动.  
编译引擎后会在所有的Staticmesh和骨骼模型上多出一个新的材质插槽"Overlay Material". 该材质使用以下方式对模型进行第二遍渲染:
```
OverlayMeshBatch.CastShadow = false; //禁用阴影投射
OverlayMeshBatch.bSelectable = false; //不可选中
OverlayMeshBatch.bUseForDepthPass = false; //不渲染深度
OverlayMeshBatch.bUseSelectionOutline = false; //不生成选中描边
```

**材质说明**  
- 母材质:M_PlanarShadow
- 材质实例:MI_PlanarShadow  
将材质实例"MI_PlanarShadow"设置给需要产生平面阴影的模型的"Overlay Material"材质插槽上.  
调节材质参数PlanePosition的B分量,这个值代表你的地平面高度,请根据项目实际情况进行设置:  
![image](https://github.com/user-attachments/assets/967d23a2-18c4-4551-91f6-621bfbf84732)


当前提供的材质使用的Opaque混合模式,所有可能会造成阴影过黑:
![image](https://github.com/user-attachments/assets/58fd7364-99f4-4977-9505-3c87e5bf1a6f)  

不过虽然可以通过调节颜色来调节,但阴影后面的颜色不会透过来,显得不太自然:
![image](https://github.com/user-attachments/assets/1413beef-a7eb-48fb-b710-1d378360cd10)  

但如果改成半透明模式,又会出现阴影重叠问题:
![image](https://github.com/user-attachments/assets/60ea3a6b-f379-4984-b0ee-29850da67c79)  

可以看到在骨骼模型的不同部件之间阴影会重叠.这个问题在Unity中可以通过增加Stencil Test的方式很方便解决,但是在虚幻里则不好做到.所以,这里我提供另一种解决思路,就是将地表颜色在阴影材质中与阴影颜色进行混合,只是一个妥协的办法,会增加消耗.



**后续优化:**  
对骨骼模型来说通常有多个部件组成,而对阴影来说不必要绘制所有的部件(阴影只需要外轮廓),可以参考这篇文章:[【虚幻小技巧-5】为网格体添加逐插槽覆层材质 - 知乎](https://zhuanlan.zhihu.com/p/709482942) 使用材质数组对部件有选择的绘制,以进一步提高性能.
