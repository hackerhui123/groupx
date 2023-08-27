Project：report on the application of this deduce technique in Ethereum with ECDSA
**实验原理**
签名的生成比较简单，直接调用Python中的ecdsa模块，确定使用的椭圆曲线为NIST256p，使用库中的函数和类即可完成
得到消息$m$和其对应的签名$(r,s)$后，我们需要恢复能够成功验证签名的公钥Q(曲线上的点)：
![image](https://github.com/hackerhui123/groupx/assets/107422784/8024cf59-f392-4499-b4a4-f7f9c46ea36c)
观察上述等式，已知$s,r,e=H(m)$，则只需知道$P=kG$，即可恢复出正确的验签公钥$Q$.

但是敌手并不知道$P$，只知道$r \equiv x \mod n，x\in[1,p],p$为椭圆曲线上素域的阶，通常来说$n\lt p$，因此一个$r$可能对应两个$x$：

若$r\lt p-n$，则$x=r$或$ x=r+n$
若$r\gt p-n$，则$x=r$
在$x$被确定后，需要用其确定$P$，根据椭圆曲线的公式可知一个$x$对应了两个曲线上的点$P_1,P_2$，且这两个点生成的$Q$都可以用于验签，所以理论上来看最多可以恢复4个能够正确验签的公钥。

恢复公钥需要用到的各类计算与公式在ecdsa库中都已有，只需调用即可。
**代码实现**
import ecdsa
import random
import hashlib


if __name__ == "__main__":
    G = ecdsa.NIST256p.generator
    n = G.order()
    d_A = random.randrange(1, n - 1)      # 生成私钥d_A
    # 生成公私钥对象
    public_key = ecdsa.ecdsa.Public_key(G, G * d_A)
    private_key = ecdsa.ecdsa.Private_key(public_key, d_A)
    message = "SDUCyberspaceSecurity"
    e = int(hashlib.sha1(message.encode("utf8")).hexdigest(), 16)   # Hash(m)
    k = random.randrange(1, n - 1)    # 生成临时密钥
    sign = private_key.sign(e, k)      # 生成签名(r,s)
    r = sign.r
    s = sign.s
    curve = G.curve()
    p = curve.p()       # 素域阶
    de_p_key = []       # 被恢复出的公钥
    x_lst = [r]         # P = kG,x为P横坐标,x = r mod n
    if r < p-n:
        x_lst.append(r + n)     # x = r + n是一种可能存在的情况
    for i in range(len(x_lst)):
        x = x_lst[i]
        y_square = (pow(x, 3, curve.p()) + (curve.a() * x) + curve.b()) % curve.p()     # y^2 = x^3 + ax + b
        y_hat = ecdsa.numbertheory.square_root_mod_prime(y_square, curve.p())
        y = y_hat if y_hat % 2 == 0 else curve.p() - y_hat

        R1 = ecdsa.ellipticcurve.PointJacobi(curve, x, y, 1, n)
        Q1 = ecdsa.numbertheory.inverse_mod(r, n) * (s * R1 + (-e % n) * G)     # P = (s*kG - e*G)*r^(-1) mod n
        Pk1 = ecdsa.ecdsa.Public_key(G, Q1)

        R2 = ecdsa.ellipticcurve.PointJacobi(curve, r, -y, 1, n)
        Q2 = ecdsa.numbertheory.inverse_mod(r, n) * (s * R2 + (-e % n) * G)     # P = (s*kG - e*G)*r^(-1) mod n
        Pk2 = ecdsa.ecdsa.Public_key(G, Q2)
        de_p_key.append([Pk1.point.x(), Pk1.point.y()])
        de_p_key.append([Pk2.point.x(), Pk2.point.y()])
    # 恢复的公钥不止一个
    p_key = [public_key.point.x(), public_key.point.y()]
    if p_key in de_p_key:
        print("Deduce public key from signature successfully!")
        print("The deduced public keys are:")
        for i in range(len(de_p_key)):
            print(de_p_key[i])
        print("The true public key is:\n", p_key)
    else:
        print("Didn't deduce public key from signature")
**运行截图**

![image](https://github.com/hackerhui123/groupx/assets/107422784/860bc69b-9273-4d1a-8b09-106193e12a46)
![image](https://github.com/hackerhui123/groupx/assets/107422784/481bc4dc-2a83-4e81-b544-09a7d0d06a8f)

