**实验思路**

该方法是一种改进的生日攻击，其并不会优化运行速度，但是可以节省更多空间。与之前生日攻击中使用字典的方法类似，该方法并不会先开辟一可以容纳所有原像的空间，而是边寻找碰撞边存储，使得一些不必要的信息对可以不用存储。

若按照该方法找到一对相同的数据，则一定能够找到碰撞，其正确性证明如下：
![Uploading image.png…]()


**代码运行过程截图**

通过该方法找到了多个不同的碰撞
![image](https://github.com/hackerhui123/groupx/assets/107422784/50de94b8-ee1e-4b7a-8356-296c2e21517d)
![image](https://github.com/hackerhui123/groupx/assets/107422784/55963d11-5421-4cef-bf01-f3d60d7922fe)
![image](https://github.com/hackerhui123/groupx/assets/107422784/eca3e4c8-191c-4b44-ae59-e2787d67473d)

