## 为什么BN可以起到正则化的作用？

（1）过拟合一般发生在数据边界处，而BN把数据归一化了

（2）归一化的数据引入了噪声，这在训练时一定程度有正则化的效果。

##为什么使用BN可以增大学习率？

如果每层的scale不一致，每层需要的学习率是不一样的，通常需要使用最小的那个学习率才能保证损失函数有效下降，Batch Normalization将每层、每维的scale保持一致，那么我们就可以直接使用较高的学习率进行优化。



class Solution(object):
    def buildTree(self, inorder, postorder):
        """
        :type inorder: List[int]
        :type postorder: List[int]
        :rtype: TreeNode
        """
        inOrderMap = {}
        for index, val in enumerate(inorder):
            inOrderMap[val] = index
        return self.build(inorder, 0, len(inorder) - 1, postorder, 0, len(postorder) - 1, inOrderMap)
    def build(self, inorder, iStart, iEnd, postorder, pStart, pEnd, inOrderMap):
        if iStart > iEnd:
            return None
        root = TreeNode(postorder[pEnd])
        index = inOrderMap[root.val]
        lsize = index - iStart
        rsize = iEnd - index
        root.left = self.build(inorder, iStart, index-1, postorder, pStart, pStart + lsize - 1, inOrderMap)
        root.right = self.build(inorder, index+1, iEnd, postorder, pEnd - rsize, pEnd - 1, inOrderMap)
        return root