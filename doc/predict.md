# LTP 预测过程
## 分词
- 初始化
    - 加载模型
- 分词
    - 预处理
        - 正则表达式切分 URI
        - 匹配特殊字符 special_token
        - 正则表达式切分 english
        - 对上述结果进行处理，匹配出的结果合并成一个串保存到raw\_forms，类型和汉字保存到forms，type保存到chartype。其他内容转成双字节，单独存到raw_forms。
            - chartype = 左侧type（if exist） | 本身type | 右侧type（if exist）
            - raw_forms，将eng，uri等进行组合的结果；如果是其他内容，为单字
            - forms，如果是eng，uri，为_eng_，\_uri\_；如果是其他内容，与raw_forms一致
    - 匹配词典
        - 最长为5个forms，进行词典匹配
            - inst->lexicon_match_state[i] & 0x0f                           以i开头的最长词长
            - (inst->lexicon_match_state[i]>>4) & 0x0f                  以i结尾的最长词长
            - (inst->lexicon_match_state[i]>>8) & 0x0f                  i居中的最长词长
    - 抽取特征
        - 单独抽取特征
        - 根据模板对特征进行组合
        - 查找特征对应的特征id
    - 计算得分
        - 根据特征设置发射和跳转的得分
    - 解码
        - viterbi解码，没有裁剪，先前向计算，然后后向求最优路径
    - 生成结果
        - 将解码结果输出

## 词性标注
- 初始化
    - 加载模型
- 词性标注
    - 单字节字符转双字节
    - 抽取特征
        - ngram      前后共五个词，及左右词构成的二元关系
        - 前缀          穷举词的前三个字，如“北京天安门”拆成“北”，“北京”，“北京天”
        - 后缀          穷举词的后三个字，如“北京天安门”拆成“门”，“安门”，“天安门”
    - 计算得分
        - 根据特征设置发射和跳转的得分
    - 解码
        - viterbi解码，没有裁剪，先前向计算，然后后向求最优路径
    - 生成结果
        - 将解码结果输出
        - 给出标签

## 命名实体识别
- 初始化
    - 加载模型
- 命名实体识别
    - 单字节字符转双字节
    - 抽取特征
        - 1={c-2}           # 词
        - 2={c-1}
        - 3={c-0}
        - 4={c+1}
        - 5={c+2}
        - 6={c-2}-{c-1}
        - 7={c-1}-{c-0}
        - 8={c-0}-{c+1}
        - 9={c+1}-{c+2}
        - 10={p-2}          # 词性
        - 11={p-1}
        - 12={p-0}
        - 13={p+1}
        - 14={p+2}
        - 15={p-2}-{p-1}
        - 16={p-1}-{p-0}
        - 17={p-0}-{p+1}
        - 18={p+1}-{p+2}
    - 计算得分
        - 根据特征设置发射和跳转的得分
    - 解码
        - viterbi解码，没有裁剪，先前向计算，然后后向求最优路径
    - 生成结果
        - 将解码结果输出
        - 给出标签

## 依存句法分析
- 初始化
    - 加载模型
```
forms_alphabet:             73506    # 词表大小
postags_alphabet:           29       # 词性大小
deprels_alphabet:           16       # 依存标记
clusterS_types_alphabet:    19       # 聚类
clusterM_types_alphabet:    48       # 聚类
clusterB_types_alphabet:    515      # 


classifier: E(50,74151)
classifier: W1(400,3750)
classifier: b1(400)
classifier: W2(29,400)
classifier: saved(400,100000)
classifier: precomputed size=100000
classifier: hidden layer size=400
classifier: embedding size=50
classifier: number of classes=29
classifier: number of feature types=75
```
- 预测
    - 将输入的词与词性映射成id，及各cluster中的id
    - 初始化，将根节点压入栈
    - 把句子从左到右看做一个队列，遍历队列，一共 2 * N(Words) - 1步
        - 提取特征 共75个特征
            - 提取上下文 get_context
                - 栈首三个元素 (S0, S1, S2)
                - 队列前三个元素 (N0, N1, N2)
                - 栈首元素最新连接的左节点，右节点 (S0L, S0R)
                - 栈首元素次新连接的左节点，右节点 (S0L2, S0R2)
                - 栈首元素最新连接的左节点的最新连接的左节点，最新连接的右节点的最新连接的右节点 (S0LL, S0RR)
                - 栈第二个元素最新连接的左节点，右节点 (S1L, S1R)
                - 栈第二个元素次新连接的左节点，右节点 (S1L2, S1R2)
                - 栈第二个元素最新连接的左节点的最新连接的左节点，最新连接的右节点的最新连接的右节点 (S1LL, S1RR)
            - 提取基本特征 get_basic_feature 48个特征
                - S0, S1, S2, N0, N1, N2 对应的 词id + 偏移量，词性id + 偏移量
                - S0L, S0R, S0L2, S0R2, S0LL, S0RR, S1L, S1R, S1L2, S1R2, S1LL, S1RR 对应的 词id + 偏移量，词性id + 偏移量，与父节点连接的依存标记id + 偏移量
            - 提取栈首两个元素的距离特征 1个特征
                - MAP[S0 - S1] + 偏移量
            - 提取“价”特征 4个特征
                - S0 和 S1 的左孩子，右孩子数目
            - 提取聚类特征 22个特征
                - S0 和 N0 对应的 三个聚类的id
                - S1，S2，N1，N2 对应 cluster的id
                - S0L, S0R, S0L2, S0R2, S0LL, S0RR, S1L, S1R, S1L2, S1R2, S1LL, S1RR 对应 cluster的id
        - 使用NN对特征进行计算，获取各类别得分(共29类，14种标签 * 左右两种情况 + 入栈操作)
            - 遍历各特征，计算其对应的特征id，查找其对应的embedding
                - 如果找到，hidden_layer += embedding
                - 否则，hidden_layer += W1 * E(fid)
            - hidden_layer += bias
            - output = W2 * hidden_layer
        - 获取可能的动作
            - 如果没有遍历到队列尾，可以进行入栈
            - 如果栈内只有两个元素且遍历到队列尾，可以添加右侧边
            - 如果栈内超过两个元素，遍历所有标签，可以添加左侧边，也可以添加右侧边
        - 对可能的动作查分，取得得分最高的动作
        - 执行动作
            - 入栈
                - 将队列首个元素入栈
            - 添加左侧边
                - 将栈首第二个元素(top1)出栈，将top1的头连接到top0上，top1的标签为deprel
                - 更新top0的left_most_child和left_2nd_most_child
                - 更新top0的左孩子数目
                - 记录当前状态的动作和上一个状态的信息
            - 添加右侧边
                - 将栈首第一个元素(top0)出栈，将top0的头连接到top1上，top0的标签为deprel
                - 更新top1的right_most_child和right_2nd_most_child
                - 更新top1的右孩子数目
                - 记录当前状态的动作和上一个状态的信息
    - 整理结果并输出
