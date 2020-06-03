---
title: gpu.js
date: "2018-06-08 11:17:54"
tags: ["javascript", "GPU"]
categories: ["Javascript", "WebGPU"]
desc: gpu.js 记录
---

## gpu.js 使用记录

1. 定义GPU

   ```javascript
   const mode = 'gpu';
   // gpu, cpu
   const gpu = new GPU({ mode: mode });
   console.log(gpu.getMode());
   // gpu 
   ```

<!-- more -->

2. 输出

   * output：

     array： [width], [width, height], [width, height, depth]

     object: { x: width, y: height, z: depth}

   * outputToTexture (Boolean)

     值保存到texture中。

   * floatOutput

   * floatTextures

   * functions

3. 输入

   * 数字
   * 一位数组
   * 二维数组
   * 三维数组
   * html image
   * array of html image

4. 使用thread

   ```javascript
   const myFunc = gpu.createKernel(function(image) {
       const pixel = image[this.thread.y][this.thread.x];
       this.color(pixel[0], pixel[1], pixel[2], pixel[3]);
   })
     .setGraphical(true)
     .setOutput([100]);
       
   myFunc([1, 2, 3]);
   // Result: colorful image
   ```

   输入[x, y, z]

5. 使用流水线，即在GPU中保存值。

   可以调用`outputToTexture: boolean ` 或者 `kernel.setOutputToTexture(true) `

   ```javascript
   const importAsTexture = gpu.createKernel(function(value) {
       return value[this.thread.y][this.thread.x];
   })
       .setOutput([512, 512])
       .setOutputToTexture(true);
   
   const texture = importAsTexture([1, 2]);
   // console.log(texture);
   ```
   ![存储在内存中的texture](https://i.loli.net/2018/06/08/5b19e936e027a.png)

6. Gpu.js 处理图片 https://github.com/gpujs/gpu.js/issues/278

7. 输出数组

   ```
   const k = gpu.createKernel(function () {
     return this.thread.x % 2;
   }).setOutput([6, 2, 1]);
   
   console.log(k());
   ```

   ![QQ截图20180608103424.png](https://i.loli.net/2018/06/08/5b19eb44b7b8e.png)

8. 添加额外的定制函数

   `addFunction` 

   ```javascript
   gpu.addFunction(function mySuperFunction(a, b) {
   	return a - b;
   });
   function anotherFunction(value) {
   	return value + 1;
   }
   gpu.addFunction(anotherFunction);
   const kernel = gpu.createKernel(function(a, b) {
   	return anotherFunction(mySuperFunction(a[this.thread.x], b[this.thread.x]));
   }).setOutput([20]);
   ```

   或者添加 `setFunctions`

   ```javascript
   function mySuperFunction(a, b) {
   	return a - b;
   }
   const kernel = gpu.createKernel(function(a, b) {
   	return mySuperFunction(a[this.thread.x], b[this.thread.x]);
   })
     .setOutput([20])
     .setFunctions([mySuperFunction]);
   ```

   

9.  传递常量

   constants中定义常量。

   ```javascript
   const matMult = gpu.createKernel(function(a, b) {
       var sum = 0;
       for (var i = 0; i < this.constants.size; i++) {
           sum += a[this.thread.y][i] * b[i][this.thread.x];
       }
       return sum;
   }, {
     constants: { size: 512 },
     output: [512, 512],
   });
   ```

10. GPU.Input

   ```javascript
   import GPU, { input } from 'gpu.js';
   
   function run() {
     const gpu = new GPU({
       mode: 'cpu'
     });
   
     const opt = { output: [3, 3] };
     const kernel = gpu.createKernel(function(values) {
       return values[this.thread.x] + values[this.thread.y];
     }, opt);
   
   
     const gpuInput = input(new Float32Array([2, 4, 6, 1, 3, 5, 8, 10, 12]), [3, 3]);
     const output = kernel(gpuInput);
     console.log(output);
   }
   
   run();
   ```

   in GPU mode:

   ```
   (3) [Array(3), Array(3), Array(3)]
   0:(3) [4, 6, 8]
   1:(3) [6, 8, 10]
   2:(3) [8, 10, 12]
   length:3
   __proto__:Array(0)
   ```

   此方法在cpu 模式中无法使用，主要原因是：

   > Input 的实现中，主要使用了 glsl语言。
   >
   > https://github.com/gpujs/gpu.js/blob/4173dd9610b9c04fa064e7e2e12b5f066d7eeac7/src/backend/web-gl/kernel.js#L627
   >
   > That is correct, it just has not yet been implemented. The key difference on GPU over CPU is that at this time on GPU the Array's are always flat, but there is some architecture in place to help flatten them, which is expensive, but isn't worth making everyone have to pre-flatten their arrays. GPU.input skips this step, allowing for very fast computation without having to flatten, because the data is pre-flattened. I imagine the best means of which to implement this would be to use input, and then on the CPUFunctionBuilder we'd have an option (or detection?) to simply use them, altering the compiled kernel. So: input1[this.thread.z][this.thread.y][this.thread.x] would become something more like: input1[this.thread.hypotheticalndexOrSomething].
   >
   > https://github.com/gpujs/gpu.js/issues/243

   

11. CPU 、GPU mode

    ```
    const gpu = new GPU({ mode: mode });
    console.log(gpu.getMode());
    
    var cpu = new GPU({mode:'cpu'});
    // var gpu = new GPU({mode:'gpu'});
    
    function testFunc(inp) {
        return inp[this.thread.y][this.thread.x];
    }
    
    const cpuKernel = cpu.createKernel(testFunc).setOutput([3, 3]);
    const gpuKernel = gpu.createKernel(testFunc).setOutput([3, 3]);
    
    console.log('cpu:', cpuKernel( [[0,1,2], [3,4,5], [6,7,8]] ) );
    console.log('gpu:', gpuKernel( [[0,1,2], [3,4,5], [6,7,8]] ) );
    ```

    ![QQ截图20180608105121.png](https://i.loli.net/2018/06/08/5b19efbfdc3e4.png)

    如果参数不全，则在cpu中出现未定义，在GPU 中出现其他值。

    ```javascript
    const gpu = new GPU({ mode: mode });
    console.log(gpu.getMode());
    
    var cpu = new GPU({mode:'cpu'});
    // var gpu = new GPU({mode:'gpu'});
    
    function testFunc(inp) {
        return inp[this.thread.y][this.thread.x];
    }
    
    const cpuKernel = cpu.createKernel(testFunc).setOutput([3, 3]);
    const gpuKernel = gpu.createKernel(testFunc).setOutput([3, 3]);
    
    console.log('cpu:', cpuKernel( [[0, 212], [1, 12], [2,21]] ) );
    console.log('gpu:', gpuKernel( [[0, 212], [1, 12], [2,21]] ) );
    ```

    ![QQ截图20180608105323.png](https://i.loli.net/2018/06/08/5b19efc0a5d7f.png)

12. SSSP 算法

    https://github.com/pan-long/SSSP-on-GPU/blob/master/bellman_ford_gpu.js

13. 多GPU 支持，取决于运行环境（浏览器是否使用多GPU）

    https://github.com/gpujs/gpu.js/issues/190

    webGL 支持检测：

    http://alteredqualia.com/tmp/webgl-maxparams-test/

14. createKernelMap

    ```javascript
    var km = new GPU().createKernelMap([
        function add(v1, v2) {
            return v1 + v2;
        },
        function divide(v1, v2) {
            return v1 / v2;
        }
    ], function (a, b, c) {
        return divide(add(a[this.thread.x], b[this.thread.x]), c[this.thread.x]);
    }, {output: [3], floatOutput: true});
    console.log(km([1], [1], [3]));
    ```

    输出：

    ![QQ截图20180608110503.png](https://i.loli.net/2018/06/08/5b19f277153f4.png)

    完整实例：

    ```javascript
    const mode = 'gpu';
    // gpu, cpu
    const gpu = new GPU({ mode: mode });
    console.log(gpu.getMode());
    
    const size = 4; // keeping it small for testing
    const a = new Float32Array(size);
    const b = new Float32Array(size);
    const c = new Float32Array(size);
    
    for (let i = 0; i < size; ++i) {
      a[i] = 0;
      b[i] = i;
      c[i] = 2;
    }
    
    const megaKernel = gpu.createKernelMap([
      function add(a, b) {
        return a + b;
      },
      function multiply(a, b) {
        return a * b;
      }
    ], function (a, b, c) {
      return multiply(add(a[this.thread.x], b[this.thread.x]), c[this.thread.x]);
    });
    megaKernel.setOutput([ size ]);
    console.log(megaKernel(a, b, c).result);
    ```

    输出：float32Array(4) [0, 2, 4, 6]，

    完整输出：

    ![QQ截图20180608111402.png](https://i.loli.net/2018/06/08/5b19f48d45e17.png)

    https://github.com/gpujs/gpu.js/issues/258

    

     

