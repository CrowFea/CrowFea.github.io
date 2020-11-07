---
title: Metal | 基本渲染流程
date: 2020-07-09 21:28:12
categories:
    - iOS
tags: 
    - Metal
mathjax: true
---

## Setup
### device
Metal的绘图是从获取device开始的。
```swift
let device = MTLCreateSystemDefaultDevice()
```
MTLDevice是一套来访问GPU的接口，实际上获得的就是对某个GPU的使用权。由device可以继续构造出后面使用的Texture、Buffer、以及Pipeline。
开发时可以选择自己要使用的GPU，对于iOS，GPU只有一块；对于macOS可以有多块显卡，因此可以去选择显卡。
可以通过以上方法来创建device，也可以通过下面的方法来获得所有的GPU。
```swift
func MTLCopyAllDevices() -> [MTLDevice]
```
<!--more-->
### MTKView
所有的metal绘制在一个特殊的view上即MTKView，当调用到view的时候，view会提供一个`MTLRenderPassDescriptor`，描述符会提供一个可供渲染的材质。描述符本身实际上就是一堆附属的属性，描述的是render出来的像素要去哪里（指的就是材质）。view管理内部的object通过`CAMetalLayer`，本质上是一个CALayer，专门用来给metal渲染。
```swift
let frame = CGRect(x: 0, y: 0, width: 600, height: 600)
let view = MTKView(frame: frame, device: device)
view.clearColor = MTLClearColor(red: 1.0, green: 1.0, blue: 0.8, alpha: 1.0)
```

## model
### MDLMesh & MTKMesh
MDLMesh是顶点数据的存储结构，MTKMesh是用来装MDLMesh的。对于初始化，MDLMesh一般是通过对MDLAsset的操作获得。后者的初始化方式同通常是通过读取一个特定的url，获取的一整个model。MDLMesh同样可以通过开发者自定义的顶点数据进行初始化。MDLMesh初始化时支持了多个基础的几何图形，包括立方体、球体、圆柱、圆锥、平面、胶囊、20面体、半球体。

MDLMesh内部实际上有很多个submesh，对于一个体，创建的三维模型可能由不同的submesh组合起来。比如说一个汽车，车体和轮胎很有可能是两个不同的submesh，后者的submesh会 重复的使用四次。
[Apple Developer Documentation](https://developer.apple.com/documentation/modelio/mdlmesh)
```swift
let allocator = MTKMeshBufferAllocator(device: device)

let mdlMesh = MDLMesh(sphereWithExtent: [0.75,0.75,0.75], segments: [100,100], inwardNormals: false, geometryType: .triangles, allocator: allocator)
let mesh = try MTKMesh(mesh: mdlMesh, device: device)

```


## Render
### MTLLibrary
创建你的lib，一个library中包含了顶点着色器、片源着色器等方法。Library可以通过多种方式创建，我们可以在运行时才进行创建，通过着色器的名称进行连接。

这其中连接的就是shader的方法，shader可以单独开一个metal文件。也可以通过string的方式连接起来。

```swift
let shader = """
#include <metal_stdlib> using namespace metal;
struct VertexIn {
  float4 position [[ attribute(0) ]];
};
vertex float4 vertex_main(const VertexIn vertex_in [[ stage_in ]]) {
return vertex_in.position; }
fragment float4 fragment_main() {
  return float4(1, 0, 0, 1);
}
"""

let lib = try device.makeLibrary(source: shader, options: nil)
let vertexFunction = lib.makeFunction(name: "vertex_main")
let fragmentFunction = lib.makeFunction(name: "fragment_main")
```

### Pipeline
管线可以是持久的，创建管线通过descriptor。一个描述符内包含了对这个管线的大部分配置，一个最小化配置的管线至少包括以下三个部分
- 顶点着色器
- 片源着色器
- 颜色的编码格式
```swift
let descrioptor = MTLRenderPipelineDescriptor()
descrioptor.colorAttachments[0].pixelFormat = .bgra8Unorm
descrioptor.vertexFunction = vertexFunction
descrioptor.fragmentFunction = fragmentFunction
descrioptor.vertexDescriptor = MTKMetalVertexDescriptorFromModelIO(mesh.vertexDescriptor)
```
最后通过描述符创建一个`pipelinestate`，一个state代表了一个编译好的渲染管线。
```swift
let pipelineState = try device.makeRenderPipelineState(descriptor: descrioptor)
```

### Queen & Buffer
这里其实也是设置的一部分。commandQueen是是用来管理commandBuffer的。commandBuffer是来存储提交给GPU的指令。一个buffer并不是需要持有引用，在每一帧的时候可能会创建，提交给queen，GPU处理完之后再去创建新的buffer，他的生命周期会自己结束。
Queen是一个持久的对象，并不需要在每一帧的时候单独的创建。
```swift
guard let commandQueen = device.makeCommandQueue() else {
    fatalError("oops")
}
```

### render
commandBuffer创建renderEncoder，后者来负责将渲染的状态和代码翻译成机器可以执行的指令。encoder的初始化通过`renderpassdescriptor`，这个可以直接通过MTKView来获取，接下来还要设置对应的pipelineState，顶点buffer等

然后就可以开始发起draw call，再draw call结束后显示的声明draw call结束即可。

```swift
guard let commandBuffer = commandQueen.makeCommandBuffer(),
let descriptor = view.currentRenderPassDescriptor,
let renderEncoder = commandBuffer.makeRenderCommandEncoder(descriptor: descriptor) else { fatalError() }

renderEncoder.setRenderPipelineState(pipelineState)
renderEncoder.setVertexBuffer(mesh.vertexBuffers[0].buffer, offset: 0, index: 0)

guard let submesh = mesh.submeshes.first else {
    fatalError("oops")
}

renderEncoder.drawIndexedPrimitives(type: .triangle, indexCount: submesh.indexCount, indexType: submesh.indexType, indexBuffer: submesh.indexBuffer.buffer, indexBufferOffset: 0)

renderEncoder.endEncoding()
```

## Present
虽然渲染完了，但是还没有展示到屏幕上。这是只需要commandBuffer调用方法，最后commit提交给GPU即可。
```swift
guard let drawable = view.currentDrawable else {
    fatalError()
}

commandBuffer.present(drawable)
commandBuffer.commit()
```