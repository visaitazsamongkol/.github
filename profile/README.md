## Problem Statement

Nowadays, computer vision has made a huge leap into several fields and we use data from images to provide value for ourselves. Apps like Google Translate use computer vision to translate text from the image. Based on that, we proposed an app for learning language (English) on the go. This application will take for input the taken image from the mobile phone and through optical character recognition (OCR), find all words in the image and provide users with definitions and examples from the dictionary API.

<hr/>

## Technical Challenges

Keras-ocr could provide satisfactory results for a text recognition task. The model already outputs each word separately and thus is suitable for our work. Nonetheless, the model cannot recognize punctuation, as described on the GitHub page of keras-ocr.

<hr/>

## Related Works
	
### Application 

- Yomiwa: A Japanese dictionary application that gives definition and usage examples of the selected word from a taken photo. This app has the same purpose as our app but they only provide Japanese to English translation.

- Google Translate: A translator application that can translate the text as sentences from a taken photo. The app focuses on translating whole sentences, not giving information of a single word.

### OCR Model

- TesseractOCR: An open source Apache project that gives both localization and recognition for each word found in the image. It works well in a well-lighted condition and on words that align in straight lines, also works better for English document images. Tesseract model does not work well with scene text images.

- PaddlePaddleOCR: An ultra-lightweight OCR system that supports a lot of language recognition. It works well with a sentence or a long text but it’s not the best option when we want a separate word.

<hr/>

## Method

### Model

The keras-ocr model mainly comprises 2 submodules: a text detection module and a text recognition module.

1. Text Detection

   The text detection module of keras-ocr is the CRAFT text detection model [1]. The architecture is encoder-decoder where the model outputs region score and affinity. Next, traditional computer vision methods are used to detect the word regions.

2. Text Recognition
 
   The architecture of the text recognition model is CRNN [2]. The input image is a cropped image containing a word region which will be passed into convolutional layers to extract feature and output feature maps. Then, The feature maps are vertically partitioned into sequential data and are inserted into bidirectional LSTM layers. Finally, the output from LSTM will be processed and transcripted into a word.

In order to train the model such that it can recognize punctuation, we replaced the head of the text recognition module with a new classification head and trained only the new inserted head.

We trained the model for 127 epochs with RMSProp as the optimizer, CTCLoss as the loss function, learning rate of 10-3, batch size of 224, and ReduceLROnPlateau as the learning rate scheduler with patience of 10, and STRAug [3] as augmentation.

The images we used for the training are generated from background randomly selected from a background pool consisting of about one thousand background images. After that, randomly generated words are drawn onto the background images to create artificial scene text images. 

### Dictionary APIs

We used two different types of Merriam-webster APIs, Collegiate® Dictionary with Audio and Collegiate® Thesaurus, for looking up each word. We got information about meanings, sentence examples, part of speech, pronunciation and audio for Collegiate Dictionary, and additional synonyms, antonyms but no pronunciation and audio for Collegiate Thesaurus.

### Application

We used the Flutter framework in Dart programming language to develop our application due to its popularity, speed, and cross-platform ability. The dictionary APIs were also developed in Dart and run on the user’s device. The user interface is divided into 3 parts: homepage, dictionary search page, and each word definition result page.

<hr/>

## Evaluation

We evaluated the trained recognizer on 3 datasets using Word Recognition Accuracy metric calculated from the equation

![formula](https://render.githubusercontent.com/render/math?math=WRA=\frac{W_r}{W})

&nbsp;&nbsp;&nbsp;&nbsp; Where `WRA` is the Word Recognition Accuracy,

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `W`<sub>`r`</sub> is the number of words predicted correctly,

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `W` is the total number of words in the dataset.

The accuracy of the recognizer is shown below

| Dataset      | Original | Trained |
|--------------|----------|---------|
| ICDAR2013    | 0.6      | 0.775   |
| Born Digital | 0.5      | 0.748   |
| Coco Text    | 0.304    | 0.365   |

<hr/>

## Discussion and Further work

From the evaluation results, the trained recognizer performs better than the original recognizer of keras-ocr on all text datasets and can recognize certain punctuations.

For further work, we consider deploying the model on edge devices. As Flutter does not support the ONNX model and deploying via Tensorflow Lite involves immense difficulties, we consider switching from Flutter to other frameworks or tools such as Android SDK or Swift. Other than that, we may change the model’s architecture to a lighter one such as MobileNet to reduce resource consumption and to be more fit for edge devices or we may go for more accuracy by using modern variances of models such as Vision Transformer.

<hr/>

## References

[1]	Y. Baek, B. Lee, D. Han, S. Yun, and H. Lee, “Character Region Awareness for Text Detection,” ArXiv190401941 Cs, Apr. 2019, Accessed: Apr. 05, 2022. [Online]. Available: http://arxiv.org/abs/1904.01941http://arxiv.org/abs/1507.05717

[2]	B. Shi, X. Bai, and C. Yao, “An End-to-End Trainable Neural Network for Image-based Sequence Recognition and Its Application to Scene Text Recognition,” ArXiv150705717 Cs, Jul. 2015, Accessed: Apr. 12, 2022. [Online]. Available: http://arxiv.org/abs/1507.05717

[3]	R. Atienza, “Data Augmentation for Scene Text Recognition,” ArXiv210806949 Cs, Aug. 2021, Accessed: Apr. 13, 2022. [Online]. Available: http://arxiv.org/abs/2108.06949
