# Neural Turing Machine

This repository contains a stable, successful Tensorflow implementation of a Neural Turing Machine which has been tested on the Copy, Repeat Copy and Associative Recall tasks from the [original paper](https://arxiv.org/abs/1410.5401).

The implementation is based on: https://github.com/snowkylin/ntm but contains some substantial modifications. Most importantly, I compare three different memory initialization schemes and find that initializing the memory contents of a Neural Turing Machine to small constant values works much better than random initilization or backpropagating through memory initialization. I never see loss go to NaN as some other implementations report.

The NTMCell implements the [Tensorflow RNNCell interface](https://www.tensorflow.org/api_docs/python/tf/contrib/rnn/RNNCell) so can be used directly with [tf.nn.dynamic_rnn](https://www.tensorflow.org/api_docs/python/tf/nn/dynamic_rnn), etc.

### Paper

For a description of our implementation and experimental results please see the pre-print of our paper which will appear at ICANN 2018: https://arxiv.org/abs/1807.08518

## Usage

```python
from ntm import NTMCell

cell = NTMCell(num_controller_layers, num_controller_units, num_memory_locations, memory_size,
    num_read_heads, num_write_heads, shift_range=3, output_dim=num_bits_per_output_vector,
    clip_value=clip_controller_output_to_value)

outputs, _ = tf.nn.dynamic_rnn(
    cell=cell,
    inputs=inputs,
    time_major=False)
```

## Sample Outputs

Below are some sample outputs on the Copy and Associative Recall tasks. I replicated the hyperparameters from the [original paper](https://arxiv.org/abs/1410.5401) for the 2 tasks:

- Memory Size: 128 X 20
- Controller: LSTM - 100 units
- Optimizer: RMSProp - learning rate = 10^-4

The Copy task network was trained on sequences of length sampled from Uniform(1,20) with 8-dimensional random bit vectors. The Associative Recall task network was trained on sequences with the number of items sampled from Uniform(2,6) each item consisted of 3 6-dimensional random bit vectors.

#### Example performance of NTM on Copy task with sequence length = 20 (output is perfect):
![Neural Turing Machine Copy Task - Seq len=20](/img/copy_ntm_20_0.png)

#### Example performance of NTM on Copy task with sequence length = 40 (network only trained on sequences of length up to 20 - performance degrades on example after 36th input):
![Neural Turing Machine Copy Task - Seq len=40](/img/copy_ntm_40_1.png)

#### Example performance of NTM on Associative Recall task with 6 items (output is perfect):
![Neural Turing Machine Associate Recall Task - Seq len=6 items](/img/associative_recall_ntm_6_0.png)

#### Example performance of NTM on Associative Recall task with 12 items (despite only being trained on sequences of up to 6 items to network generalizes perfectly to 12 items):
![Neural Turing Machine Associate Recall Task - Seq len=12 items](/img/associative_recall_ntm_12_0.png)

In order to interpret how the NTM used its external memory I trained a network with 32 memory locations on the Copy task and graphed the read and write head address locations over time.

As you can see from the below graphs, the network first writes the sequence to memory and then reads it back in the same order it wrote it to memory. This uses both the content and location based addressing capabilities of the NTM. The pattern of writes followed by reads is what we would expect of a reasonable solution to the Copy task.

#### Write head locations of NTM with 32 memory locations trained on Copy task:
![Write head locations of NTM with 32 memory locations trained on Copy task](/img/ntm_copy_write_head.png)

#### Read head locations of NTM with 32 memory locations trained on Copy task:
![Read head locations of NTM with 32 memory locations trained on Copy task](/img/ntm_copy_read_head.png)

Further results on memory initilization comparison and learning curves to come...
