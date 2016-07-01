# 分词
##   解析命令行参数
  

参数 | 用途
---|---
model | 模型文件前缀
reference | 训练文件
development | 测试文件
algorithm | 算法 pa或ap
max-iter | 最大迭代次数
rare-feature-threshold | 稀疏特征裁剪阈值
dump-details | 是否对模型写入详细信息，可用于增量训练


##   训练
- 读取训练文件
  - 逐行读入文件，按照tag切词
  - 预处理
  - 构造词典
    - 读取每行的词，统计词频
    - 统计词频，当前n个词的词频>总词频的90%，取前n个词，存入词典
  - 构造特征空间
    - 提取特征，将提取出的特征加入模型
  - 迭代更新参数
    - 对每个训练数据，
      - 提取特征，分别按照答案的标记和提取的特征进行解码
      - 统计解码错误的个数，计算参数更新的步长，对所有错误结果相关的参数进行更新
      - pa方法
      - ap方法
      - 按组统计特征更新的次数
      - 对更新次数少的组进行裁剪
    - 计算测试集的准确率
      - p = (double)num_recalled_words / num_predicted_words;
      - r = (double)num_recalled_words / num_gold_words;
      - f = 2 * p * r / (p + r);
    - 当前f优于之前最优的f，保存模型

## 变量解释
- tid            模板id
- itx            特征id


# 词性标注
##   ● 解析命令行参数
  

参数 | 用途
---|---
model | 模型文件前缀
reference | 训练文件
development | 测试文件
algorithm | 算法 pa或ap
max-iter | 最大迭代次数
rare-feature-threshold | 稀疏特征裁剪阈值
dump-details | 是否对模型写入详细信息，可用于增量训练


# 命名实体识别
##   ● 解析命令行参数
参数 | 用途
---|---
model | 模型文件前缀
reference | 训练文件
development | 测试文件
algorithm | 算法 pa或ap
max-iter | 最大迭代次数
rare-feature-threshold | 稀疏特征裁剪阈值
dump-details | 是否对模型写入详细信息，可用于增量训练

# 附录：代码
## pa方法

```
void learn_passive_aggressive(const math::SparseVec& updated_features,
    const size_t& timestamp, const double& error, Model* model) {
    double score = model->param.dot(updated_features, false);
    learn_passive_aggressive(updated_features, timestamp, error, score, model);
  }
  double dot(const math::SparseVec& vec, bool avg = false) const {
    const double * const p = (avg ? _W_sum : _W);
    double ret = 0.;
    for (math::SparseVec::const_iterator itx = vec.begin();
        itx != vec.end();
        ++ itx) {
      ret += p[itx->first] * itx->second;
    }
    return ret;
  }
  inline double L2() const {
    double norm = 0;
    for (const_iterator itx = _vec.begin();
        itx != _vec.end(); ++ itx) {
      double val = itx->second;
      norm += (val * val);
    }
    return norm;
  }
  void add(const math::SparseVec& vec, const uint32_t& now, const double& scale = 1.) {
    for (math::SparseVec::const_iterator itx = vec.begin();
        itx != vec.end(); ++ itx) {
      uint32_t idx = itx->first;
      uint32_t elapsed = now - _W_time[idx];
      double upd = scale * itx->second;
      double cur_val = _W[idx];

      _W[idx] = cur_val + upd;
      _W_sum[idx] += elapsed * cur_val + upd;
      _W_time[idx] = now;
    }

    if (_last_timestamp < now) {
      _last_timestamp = now;
    }
  }
  void learn_passive_aggressive(const math::SparseVec& updated_features,
    const size_t& timestamp, const double& error, const double& score, Model* model) {
    double norm = updated_features.L2();
    double step = 0.;

    if (norm < 1e-8) {
      step = 0;
    }
    else {
      step = (error - score) / norm;
    }
    model->param.add(updated_features, timestamp, step);
  }
```

## ap代码
```
void learn_averaged_perceptron(const math::SparseVec& updated_features,
    const size_t& timestamp, Model* model) {
    model->param.add(updated_features, timestamp, 1.);
}

void add(const math::SparseVec& vec, const uint32_t& now, const double& scale = 1.) {
    for (math::SparseVec::const_iterator itx = vec.begin();
        itx != vec.end(); ++ itx) {
      uint32_t idx = itx->first;
      uint32_t elapsed = now - _W_time[idx];
      double upd = scale * itx->second;
      double cur_val = _W[idx];

      _W[idx] = cur_val + upd;
      _W_sum[idx] += elapsed * cur_val + upd;
      _W_time[idx] = now;
    }

    if (_last_timestamp < now) {
      _last_timestamp = now;
    }
  }
```

