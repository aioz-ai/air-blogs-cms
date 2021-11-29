---
last_modified_on: "2021-11-29"
title: Video recognition and categorization (Part 1 - Introduction)
description: A review about video categorization.
series_position: 6
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: easy", "guides: video_category"]
---
The internet is now widely utilized by individuals all around the world. Social media plays an important role in content dissemination (audio, video, text, and picture). During the same time period, people also post their emotions on social media about a certain component so that other users can rapidly learn what is going on, and as a result, user views are used to estimate public opinion on specific topics. However, hiring someone to analyze people's opinions based on a large amount of material is tough and time intensive. The researchers provide a machine learning technique to data mining in order to assess public sentiments. Video classification is a type of mining that examines text using natural language processing and video using machine linguistics in order to discover people's points of view by gathering and evaluating social and other subjective knowledge resources.

![Fig-1](https://pyimagesearch.com/wp-content/uploads/2019/07/keras_video_classification_sports_dataset.jpg)  
*<center>**Figure 1**:  Examples of video categorization [(Source)](https://www.pyimagesearch.com/2019/07/15/video-classification-with-keras-and-deep-learning/).</center>*

Nowaday, we can automatically extract the categories of videos using AI, which may be used for a variety of applications or as input for other processes. As a sequence of pictures with sufficient motion, a video file comprises audio, text, and visual material in frames. To achieve efficient video classification, all accessible resources must be used and analyzed. As a result, in order to get a satisfactory result, a video categorization system must investigate all of the video's main features, such as text, audio, and frames. Modalities are the names given to these components. The method should deliver the text, audio, and visual content analysis findings, which are then merged to produce the final video category output.

## Aproaches

There are three main groups:

* Text-based approaches.
* Audio-based approaches.
* Visual-based approaches.

#### Text-based approaches

This is the least commom approach for video classification. We collect text from video and evaluate it for categorization. The text based approach is further divided into two types:

- **Text on the screen**. The text shown on the screen is retrieved in the first category. For example, a game's scoreboard, a player's number on her shirt, or subtitles displayed on the screen. Optical Character Recognition (OCR) can be used to extract such text.

![Fig-2](https://i.ytimg.com/vi/I-JwpyTOsaY/maxresdefault.jpg)  
*<center>**Figure 2**: A example of a video has the text on screen. [(Source)](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DI-JwpyTOsaY&psig=AOvVaw12-sufqh2s-cfpLDbEUjGL&ust=1638095827689000&source=images&cd=vfe&ved=0CAwQjhxqFwoTCLiLztesuPQCFQAAAAAdAAAAABAD).</center>*

- **A transcript of the speech**. Voice recognition is used to extract text from speech in the second category. This approach is mostly used to provide closed captions or subtitles. Closed captions are typically utilized to give additional sounds, such as animal sounds or music. Subtitles are displayed on the screen to aid comprehension in a common language.

Text-based approaches have the benefit of utilizing the enormous body of research on document text categorization. The main benefit is that people can easily recognize the relationship between words and specific genres. However, the text obtained via OCR of on-screen text has quite high mistake rates, making it difficult to use text-based functionality. While the number of closed captions for a movie reaches a certain limit.

#### Audio-based approaches

This approach is employed more frequently than text-based analysis. Because there is a fact that audio processing takes less time and energy. Audio takes up less storage space than video and text. A single signal is sampled for audio processing, and some characteristics are collected for each sample's examination. These samples may be overlapping in some cases. The functions might be in the time domain or in the frequency domain.

Audio features can be derived by:

- Time domain. The features of time domain are further divided into two parts:
    - Root Mean Square (RMS) of the signal energy approximates the human perception of the loudness or volume of a sound.
    - Zero Crossing Rate (ZCR) is the number of signal amplitude sign changes in the current frame.

![Fig-3](https://vision.aioz.io/thumbnail/20491a3f957e45d1a2ca/1024/frequency_domain.PNG)  
*<center>**Figure 3**: An example of the time domain, in which the amplitude of a signal is plotted with respect to time [(Source)](https://eecs.wsu.edu/~cook/pubs/smc08.pdf).</center>*

- Frequency domain. The signal distribution across frequency components is known as the energy distribution. The Frequency Centroid, which is the midpoint of the spectral energy distribution and offers a measure of where the frequency components are concentrated, is the midpoint of the spectral energy distribution and approximates brightness.

![Fig-4](https://vision.aioz.io/thumbnail/6807cacde0d44096964e/1024/time_domain.PNG)
*<center>**Figure 4**: Using the Fourier transform, a signal in the time domain can be transformed to the frequency domain, also known as the spectrum of the signal [(Source)](https://eecs.wsu.edu/~cook/pubs/smc08.pdf).</center>*

#### Visual-based approaches

The technique was utilized by the majority of researchers since most knowledge based on vision is interpreted by humans. Some authors have combined these visual elements, including music and text, as required. Image sequences or video files are the primary sources of visual capacity. A vital aspect of video is its basic structure, which is similar to a compilation of photographs. A collection of frames is another word for video. Color, motion, and shooting duration are all factors that influence visual features. These elements must transmit information about lighting, movement, backdrop, and video speed.  Video based approach is further divided into 3 parts:

- Color Based.
- Shot Based.
- Object Based.

In the next blogs, we will analysis futher about **datasets and deep learning approaches** of the video classification task.

