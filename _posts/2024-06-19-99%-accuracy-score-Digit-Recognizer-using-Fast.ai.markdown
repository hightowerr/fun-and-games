---
layout: posts
title:  "99% accuracy on Digit Recognizer problem using Fast.ai"
header:
  image: /assets/images/markus-krisetya-Vkp9wg-VAsQ-unsplash.jpg
  og_image: /assets/images/markus-krisetya-Vkp9wg-VAsQ-unsplash.jpg
date:   2024-06-19 09:00:00 +0100
categories: GPU, PyTorch, Image Classification, Data Augmentation
author: Olayinka Ola
---
This is the first of many notebooks on my way to complete the [fastai course](https://course.fast.ai/). I'll use Kaggle as a playground to test my knowledge after each lessons.

MNIST ("Modified National Institute of Standards and Technology") is the de facto “hello world” dataset of computer vision. The  goal is to correctly identify digits from a dataset of tens of thousands of handwritten images.

My solution uses data augmentation, presizing and Discriminative Learning Rate to achieve 99% accuracy. My kaggle notebook can be found [here](https://www.kaggle.com/code/madcontender/mnist-simple-fastai-visionlearner)

## Ingredients

* Presizing - Presizing is the name given to the techniques of resizing and augmenting the data before feeding it to the machine learning models.
* Discriminative Learning Rate - Discriminative learning rates refers to the training trick of using different learning rates for different layers of the model.
* Mixup - Mixup creates synthetic training examples by taking a weighted linear combination of two random samples and their corresponding labels.

## Key Directions

1. Import libraries

```python
from fastai.vision.all import *
matplotlib.rc('image', cmap='Greys')
```
2. Read in the CSVs, and transform the data into a usable format for training, testing and inference with Pytorch

```python

def get_image(row):
    image_data = np.array(row[:28*28]) # Convert to numpy array
    image_data = image_data.reshape(28, 28) # Reshape to 28x28
    return torch.tensor(image_data).float() # Convert to torch tensor
    
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Pasted image 20240615081012.png" alt="Learning rate finder">

3. . Configure the 'DataBlock' for a consistent and structured processing and data preprocessing. item_tfms & batch_tfms are particularly important for a data augmentation technique called [[Presizing]] which can help improve the performance of image classification problems

```python
mnist_block = DataBlock(
    blocks=(ImageBlock(cls=PILImageBW), CategoryBlock), 
    get_x=get_image,
    get_y=(lambda i: i['label']),
    splitter=RandomSplitter(valid_pct=0.2, seed=69),
    item_tfms=Resize(size = 460),
    batch_tfms=(aug_transforms(size = 128, do_flip=False, max_zoom=1.25))
)
```

4. Add in the architecture, the data and a metric to validate performance. In this project MixUp is used to help make the model more robust to overfitting.

```python
learn = vision_learner(dls, resnet50, metrics=accuracy, cbs = MixUp(0.8)).to_fp16()
lr_min,lr_steep = learn.lr_find(suggest_funcs=(minimum, steep))
```

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Pasted image 20240615080608.png" alt="Learning rate finder">

5. To improve the performance of Resnet50 for new tasks like predicting handwritten digits, the discriminative learning rate technique is used to adjust the base layers while fine-tuning the later layers.

```python
learn.fit_one_cycle(2, 3e-2)
learn.unfreeze()
learn.fit_one_cycle(20, lr_max=slice(1e-5, 1e-2))
```

|epoch|train_loss|valid_loss|accuracy|time|
|---|---|---|---|---|
|0|0.980438|0.150517|0.973571|01:37|
|1|0.794551|0.091658|0.985357|01:37|

| epoch | train_loss | valid_loss | accuracy | time  |
| ----- | ---------- | ---------- | -------- | ----- |
| 0     | 0.742655   | 0.075108   | 0.989524 | 01:47 |
| 1     | 0.694828   | 0.068573   | 0.990833 | 01:46 |
| 2     | 0.649477   | 0.054868   | 0.990476 | 01:45 |
| 3     | 0.620499   | 0.061068   | 0.989405 | 01:44 |
| 4     | 0.603707   | 0.045559   | 0.992024 | 01:45 |
| 5     | 0.582042   | 0.050043   | 0.991310 | 01:47 |
| 6     | 0.558974   | 0.027005   | 0.995357 | 01:45 |
| 7     | 0.555178   | 0.024780   | 0.996071 | 01:45 |
| 8     | 0.541447   | 0.023236   | 0.995833 | 01:45 |
| 9     | 0.522384   | 0.028931   | 0.993571 | 01:45 |
| 10    | 0.514447   | 0.020091   | 0.996310 | 01:45 |
| 11    | 0.503431   | 0.020571   | 0.996310 | 01:45 |
| 12    | 0.492349   | 0.024063   | 0.995714 | 01:45 |
| 13    | 0.485419   | 0.014886   | 0.996429 | 01:45 |
| 14    | 0.484940   | 0.017684   | 0.996548 | 01:45 |
| 15    | 0.469177   | 0.015092   | 0.997262 | 01:46 |
| 16    | 0.464866   | 0.015254   | 0.997143 | 01:45 |
| 17    | 0.470034   | 0.015710   | 0.997500 | 01:46 |
| 18    | 0.467691   | 0.014679   | 0.997143 | 01:46 |
| 19    | 0.460081   | 0.015294   | 0.997262 | 01:44 |

6. The performance can be visualised using a confusion matrix. Overall very good performance. TThe model struggled most with predicting 9

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/Pasted image 20240615082156.png" alt="Confusion Matrix">

### Extensions
- Progressive Resizing - Gradually using larger and larger images as you train.
- Test-time Augmentation - Creating multiple versions of each image using data augmentation during inference or validation then averaging/maximising for prediction.