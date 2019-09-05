# Model and Neural Network Architecture

After the data preperation and vetting, comes the essence of the project at hand, designing
a model that fits our dataset. Due to the small number of training pool, we had to choose
an architecture that can work efficiently and with high precision on small datasets.

The architecture fitting that particular criteria was the _U-Net Architecture_.

<div align="center">
<img src = "media/u-net-architecture.png">
</div>

The main idea is to supplement a usual contracting network by successive layers,
where pooling operators are replaced by upsampling operators and eventually these
layers increase the resolution of the output. To be able to localize, high resolution features
from the contracting path are combined with the upsampled output.

P.S: Inquire about the excessive data augmentation (Pg. 3)

The U-Net architecture is composed of two main branches as shown in the figure above. These
branches are:

- Contracting Path
- Expansive Path

## Contracting Path

- Repeated application of two 3x3 convolutions.
- Each convolution is followed by a ReLu and 2x2 max-pooling operation with stride 2 for downsampling.
- Every downsampling step, feature channels are doubled.

## Expansive Path

- Every step is composed of upsampling of feature map followed by 2x2 convolution halving feature channels.
- Concatenation of halved channels with corresponding cropped feature map from the contracting path.
- Two 3x3 convolutions each followed by ReLu
- A final 1x1 convolution is used to map each component feature vector to number of classes.

## References

- Ronneberger O., Fischer P., et al [U-Net: Convolutional Networks for Biomedical Image Segmentation](https://arxiv.org/pdf/1505.04597.pdf)
