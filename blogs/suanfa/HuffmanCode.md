# 哈夫曼编码详解与代码实现
## 简介
哈夫曼编码(霍夫曼编码)是由美国计算机科学家 David Albert Huffman 于 1952 年发明的数据压缩算法
## 压缩原理
现实中我们看到的字符编码都是**等长编码**，例如常见的**ASCII编码**大小为一个字节(8bit),但一个字符使用
的频率有高低，使用等长编码就造成了浪费，**哈夫曼编码就是一种变长编码算法**，根据字符出现频率的不同，对应的编码
长度也不一样，规律是频率越高，字符编码长度越短，总体的平均编码长度小于等于等长编码  
例如，一条英文语句使用了10种英文字符，使用ASCII编码每个字符占用的大小为 8bit, 而使用哈夫曼编码每个字符的平均编码
长度为 4bit，生出来编码长度就是压缩空间
## 实现原理
### 哈夫曼编码
编码过程**本质是在构造哈夫曼树**，哈夫曼树是带权路径长度最短的二叉树，构造过程如下
1. 将要编码的字符加入队列，按出现的频率(权重)升序排序
2. 取出两个频率最小的节点，构造哈夫曼树，父节点为频率只和
3. 将父节点加入队列，并按频率升序排序
4. 重复第二步和第三步，直到队列中只有一个节点，即为构造好的哈夫曼树     

得到哈夫曼树，**遍历哈夫曼树即可得到哈夫曼编码**，本质是二叉树的深度遍历，唯一多出的步骤是向左遍历添加字符0，
向右遍历添加字符1，遇到叶子节点，停止添加，得到该字符的哈夫曼编码
### 哈夫曼解码
解码是哈夫曼树遍历的逆过程，遇到字符0选择左子节点，遇到字符1选择右子节点，直到遇到叶子节点，即可得到该编码对应的
字符
## 代码实现
### 哈夫曼树结构定义
```java
abstract public class HuffmanTree implements Comparable<HuffmanTree> {

    public final int frequency;
    public HuffmanTree(int freq) {
        frequency =  freq;
    }

    //按频率升序排序
    @Override
    public int compareTo(HuffmanTree o) {
        return frequency - o.frequency;
    }
}

public class HuffmanLeaf extends HuffmanTree {

    public final char value;

    public HuffmanLeaf(int freq, char val) {
        super(freq);
        value = val;
    }
}

public class HuffmanNode extends HuffmanTree {

    public final HuffmanTree left, right;

    public HuffmanNode(HuffmanTree left, HuffmanTree right) {
        super(left.frequency + right.frequency);
        this.left = left;
        this.right = right;
    }
}
```
### 编码与解码
```java
public class HuffmanCode {

    //构造哈夫曼树
    public static HuffmanTree buildTree(int[] charFreqs) {

        //本质是小顶堆
        PriorityQueue<HuffmanTree> trees = new PriorityQueue<>();

        //构造叶子结点
        for (int i = 0; i < charFreqs.length; i++) {
            if (charFreqs[i] > 0) {
                trees.offer(new HuffmanLeaf(charFreqs[i], (char)i));
            }
        }

        if(trees.size() < 0) return null;

        //构造Huffman树，取出权重最小的两个叶子结点，构造树结构后再次加入小顶堆，再次取出，直到只有一棵树
        while (trees.size() > 1) {
            HuffmanTree a = trees.poll();
            HuffmanTree b = trees.poll();

            trees.offer(new HuffmanNode(a, b));
        }

        return trees.poll();
    }

    //哈夫曼编码
    public static void huffmanEncode(HuffmanTree tree, StringBuffer prefix) {

        if(tree == null) return;

        //遇到叶子节点，停止遍历，打印编码值
        if(tree instanceof HuffmanLeaf) {
            HuffmanLeaf leaf = (HuffmanLeaf) tree;
            System.out.println(leaf.value + "\t" + leaf.frequency + "\t" + prefix);
        } else if (tree instanceof HuffmanNode) {
            HuffmanNode node = (HuffmanNode) tree;

            // 遍历左树
            prefix.append('0');
            huffmanEncode(node.left, prefix);
            //回退一步，遍历另一边的分支
            prefix.deleteCharAt(prefix.length() -1);

            // 遍历右树
            prefix.append('1');
            huffmanEncode(node.right, prefix);
            //回退一步，遍历另一边的分支
            prefix.deleteCharAt(prefix.length() -1);

        }
    }

    // 哈夫曼解码
    public static void huffmanDecode(HuffmanTree temp, String code, int index, HuffmanTree tree) {

        if(temp == null) throw new RuntimeException("输入了错误编码");

        //遇到叶子节点，输出该编码对应的字符
        if(temp instanceof HuffmanLeaf) {
            HuffmanLeaf leaf = (HuffmanLeaf) temp;
            System.out.print(leaf.value);
            //走到叶子节点，临时节点恢复为根节点
            temp = tree;
            if(index < code.length()) {
                huffmanDecode(temp, code, index, tree);
            }
        } else if (temp instanceof HuffmanNode) {
            HuffmanNode node = (HuffmanNode) temp;
            if('0' == code.charAt(index)) {
                huffmanDecode(node.left, code, index+1, tree);
            } else if('1' == code.charAt(index)) {
                huffmanDecode(node.right, code, index+1, tree);
            } else {
                throw new RuntimeException();
            }
        }

    }

    public static void main(String[] args) {

        String test = "this is an example for huffman encoding and decoding, very nice! ";
        int[] charFreqs = new int[256];

        //存储频率，索引即为字符本身
        for (char c : test.toCharArray()) {
            charFreqs[c] ++;
        }

        HuffmanTree tree = buildTree(charFreqs);

        System.out.println("SYMBOL\tWEIGH\tTHUFFMAN CODE");
        huffmanEncode(tree, new StringBuffer());

        //解码后得到"today good!"
        String code = "10101111011011010000111011111011111011110110110011100";
        huffmanDecode(tree, code, 0, tree);

    }
}
```
## 时间复杂度与空间复杂度
**时间复杂度**：O(nlogn)，n 为编码字符的个数，一次堆排和遍历到叶子结点时间复杂度为 O(logn)，n 个节点得到
时间复杂度为 O(nlogn)
**空间复杂度**：O(n)，n 个叶子节点，总共 2n-1 个节点
## 哈夫曼编码思维在日常生活中的实践
在我看来，哈夫曼编码本质就是**将更高权重的事情以更高的效率和权重分配**，以前上学的时，总是将时间均等的分配在
各科上，甚至是擅长哪个科目，就更爱花时间在这个科目上，现在想来应该优先将时间更多分配在那些提升空间大的科目上，
那会我有这意识，定能上清华(狗头)


