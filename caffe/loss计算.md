caffe �� loss ��ʼ�������㡢�Լ��Բ����ĸ���
---
���Խ׶�û�в����ĸ���(Ҳ����û��BP)�����ԶԾ�����Ӱ��

# 1 ��ʼ��
��loss�й��׵Ķ��������д���loss���۵Ĳ㣬�Դ�Ϊ���Ŀ�ʼ��Layer��ĳ�ʼ����  
```
template <typename Dtype>
class Layer {
	/**
   * @brief Implements common layer setup functionality.
   *
   * @param bottom the preshaped input blobs
   * @param top
   *     the allocated but unshaped output blobs, to be shaped by Reshape
   *
   * Checks that the number of bottom and top blobs is correct.
   * Calls LayerSetUp to do special layer setup for individual layer types,
   * followed by Reshape to set up sizes of top blobs and internal buffers.
   * Sets up the loss weight multiplier blobs for any non-zero loss weights.
   * This method may not be overridden.
   */
  void SetUp(const vector<Blob<Dtype>*>& bottom,
      const vector<Blob<Dtype>*>& top) {
    CheckBlobCounts(bottom, top);
    LayerSetUp(bottom, top);  // �麯�� ����̳�ʵ��
    Reshape(bottom, top);
    SetLossWeights(top);  // ��¼ ���� top blob �� loss_weight
  }
  /**
   * @brief Does layer-specific setup: your layer should implement this function
   *        as well as Reshape.
   *
   * @param bottom
   *     the preshaped input blobs, whose data fields store the input data for
   *     this layer
   * @param top
   *     the allocated but unshaped output blobs
   *
   * This method should do one-time layer specific setup. This includes reading
   * and processing relevent parameters from the <code>layer_param_</code>.
   * Setting up the shapes of top blobs and internal buffers should be done in
   * <code>Reshape</code>, which will be called before the forward pass to
   * adjust the top blob sizes.
   */
  virtual void LayerSetUp(const vector<Blob<Dtype>*>& bottom,
      const vector<Blob<Dtype>*>& top) {}

  /**
   * @brief Sets the loss associated with a top blob at a given index.
   */
  inline void set_loss(const int top_index, const Dtype value) {
    if (loss_.size() <= top_index) {
      loss_.resize(top_index + 1, Dtype(0));
    }
    loss_[top_index] = value;
  }
  /** The vector that indicates whether each top blob has a non-zero weight in
   *  the objective function. */
  vector<Dtype> loss_;
  // ĳ��top blob �� loss_weight ��Ϊ0�Ļ� �ͱ���������

  /**
   * Called by SetUp to initialize the weights associated with any top blobs in
   * the loss function. Store non-zero loss weights in the diff blob.
   */
  inline void SetLossWeights(const vector<Blob<Dtype>*>& top) {
    const int num_loss_weights = layer_param_.loss_weight_size();
    if (num_loss_weights) {
      CHECK_EQ(top.size(), num_loss_weights) << "loss_weight must be "
          "unspecified or specified once per top blob.";
      for (int top_id = 0; top_id < top.size(); ++top_id) {
        const Dtype loss_weight = layer_param_.loss_weight(top_id);
        if (loss_weight == Dtype(0)) { continue; }
        this->set_loss(top_id, loss_weight);
        const int count = top[top_id]->count();
        Dtype* loss_multiplier = top[top_id]->mutable_cpu_diff();
        // loss layer �� top diff ����Ϊ1
        caffe_set(count, loss_weight, loss_multiplier);
      }
    }
  }
}
```
�� SoftmaxWithLossLayer Ϊ�������̳��� LossLayer
```
template <typename Dtype>
void LossLayer<Dtype>::LayerSetUp(
    const vector<Blob<Dtype>*>& bottom, const vector<Blob<Dtype>*>& top) {
  // LossLayers have a non-zero (1) loss by default.
  if (this->layer_param_.loss_weight_size() == 0) {
    this->layer_param_.add_loss_weight(Dtype(1));  // ��� loss_weight 1
  }
}
```
Ҳ����˵�������� LossLayer���� ���࣬���ǵ� loss_weight_size() ��0

# 2 ����
Net forward �Ĺ����л��ۼ� loss�����Ͳο�����1�����һ�£�α��������
```
loss := 0
for layer in layers:
  for top, loss_weight in layer.tops, layer.loss_weights:
    loss += loss_weight * sum(top)
```
caffe �� train ���̣����Ĺ��̴� solver.solve() ��ʼ
����  
``` 
void Solver<Dtype>::Solve(const char* resume_file)
{ 
  Step(param_.max_iter() - iter_); // line 286
}

void Solver<Dtype>::Step(int iters)
{
  // solver.cpp, line 209
  for (int i = 0; i < param_.iter_size(); ++i) {
      loss += net_->ForwardBackward();
  }
  // iter_size ���� caffe -iterations ����
    loss /= param_.iter_size();
  // average the loss across iterations for smoothed reporting
  UpdateSmoothedLoss(loss, start_iter, average_loss)
}
```

```
message SolverParameter {
  ...
// accumulate gradients over `iter_size` x `batch_size` instances
  optional int32 iter_size = 36 [default = 1];
  ...
```

ForwardBackward = forward + backward
����
```
template <typename Dtype>
Dtype Net<Dtype>::ForwardFromTo(int start, int end) {
  Dtype loss = 0;
  for (int i = start; i <= end; ++i) {
    Dtype layer_loss = layers_[i]->Forward(bottom_vecs_[i], top_vecs_[i]);
    loss += layer_loss;
  }
  return loss;
}

/// top_vecs stores the vectors containing the output for each layer
  vector<vector<Blob<Dtype>*> > top_vecs_;

template <typename Dtype>
inline Dtype Layer<Dtype>::Forward(const vector<Blob<Dtype>*>& bottom,
    const vector<Blob<Dtype>*>& top) {
	Forward_cpu(bottom, top);
    for (int top_id = 0; top_id < top.size(); ++top_id) {
      if (!this->loss(top_id)) { continue; }
      const int count = top[top_id]->count();
      const Dtype* data = top[top_id]->cpu_data();
      const Dtype* loss_weights = top[top_id]->cpu_diff();
      loss += caffe_cpu_dot(count, data, loss_weights);
    }
}
```

Layer.loss �ĺ���
```
/** The vector that indicates whether each top blob has a non-zero weight in
   *  the objective function. */
  vector<Dtype> loss_;
```
���� loss_weight
```
message LayerParameter {
// The amount of weight to assign each top blob in the objective.
  // Each layer assigns a default value, usually of either 0 or 1,
  // to each top blob.
  repeated float loss_weight = 5;
}
```



net.cpp �� init ������������һ��ע��
```
// Go through the net backwards to determine which blobs contribute to the
// loss.  We can skip backward computation for blobs that don't contribute
// to the loss.
// Also checks if all bottom blobs don't need backward computation (possible
// because the skip_propagate_down param) and so we can skip bacward
// computation for the entire layer
```

type����loss�Ĳ� top blob ��ͼ�ΪĿ�꺯��  
�м��Ҳ���Լ� loss


# �ο�
1. Caffeѧϰ��Loss  
<http://blog.csdn.net/u011762313/article/details/47356285>  
2. Caffe�ٷ��̳����뱾v1.0  
<http://caffecn.cn/?/page/tutorial>  
<http://pan.baidu.com/s/1c0Ri2Py>  
2. caffe��loss�����������--caffeѧϰ��16��  
<http://blog.csdn.net/u014381600/article/details/54340613>  