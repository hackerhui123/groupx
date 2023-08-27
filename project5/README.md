**实验原理**
Merkle 树（Merkle Tree）是一种哈希树的数据结构，由于其高效的验证和完整性检查特性，在密码学和分布式系统中被广泛使用。它的实现原理如下：

数据分割：将要构建 Merkle 树的数据按照固定大小（通常是2的幂次方）进行划分，每个划分称为一个叶子节点。

哈希计算：对每个叶子节点进行哈希计算，生成对应的哈希值。常用的哈希函数有 SHA-256、MD5 等。

层级构建：将生成的哈希值两两配对，再对两个哈希值进行合并计算得到父节点哈希值。如果叶子节点个数为奇数，最后一个节点可以重复使用。

重复步骤 3 直到只剩下一个根节点，即 Merkle 树的根节点。根节点存储着整个树的哈希值，作为唯一标识。

通过这种方式，Merkle 树逐层向上构建，最终形成以根节点为根的哈希树。Merkle 树的主要优势在于验证数据完整性的高效性，因为如果数据块发生变化，只需要重新计算受影响的节点及其父节点的哈希值即可。

![image](https://github.com/hackerhui123/groupx/assets/107422784/910a4919-40ef-481b-85dd-33ec2dcd69d9)
**代码实现**
from hashlib import sha256
import random
import string
import math


# 随机生成data
def gen_data(length):
    res = []
    for i in range(length):
        block = [random.choice(string.digits + string.ascii_letters) for i in range(5)]  # 一个消息块长为5
        res.append(''.join(block))
    return res


# 生成MerkleTree
def gen_mkt(data):
    depth = math.ceil(math.log2(len(data)) + 1)  # merkle tree深度
    mktree = [[sha256(i.encode()).hexdigest() for i in data]]  # merkle tree的第0层(倒置)
    for i in range(depth - 1):
        len_lay = len(mktree[i])  # 第i层消息块个数
        mkt_lay = [sha256(mktree[i][j * 2].encode() + mktree[i][j * 2 + 1].encode()).hexdigest() for j in
                   range(int(len_lay / 2))]  # merkle tree的第i+1层
        if len_lay % 2 != 0:
            mkt_lay.append(mktree[i][-1])  # 若块数为奇数，则最后一个块直接放入下一层
        mktree.append(mkt_lay)
    return mktree


def proof(spec_elm, mktree, root):
    hash_se = (sha256(spec_elm.encode())).hexdigest()
    if hash_se in mktree[0]:
        index_se = mktree[0].index(hash_se)  # 指定元素在数第一层的索引值
    else:
        return "This message isn't in the data."
    depth = len(mktree)  # merkle tree深度
    audit_path = []  # 审计路线
    for i in range(depth - 1):
        if index_se % 2 == 0:  # 左子结点
            if len(mktree[i]) - 1 != index_se:  # 该结点不是该层最后的一个单独结点
                audit_path.append(['l', mktree[i][index_se + 1]])
        else:  # 右子结点
            audit_path.append(['r', mktree[i][index_se - 1]])
        index_se = int(index_se / 2)    # 更新索引值
    for ele in audit_path:
        if ele[0] == 'l':
            hash_se = sha256(hash_se.encode() + ele[1].encode()).hexdigest()
        else:
            hash_se = sha256(ele[1].encode() + hash_se.encode()).hexdigest()
    if hash_se == root:
        return "This message is in the merkle tree."
    else:
        return "This message is in the data but it isn't in the merkle tree."


if __name__ == "__main__":
    index = random.randint(0, 99999)
    data = gen_data(100000)
    data2 = gen_data(100000)
    spec_element = data[index]       # inclusion proof
    data2[index] = spec_element      # exclusion proof 指定消息在data中
    spec_element2 = '12345'          # exclusion proof 指定消息不在data中(大概率)
    mktree = gen_mkt(data)
    mktree2 = gen_mkt(data2)
    root = mktree[-1][0]
    # for i in mktree:          # 输出merkle tree
    #     print(i, "\n")
    print("For message:", spec_element, proof(spec_element, mktree, root))
    print("For message:", spec_element, proof(spec_element, mktree2, root))
    print("For message:", spec_element2, proof(spec_element2, mktree, root))
    **运行结果**
    代码会进行如下三步：

1.创建一棵MerkleTree，称之为MkT1，并随机从其Data中挑选一个块检验其是否在MkT1中。

2.创建一棵MerkleTree，称之为MkT2，且保证其Data中有一个块与MkT1的Data中有一个块相同，并检验该块是否在MkT1中。

3.检验一个已知的数据块(大概率不在MkT1的Data中)是否在MkT1的Data中。

![image](https://github.com/hackerhui123/groupx/assets/107422784/60a69f3d-818c-4720-ad06-f3e7022a2333)
