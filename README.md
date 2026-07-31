# MRI Brain Tissue Segmentation

Classical 2D and 3D computer-vision techniques for segmenting anatomical tissue regions from T1-weighted brain MRI data.

This project compares standard K-means clustering, Multi-Otsu thresholding, an extended K-means method using Sobel gradient features, and volume-based 3D K-means with spatial smoothing.

## Overview

The objective was to segment five anatomical tissue regions from ten consecutive axial MRI slices and compare the predicted segmentations against manually labelled ground truth.

The supplied MRI volume has the shape:

```text
362 × 434 × 10

Each voxel belongs to one of six labels:

Label	Region
0	Air / background
1	Skin / scalp
2	Skull
3	Cerebrospinal fluid
4	Grey matter
5	White matter

The project investigates whether incorporating image-gradient information or spatial information across neighbouring slices improves segmentation performance over intensity-only methods.

Methods
Preprocessing

The same preprocessing pipeline is applied before every segmentation method:

Convert the complete T1 volume to float32.
Apply global min-max normalisation to scale intensities to [0, 1].
Use K-means clustering with K=2 to separate low-intensity background from foreground.
Clean each foreground mask using:
Binary opening
Binary closing
Hole filling
Largest connected-component selection
Force all voxels outside the cleaned foreground mask to background label 0.

This ensures that every method receives the same normalised input and foreground region.

1. Standard 2D K-means

Standard K-means is applied independently to every MRI slice.

For each foreground pixel, the feature vector contains only its normalised intensity:

X = [normalised intensity]

Parameters:

K = 5
n_init = 5
max_iter = 150
random_state = 0

The same processing pipeline and parameters are used for all ten slices.

2. Multi-Otsu Thresholding

Multi-level Otsu thresholding is applied independently to the foreground pixels of each slice.

Five classes are requested, producing four thresholds that divide the intensity histogram into five regions:

threshold_multiotsu(foreground_intensities, classes=5)

The resulting intensity intervals are treated as tissue clusters.

3. Extended 2D K-means

The extended K-means method combines image intensity with Sobel gradient magnitude.

For every foreground pixel, the feature vector is:

X = [normalised intensity, α × gradient magnitude]

The Sobel gradient introduces edge information that can help distinguish tissues with similar intensity values but different boundary structures.

A grid search was performed using:

K ∈ {5, 6, 7, 8}
α ∈ {0.1, 0.2, 0.3, 0.5, 0.7}

Candidate configurations were ranked using:

score = 0.7 × foreground Dice + 0.3 × foreground accuracy

The selected configuration was:

K = 8
α = 0.5
4. Volume-Based 3D K-means

The 3D method processes all foreground voxels from the complete MRI volume together rather than clustering each slice independently.

The method consists of:

Stacking foreground intensities from all slices.
Applying K-means to the complete voxel set.
Testing candidate cluster counts from K=5 to K=8.
Selecting K=5 using an inertia-based heuristic.
Reshaping the cluster predictions back into the original 3D volume.
Applying a 3 × 3 × 3 majority filter to improve spatial consistency.

The majority filter replaces each voxel with the most common label in its local 3D neighbourhood, reducing small isolated regions and slice-to-slice inconsistencies.

Cluster-to-Label Matching

K-means and Multi-Otsu are unsupervised methods, so their cluster indices do not automatically correspond to anatomical labels.

For evaluation, each cluster is matched to the tissue label that occurs most frequently among the corresponding ground-truth pixels.

This is an evaluation-time label-matching procedure. It assigns semantic meaning to otherwise arbitrary cluster indices and is not a trained tissue-classification model.

Evaluation

The methods are evaluated using Dice similarity coefficient and pixel accuracy.

Dice Similarity Coefficient

For a predicted region P and ground-truth region G:

Dice = 2 × |P ∩ G| / (|P| + |G|)

Dice measures spatial overlap and is calculated separately for every label.

Two averages are reported:

Dice 0–5: mean Dice including background
Dice 1–5: mean Dice over the five anatomical tissue labels
Accuracy

Two accuracy measurements are also reported:

Accuracy 0–5: accuracy across the complete volume
Accuracy 1–5: accuracy only where the ground-truth label is greater than zero

Foreground metrics are particularly useful because the large background region can otherwise inflate overall accuracy.

Results
Method	Mean Dice 0–5	Mean Dice 1–5	Accuracy 0–5	Accuracy 1–5
Standard 2D K-means	0.680	0.623	78.93%	71.06%
Multi-Otsu 2D	0.691	0.636	79.83%	72.30%
3D volume K-means	0.694	0.640	80.14%	72.73%
Extended 2D K-means	0.755	0.713	82.18%	75.53%
Per-Class Dice Scores
Method	Air	Skin / Scalp	Skull	CSF	Grey Matter	White Matter
Standard 2D K-means	0.9649	0.2922	0.7518	0.3383	0.8130	0.9195
Multi-Otsu 2D	0.9649	0.2750	0.7509	0.3860	0.8271	0.9428
3D volume K-means	0.9626	0.2892	0.7389	0.3938	0.8358	0.9410
Extended 2D K-means	0.9649	0.4697	0.8253	0.5530	0.8042	0.9146
Findings

The extended 2D K-means method achieved the best overall result, with a mean foreground Dice score of 0.713 and foreground accuracy of 75.53%.

Adding Sobel gradient magnitude improved the separation of tissue boundaries that could not be distinguished reliably using intensity alone. The largest improvements were observed for skin/scalp, skull, and cerebrospinal fluid.

The 3D method slightly outperformed standard K-means and Multi-Otsu. Processing the complete volume and applying a 3D majority filter produced smoother and more spatially consistent predictions.

However, the 3D method remained intensity-only. It therefore could not match the performance of extended K-means, which used both intensity and edge information.

Repository Structure
MRI-Brain-Tissue-Segmentation/
├── README.md
├── MRI Brain Tissue Segmentation.ipynb
├── Report.pdf
└── Brain-1.mat

The dataset should only be included in the public repository when redistribution is permitted.

Installation

Clone the repository:

git clone https://github.com/Anagh09/MRI-Brain-Tissue-Segmentation.git
cd MRI-Brain-Tissue-Segmentation

Create a virtual environment:

python -m venv .venv

Activate it on Windows:

.venv\Scripts\activate

Activate it on macOS or Linux:

source .venv/bin/activate

Install the required packages directly:

pip install numpy scipy matplotlib scikit-learn scikit-image jupyter
Running the Notebook

Place Brain-1.mat in the same directory as the notebook.

Start Jupyter Notebook:

jupyter notebook

Open:

MRI Brain Tissue Segmentation.ipynb

Run the cells in order to:

Load and inspect the MRI data.
Generate the cleaned foreground mask.
Run standard 2D K-means.
Run Multi-Otsu segmentation.
tune and run extended K-means.
Run volume-based 3D K-means.
Calculate Dice and accuracy metrics.
Display the segmentation comparison and summary.
Requirements

The implementation uses Python 3 and the following libraries:

numpy
scipy
matplotlib
scikit-learn
scikit-image
jupyter
Detailed Report

The full experimental report contains the methodology, visual comparisons, evaluation and discussion:

View the MRI Brain Tissue Segmentation report

Limitations
The project evaluates only one MRI subject containing ten slices.
The unsupervised cluster labels are matched against the supplied ground truth during evaluation.
Hyperparameters are selected using the same supplied volume on which performance is reported.
Results therefore represent performance on this dataset rather than generalisation to unseen subjects.
Intensity-only methods struggle where tissue intensity distributions overlap.
The 3D majority filter can remove small isolated errors but may also smooth genuine fine structures.
No deep-learning or supervised segmentation method is used.
Possible Improvements

Future work could include:

Separating parameter tuning from final evaluation
Evaluating the methods on additional MRI subjects
Using spatial coordinates as clustering features
Incorporating gradient features into the 3D method
Comparing different 3D neighbourhood sizes
Applying tissue-specific morphological post-processing
Using automatic cluster-label assignment without access to test ground truth
Comparing the classical methods with supervised segmentation models

Author
Anagh Saha


This project was developed for the University of Birmingham Computer Vision and Imaging Extended module.

The repository is intended to demonstrate classical image-processing, clustering, segmentation and evaluation techniques.
