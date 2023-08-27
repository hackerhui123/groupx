该项目使用C++对SM3加密方案进行了加速，具体代码实现是使用SIMD程序指令实现了原加密方案中消息扩展的部分，将原本的串行优化为并行，使得程序运行更快。其他部分代码原理已在代码中注释。
直接运行代码即可
使用Visual Studio运行该代码时，Debug模式会输出详细运行信息，显示全部加密过程，Release模式只会显示最终结果。
代码运行截图：

Release模式：

![image](https://github.com/hackerhui123/groupx/assets/107422784/29f9d876-1abb-4111-9903-56f47a086d92)
Debug模式：

![image](https://github.com/hackerhui123/groupx/assets/107422784/8298786a-f3d5-477c-91dd-1852ef982098)
