---
title: "Convoluted Neural Network Models for Environmental Sound Classification"
description: "A description on using verification techniques and UVM to validate a basic transmission/receiving module"
date: "2025-11-20"
author: "Vishnu Ramesh"
math: true
---

## Abstract

This project investigated various data processing and model development options for analyzing the FSDnoisy18k dataset, which provides a large number of samples across twenty categories of sounds. A CNN model was developed, as well as a CNN and GRU hybrid model. Both models were duplicated, with one being trained on the clean data, and the other being trained on the full dataset. Comparing the results of the four models showed that the hybrid model trained on the full dataset had the highest performance, with an overall test accuracy of 66%. These results line up with the original FSDnoisy18k paper, and highlight the importance of sample size when training a model, as well as the benefits of incorporating temporal modeling through GRU layers.

## Environmental Audio Classification

Audio classification is a growing field that has garnered interest from both researchers and businesses. Advancements in machine learning and data analysis techniques have led to increased performance for audio classification, and these advancements can help businesses in the automotive, medical, and other fields.

This paper looks at the FSDnoisy18k dataset, curated by Eduardo Fonseca and Mercedes Collado. This dataset is an interesting consolidation of clips taken from Freesound, an online website where users can upload audio clips with labels. These clips are environmental sounds, examples including 'Clapping', 'Fire', and 'Wind'. There are over twenty different classes in this dataset. However, only 10% of the data is 'clean' data; that is, data that is manually verified to have the correct label. The [introductory paper](https://ieeexplore.ieee.org/document/8683158) introduced by Eduardo Fonseca brings the idea that a large, noisy dataset is much better than a small, clean dataset. More information about the dataset can be found on [Fonseca's companion page](https://www.eduardofonseca.net/FSDnoisy18k/).

## Machine Learning for Audio Classification

Instead of passing in the raw audio clips, it is best to convert each clip into a Mel spectrogram. A Mel spectrogram will take the audio clip, and convert it into an image where time runs horizontally, frequency runs vertically, and color intensity represents the loudness of different frequencies. This image will capture the major features of the audio clip, while dramatically reducing the computing power needed to train a machine learning model. In addition, images with distinct spatial information can be easily fed into a Convolutional Neural Network, which takes into account spatial information of an image when performing either classification or regression tasks.

![Example MEL Spectrogram](/images/mel_spectrogram.png)

This paper explores two different models with two different subsets of the FSDnoisy18k dataset. First, both a CNN model and a CNN-GRU model were tested. The CNN model was a baseline model, while the CNN-GRU model introduced a more complex architecture in order to capture better temporal patterns in the audio. As can be seen in the image above, not only is it important to capture spatial features, but also how these features appear across time. The performance of these two models can be seen below.

| Model | Dataset | Validation Accuracy |
| :--- | :--- | :--- |
| CNN Baseline | Clean | 46% |
| CNN Baseline | Clean + Dirty | 46% |
| CNN + GRU | Clean | 49% |
| CNN + GRU | Clean + Dirty | 62% |

The 'Clean + Dirty' subset contains both the manually verified clips, as well as audio clips that were labeled by the Freesound community, not manually verified. This dataset was also about 10x larger than just the dataset containing clean clips. Interestingly, adding in both the dirty data into the CNN+GRU model led to the highest test accuracy of about 66%. 

## Conclusion

This paper went through the processing of audio clips into Mel spectrograms, as well as the training of both a CNN and a CNN+GRU model to perform classification. This research also showcases usefulness in its practical implications. This model can be trained relatively quickly, and can be used as a baseline for businesses to quickly identify noises of interest. Future work may include testing more advanced CNN architectures, or trying out new models such as a transformer model.

A full length project report is available, as well as a Github repository with the codebase containing the machine learning models and test analysis.




