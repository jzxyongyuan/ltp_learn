# �ִ�
##   ���������в���
  

���� | ��;
---|---
model | ģ���ļ�ǰ׺
reference | ѵ���ļ�
development | �����ļ�
algorithm | �㷨 pa��ap
max-iter | ����������
rare-feature-threshold | ϡ�������ü���ֵ
dump-details | �Ƿ��ģ��д����ϸ��Ϣ������������ѵ��


##   ѵ��
- ��ȡѵ���ļ�
  - ���ж����ļ�������tag�д�
  - Ԥ����
  - ����ʵ�
    - ��ȡÿ�еĴʣ�ͳ�ƴ�Ƶ
    - ͳ�ƴ�Ƶ����ǰn���ʵĴ�Ƶ>�ܴ�Ƶ��90%��ȡǰn���ʣ�����ʵ�
  - ���������ռ�
    - ��ȡ����������ȡ������������ģ��
  - �������²���
    - ��ÿ��ѵ�����ݣ�
      - ��ȡ�������ֱ��մ𰸵ı�Ǻ���ȡ���������н���
      - ͳ�ƽ������ĸ���������������µĲ����������д�������صĲ������и���
      - pa����
      - ap����
      - ����ͳ���������µĴ���
      - �Ը��´����ٵ�����вü�
    - ������Լ���׼ȷ��
      - p = (double)num_recalled_words / num_predicted_words;
      - r = (double)num_recalled_words / num_gold_words;
      - f = 2 * p * r / (p + r);
    - ��ǰf����֮ǰ���ŵ�f������ģ��

## ��������
- tid            ģ��id
- itx            ����id


# ���Ա�ע
##   �� ���������в���
  

���� | ��;
---|---
model | ģ���ļ�ǰ׺
reference | ѵ���ļ�
development | �����ļ�
algorithm | �㷨 pa��ap
max-iter | ����������
rare-feature-threshold | ϡ�������ü���ֵ
dump-details | �Ƿ��ģ��д����ϸ��Ϣ������������ѵ��


# ����ʵ��ʶ��
##   �� ���������в���
���� | ��;
---|---
model | ģ���ļ�ǰ׺
reference | ѵ���ļ�
development | �����ļ�
algorithm | �㷨 pa��ap
max-iter | ����������
rare-feature-threshold | ϡ�������ü���ֵ
dump-details | �Ƿ��ģ��д����ϸ��Ϣ������������ѵ��

# ��¼������
## pa����

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

## ap����
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

