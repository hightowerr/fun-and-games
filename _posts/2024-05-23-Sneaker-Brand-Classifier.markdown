---
layout: posts
title:  "Sneaker brand Classifier"
date:   2024-05-23 09:00:00 +0100
categories: python, Git, Gradio, HuggingFace, Mamba
author: Olayinka Ola
---

This project allows automatic identification of sneaker brands from uploaded images, demonstrating an example application of deep learning.

---

<iframe
	src="https://hightowerr-trainers-mimal.hf.space"
	frameborder="0"
	width="850"
	height="450"
></iframe>


[https://hightowerr-trainers-mimal.hf.space](https://hightowerr-trainers-mimal.hf.space)

### **The process**

---

In this deep learning project, I set up all the necessary tools and libraries, including FastAI and CUDA. Then, I got an API key to access the Bing search API and started grabbing sneaker images from Nike, Adidas, and Puma.

Once the images were collected, I meticulously organized them into the appropriate folders, ensuring a structured and systematic approach. I also set up the data handling process for training, which involved loading, splitting, and labelling the images.

For training the model, I introduced a pre-trained ResNet18 model to my sneaker dataset. Post-training, I evaluated the model's performance using visual tools like confusion matrices and loss plots. These tools provided comprehensive insights into the model's accuracy and areas for improvement.

Each of these steps is absolutely essential for creating a top-simple image classification model. They show how easy it is to deploy powerful deep-learning tools to solve interesting problems.

To use the classifier, you can upload a picture of Nike, Adidas, or Puma sneakers, hit submit, and watch the model predict which brand you've uploaded. Not bad, right?

Adapted from - [Fastai tutorial](https://www.youtube.com/watch?v=F4tvM4Vb3A0&t=3424s)

#### **Future Extensions**

- Train a model to classify the line of sneakers; for example, Nike Dunks are a product line of the Nike brand.

[Link to code:](https://drive.google.com/file/d/1d5jkWoQTG-W9NCle3qdYyVlNPfsnWWOX/view?usp=sharing)

---

### Useful Links
[My Hugging face Space](https://huggingface.co/spaces/hightowerr/trainers_mimal)
