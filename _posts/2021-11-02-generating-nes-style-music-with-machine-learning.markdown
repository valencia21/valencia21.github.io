---
layout: post
title:  "Generating NES-style Music with Machine Learning"
date:   2021-11-02 15:00:00 +0900
categories:
---

<script src="https://cdn.jsdelivr.net/combine/npm/tone@14.7.58,npm/@magenta/music@1.22.1/es6/core.js,npm/focus-visible@5,npm/html-midi-player@1.4.0"></script>

## Objective

For this project, I wanted to generate NES-style music using some form of machine learning.

## Planning

### Neural Network Architectures for Music

Music mainfests certain properties that exist ubiquitously across pieces and genres. These features include structure (such as ABA), coherence, self-referencing, and the development of previous phases to elicit a specific response from the listener.

These properties relate to the persistence of some event throughout a single piece - the existence of an event informs later events. A neural network that generates new music well needs to information from events to persist and develop.

While not strictly necessary to produce an output for this project, I wanted to take the opportunity to understand the types of architecture that would be best suited to music generation.

#### [Recurrent Neural Networks (RNNs)](https://www.ibm.com/cloud/learn/recurrent-neural-networks)

An RNN is the most basic architecture for allowing information to persist. The output of each layer is input into the next, allowing for information from previous inputs to influence the output.

{:refdef: style="text-align: center;"}
![RNN Structure](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2021-11-02/ICLH_Diagram_Batch_02_13B-RecurrentNeuralNetworks-WHITEBG.webp){: style="text-align:center"}
{: refdef}

#### [Long Short-Term Memory Networks (LSTMs)](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)

LSTMs were created to solve the problem that previous states have decreasing influence as they get further from a present prediction. 

For example, consider trying to predict the last words in the sentence "Alice is allergic to nuts.......She can't eat __peanut butter__." The further back the previous sentence, the harder it is for a NN to understand the context.

An LSTM has "cells" containing an input, output and forget gate to control the flow of information that is important to creating a prediction. 

{:refdef: style="text-align: center;"}
![RNN Structure](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2021-11-02/LSTM3-chain.png){: style="text-align:center"}
{: refdef}

An explanation of LSTM cells is beyond the scope of ths blog post, but those interested can read more [here]((https://colah.github.io/posts/2015-08-Understanding-LSTMs/)).

Google Magenta's [Melody RNN](https://github.com/magenta/magenta/blob/main/magenta/models/melody_rnn/README.md) is an example of an LSTM built for music.

#### [Transformers](https://jalammar.github.io/illustrated-transformer/)

Transformers use the concept of "self-attention" to persist event information across longer gaps. In self-attention, each event (i.e. a word in a sentence) is given a Query vector, a Key vector and a Value vector. In the word example, the amount of attention to be paid to a particular word from the perspective of another word in the sentence is calculated by taking the dot product of the query vector for the particular word with the key vector for other words in the input sentence. This is called the self-attention score.

{:refdef: style="text-align: center;"}
![RNN Structure](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2021-11-02/transformer_self-attention_visualization.png){: style="text-align:center"}
{: refdef}

After scores are divided by 8 to obtain more stable gradients, self-attention scores are then passed through a Softmax operation so all scores a positive and add up to 1. This gives the probability that one word should pay attention to another.

There is [a paper](https://openreview.net/pdf?id=rJe4ShAcF7) on building a Music Transformer, with [example usage](https://magenta.tensorflow.org/nobodys-songs).

#### [Variational Autoencoders (VAE)](https://towardsdatascience.com/understanding-variational-autoencoders-vaes-f70510919f73)

Another type of neural net architecture of note is the Variational Autoencoder, which is typically used to compose deep generative models. While they lack the ability to persist information from events, they are far more apt at extracting detailed features from data and generating an output that corresponds to that level of detail.

One way of predicting and generating an output given a mass of input data is to encode the features of the input through dimensionality reduction, decode to some output, measure the loss in the decoded output relative to the input data and adjust the encoder/decoder combination accordingly. This is known as an "autoencoder" architecture.

The issue in this approach is that the encoding sits in a latent space, and decoding a random point from that latent space is unlikely to output anything similar to the input.

{:refdef: style="text-align: center;"}
![VAE Decoder Problem](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2021-11-02/vae_1.png){: style="text-align:center"}
{: refdef}

In a VAE, the latent space is regularized during the training process to have two properties:
- **Continuity:** Two points that are close in the latent space should be similar in their output upon decoding.
- **Completeness:** Any point in the latent space should give a meaningful output upon decoding.

In order to acheive this, the inputs are encoded not as single points, but as probability distributions in the latent space. This allows for a meaningful output when decoding from any point.

{:refdef: style="text-align: center;"}
![VAE Resulting Latent Space](https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/img/2021-11-02/vae_2.png){: style="text-align:center"}
{: refdef}

While Variational Autoencoder architectures do not inherently allow information to persist, researchers at Google were able to combine a VAE with a Recurrent Neural Network to create a [MusicVAE](https://magenta.tensorflow.org/music-vae). This model is useful for creating short but more detailed melodies, between 2-16 bars.

### Datasets

- [VGMusic](https://vgmusic.com)
  - I ended up scraping all MIDIs from the [NES page](https://www.vgmusic.com/music/console/nintendo/nes/) as my main dataset.
- [NES Music Database (w/ Python library)](https://github.com/chrisdonahue/nesmdb)

## Implementation

Google Magenta already has pre-trained models that can be further trained on your own dataset to generate new music. To get my desired output, this seemed like the path of least resistance - I wouldn't want the project to turn into a [shed](https://cassandraxia.com/writing/shed.html).

After pip installing magenta alone did not allow me to use my GPU in Tensorflow, I created a new conda virtual environment, first conda installed tensorflow-gpu=2.3.0, then pip installed magenta, and then downgraded the numba library co-installed with magenta to version 0.48.0.

## Output

I tried out a few of the different models available in Magenta. Deeper neural nets with larger layer sizes resulted in better outputs, but I found myself memory-constrained training on a [gaming laptop](https://www.amazon.co.jp/%E3%82%B2%E3%83%BC%E3%83%9F%E3%83%B3%E3%82%B0%E3%83%8E%E3%83%BC%E3%83%88%E3%83%91%E3%82%BD%E3%82%B3%E3%83%B3-Zephyrus-%E3%83%A0%E3%83%BC%E3%83%B3%E3%83%A9%E3%82%A4%E3%83%88%E3%83%9B%E3%83%AF%E3%82%A4%E3%83%88-%E3%80%90%E6%97%A5%E6%9C%AC%E6%AD%A3%E8%A6%8F%E4%BB%A3%E7%90%86%E5%BA%97%E5%93%81%E3%80%91%E3%80%90%E3%81%82%E3%82%93%E3%81%97%E3%82%93%E4%BF%9D%E8%A8%BC%E3%80%91GA503QM-R9R3060WS%E3%80%90Windows-%E7%84%A1%E6%96%99%E3%82%A2%E3%83%83%E3%83%97%E3%82%B0%E3%83%AC%E3%83%BC%E3%83%89%E5%AF%BE%E5%BF%9C%E3%80%91/dp/B08XM9DMYY) with just 6GB of VRAM. I generated ~20 tracks for each model and selected a few that were interesting.

[Basic RNN](https://github.com/magenta/magenta/tree/main/magenta/models/melody_rnn): Produced the most basic tracks. Trained on default settings.

<midi-player
  src="https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/audio/basic_rnn.mid"
  sound-font visualizer="#myVisualizer">
</midi-player>
<midi-visualizer type="piano-roll" id="myVisualizer"></midi-visualizer>
</br>

[Lookback RNN](https://magenta.tensorflow.org/2016/07/15/lookback-rnn-attention-rnn/): This model was designed to help guide the training process towards understanding music structure in the following ways:

- Events from 1-2 bars prior are also input to the model so patterns can be more readily recognised.
- Whether the last event was repeating the event from 1-2 bars prior is also recorded and input, so the model can more easily recognise whether a track is in a repetitive or non-repetitive state.

This was the only model I trained on the NES Music Database dataset, and the output below came from a model trained at layer_size=[256,256], batch_size=16.

<midi-player
  src="https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/audio/lookback_rnn_256_nesmidi.mid"
  sound-font visualizer="#myVisualizer1">
</midi-player>
<midi-visualizer type="piano-roll" id="myVisualizer1"></midi-visualizer>
</br>

[Polyphony RNN](https://github.com/magenta/magenta/tree/main/magenta/models/polyphony_rnn): This model is capable of generating melodies with simulataneous notes. The output below came from a model trained at layer_size=[256,256], batch_size=16.

<midi-player
  src="https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/audio/polyphony_rnn_256.mid"
  sound-font visualizer="#myVisualizer2">
</midi-player>
<midi-visualizer type="piano-roll" id="myVisualizer2"></midi-visualizer>
</br>

[Music VAE](https://magenta.tensorflow.org/music-vae) (2-bar): There are versions of the Music VAE that produce more than 2 bars, but even a shallow network ([64,64]) with batch_size = 1 required more VRAM than my laptop could offer. Thus the result was quite underwhelming, but tracks generated showed more variation than other models.

<midi-player
  src="https://raw.githubusercontent.com/valencia21/valencia21.github.io/master/_site/assets/audio/2bar_music_vae_128.mid"
  sound-font visualizer="#myVisualizer3">
</midi-player>
<midi-visualizer type="piano-roll" id="myVisualizer3"></midi-visualizer>