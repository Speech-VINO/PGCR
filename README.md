# Power BI App for Visualizing Dataset behaviour

In this project, a power bi based visualization app is proposed to analyze the spatial aspects of the dataset using Quality Spatial Relations (QSR). Two datasets have been used to be analysed under Monte Carlo Tree Search (MCTS) and also other popular Machine Learning algorithms. MCTS uses a UCB (Upper Confidence Bound) based reward system that simulates within the state space of every dataset. 

## Datasets used

### Speech 

From Speech project, [The RAVDESS dataset](https://www.kaggle.com/uwrfkaggler/ravdess-emotional-speech-audio) based on the paper [The Ryerson Audio-Visual Database of Emotional Speech and Song (RAVDESS)](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0196391) has been analysed using [speech_sigproc.py](./speech/audio/speech_sigproc.py). 

### OCR 

From OCR project, [The FUNSD Dataset](https://guillaumejaume.github.io/FUNSD/) based on the paper [FUNSD Paper](https://arxiv.org/abs/1905.13538) has been pre-processed using [preprocess.py](./ocr-data/text/preprocess.py).

# App Main Page

The App main page is a QSR visualizer page. When the maximum clusters are provided, the app asks for reward bias as well for input to MCTS simulation. 

## Computational Models used for the Power BI App

### How to execute the app

./speech/state_space/qsr.sh -d all

This command shows the final QSR plot after going through a `2` epoch training loop with Speech dataset, and another training loop with `all` distributions.

QSR Visualizer provides 2 plots:

- A Calculus plot (involving Region Connection Calculus (RCC), Ternary Point Configuration Calculus (TPCC))
- A Graphlet (involving temporal information integrated with the calculus)

All `.npy` files within [tmp_models](./speech/state_space/tmp_models) form the World Trace objects.

### How to train the vectorized model `factor_analysis.onnx`

python ./speech/state_space/factor_analysis.py --n_iter=20000 --learning_rate=0.001 --filepath=./speech/state_space/models/factor_analysis.onnx

#### Inference from the onnx model

```python

import onnxruntime as nxrun
sess = nxrun.InferenceSession("./speech/state_space/models/factor_analysis.onnx")

results = sess.run(None, {'latent': np.load('speech/dataset/latent.npy')})

```

## State Space Model

A state space model consisting of features and labels are fed into an MCTS algorithm. The state space model for each dataset consists of observations and transitions. When the data transitions to another state from an initial random state, the UCB model which is a multi-armed action bandit model, executes a Monte Carlo learning for `n` simulations. After obtaining the final state from an input state, the final state looks at the defined clusters which are ordered by a chosen metric. The number of maximum clusters are defined in the app's user interface. The maximum clusters should not exceed the state space dimension of the input data.

### Observations

Observations are inferences from the feature extraction stage. Let's say we converted Audio signals into `120` mels, and then execute a `FastICA` on the mels, we get a dimensionality reduced mixing matrix plotted as a point cloud plot as shown:

![ICA Plot for Item 0](./speech/simulator/ica_plot_0.png)

![ICA Plot for Item 1](./speech/simulator/ica_plot_1.png)

### Transitions

Transitions are estimated factors assumed to be hierarchical which are derived from the observations. The factors obtained are Gaussian in nature and they are vectorized into `speech/state_space/models/factor_analysis.onnx` model.

## MCTS Algorithm

The Monte Carlo Tree Search Algorithm updates the rewards based on the end result obtained instead of its online counterpart Temporal Difference Learning, TD (lambda). The algorithm clusters using ward linkage, with a maximum clusters parameter. 

Here are the results shown during training of the Reward Scores, No. of Units, Chosen probability density function:

**Below is the beta plot used for speech:**

![Speech Beta](./speech/simulator/speech_beta.png)

**Below is the weibull distribution used for OCR:**

![OCR Weibull](./ocr-data/simulator/ocr_weibull.png)

**Below is the no. of units used for calibration of weibull:**

![OCR Weibull Units](./ocr-data/simulator/ocr_weibull_units.png)

## Data Processing

For a dataset, the feature extraction is the first step. An Expectation Maximisation step has likelihoods in various forms. For factor analysis, there is a marginal likelihood; for dirichlet processes, the likelihood is a differential equation.

The episode scores obtained from running the MCTS algorithm, must be sorted in order to create a time series data for QSR analysis. The timestamps will be corresponding to `argsort()` of `episode_scores`. The rewards are hierarchical in nature, and hence the sorting of the scores are performed. 

### Feat file extraction

For evaluating the features from audio, the log mel spectrum and mel filterbank are produced. A hamming window is used to process the audio.

#### Mel Spectrum

Audio Signal
____________

![Audio Signal](./speech/plots/audio3.png)

Mel Filterbank
____________

![Mel Filterbank](./speech/plots/fbank3.png)

Mel Spectrum
____________

![Mel Spectrum](./speech/plots/mel_frequency3.png)

____________________________

## Data Wrangling

In the data wrangling stage, the data is processed through a complex plane where there is timestamp information recorded from the output of UCB scores from a Monte Carlo Simulation. Also, the time series is unit spaced with each recorded variable being: 

- no. of units required for calibration, 
- the chosen probability density function,
- scores that are scaled from 0 to 1

## Why is a probability density function chosen?

A probability density function is chosen because it can maximise the likelihood of a statistical model for a given use case. In the case of factor analysis, when the data is decomposed into its latent space, there are two Gaussian models which express the ground truth in terms of how the overall model can capture the variance. If a statistical model is compared against a probability density function, the QSR visualizer can estimate time, a histogram of intervals and a histogram of QSR relations.

## Multi-armed bandit frameworks

In a multi-armed bandit framework, a person simulates the environment by taking actions from a multi-labeled data source. The reward tree structure considers two approaches that have been documented based on:

- Least Noise model, in the case of Factor Analysis
- Maximum Similarity model, in the case of Latent Semantic Analysis

![Dataset Visualizer](./power-bi-visualizer-landing.png)

![Power BI Distribution Plots1](./power-bi-visualizer-viz1.png)

![Power BI Distribution Plots2](./power-bi-visualizer-viz2.png)

# Results of Execution (Speech Data)

## Analysis of Use Cases

These results are obtained after executing a TPCC (Ternary Point Configuration Calculus) algorithm on the World Trace. Each Graphlet is an intermediary layer that computes the Allen relation among the world traces. 

### Beta Distribution

The net time taken to produce the Qualitative Spatio-Temporal Activity Graph (QSTAG)

Time:  0.100687980652

number of eps: 39

**Histogram of Graphlets:**

_[20, 16, 13, 16, 16, 16, 2, 2, 2, 2, 1, 1, 1, 1, 2, 1, 1, 1]_

### Cauchy Distribution

The net time taken to produce the Qualitative Spatio-Temporal Activity Graph (QSTAG)

Time:  8.43655991554

number of eps: 146

**Histogram of Graphlets:**

_[70, 69, 73, 1, 70, 66, 69, 1, 2, 2, 2, 2, 2, 2, 1, 1, 1, 1]_

### Gamma Distribution

Time:  0.0908880233765

number of eps: 37

**Histogram of Graphlets:**

_[19, 16, 14, 16, 16, 2, 16, 2, 2, 2, 2, 1]_

### Rayleigh Distribution

Time:  1.20572209358

number of eps: 89

**Histogram of Graphlets:**

_[45, 41, 41, 38, 41, 41, 2, 2, 2, 1, 1, 2, 1, 2, 1, 1, 1, 1]_

### Weibull Distribution

Time:  0.0927407741547

number of eps: 37

**Histogram of Graphlets:**

_[19, 16, 14, 16, 16, 2, 16, 2, 2, 2, 2, 1]_
