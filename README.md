<p align="center"><a href="https://colab.research.google.com/github/Charmve/StegaStamp/blob/master/StegaStamp_train_model.ipynb"><img src="https://colab.research.google.com/assets/colab-badge.svg" align="center"></a>
</p>

# StegaStamp-plus
> A good exploit is one that is delivered in style.  -- Saumil Shah

Improved the original repo, <i>Invisible Hyperlinks in Physical Photographs</i> @tancik, which without datasets and training parameters.

<p align="center">
  <img src="profile.png" alt="Folders in Ubuntu">
</p>

## Features

- Improved training processing;
- Discovered training super-patrameters;
- Provided available training datasets;
- Shell scripts was organized instead of Python;

## Special Acknowledgement

The original author @[tancik](https://github.com/tancik).

## StegaStamp

> ## StegaStamp: Invisible Hyperlinks in Physical Photographs (CVPR 2020) [[Project Page]](http://www.matthewtancik.com/stegastamp)
> **[Matthew Tancik](https://www.matthewtancik.com), [Ben Mildenhall](http://people.eecs.berkeley.edu/~bmild/), [Ren Ng](https://scholar.google.com/citations?hl=en&user=6H0mhLUAAAAJ)**
*University of California, Berkeley*
> ![](https://github.com/tancik/StegaStamp/blob/master/docs/teaser.png)


## Introduction
This repository is a code release for the ArXiv report found [here](https://arxiv.org/abs/1904.05343). The project explores hiding data in images while maintaining perceptual similarity. Our contribution is the ability to extract the data after the encoded image (StegaStamp) has been printed and photographed with a camera (these steps introduce image corruptions). This repository contains the code and pretrained models to replicate the results shown in the paper. Additionally, the repository contains the code necessary to train the encoder and decoder models.

<h2 class="grey-heading">Method</h2>
<h3 class="sub_heading">Deployment</h3>
<div>
	<div class="div-block-5">
	<img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01.png" sizes="(max-width: 767px) 90vw, (max-width: 991px) 728px, 940px" srcset="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01-p-500.png 500w, https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01-p-800.png 800w, https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01-p-1080.png 1080w, https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01-p-1600.png 1600w, https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01-p-2000.png 2000w, https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca3b3c3ca205d53d7e986a1_pipeline-01.png 2407w" alt="" class="stega_pipeline_img"/>
	</div>
	<p class="paragraph-3 stega_text">Our system uses an encoder network to process the input image and hyperlink bitstring into a StegaStamp. The StegaStamp is then printed and captured by a camera. A detection network localizes and rectifies the StegaStamp before passing it to the decoder network. After the bits are recovered and error corrected, the user can follow the hyperlink.</p>
</div>

<h3 class="sub_heading">Training</h3>
<div>
<p class="paragraph-3 stega_text"> To train the encoder and decoder networks, we simulate the corruptions caused by printing, reimaging, and detecting the StegaStamp with a set of differentiable image augmentations.</p>
</div>

## Installation
- Clone repo and install submodules
```bash=
git clone --recurse-submodules https://github.com/tancik/StegaStamp.git
cd StegaStamp
```
- Install tensorflow (tested with tf 1.13)
- Python 3 required
- Download dependencies
```bash=
pip install -r requirements.txt
```

## Training
### Encoder / Decoder
- Set dataset path in train.py
```
TRAIN_PATH = DIR_OF_DATASET_IMAGES
```

- Train model
```bash=
bash scripts/base.sh EXP_NAME
```
The training is performed in `train.py`. There are a number of hyperparameters, many corresponding to the augmentation parameters. `scripts/bash.sh` provides a good starting place.

#### Pretrained network
Run the following in the base directory to download the trained network used in paper:
```bash=
wget http://people.eecs.berkeley.edu/~tancik/stegastamp/saved_models.tar.xz
tar -xJf saved_models.tar.xz
rm saved_models.tar.xz
```

### Detector
The training code for the detector model (used to segment StegaStamps) is not included in this repo. The model used in the paper was trained using the BiSeNet model released [here](https://github.com/GeorgeSeif/Semantic-Segmentation-Suite). CROP_WIDTH and CROP_HEIGHT were set to 1024, all other parameters were set to the default. The dataset was generated by randomly placing warped StegaStamps onto larger images.

The exported detector model can be downloaded with the following command:
```bash=
wget http://people.eecs.berkeley.edu/~tancik/stegastamp/detector_models.tar.xz
tar -xJf detector_models.tar.xz
rm detector_models.tar.xz
```

### Tensorboard
To visualize the training run the following command and navigate to http://localhost:6006 in your browser.
```bash=
tensorboard --logdir logs
```

## Encoding a Message
The script `encode_image.py` can be used to encode a message into an image or a directory of images. The default model expects a utf-8 encoded secret that is <= 7 characters (100 bit message -> 56 bits after ECC).

Encode a message into an image:
```bash=
python encode_image.py \
  saved_models/stegastamp_pretrained \
  --image test_im.png  \
  --save_dir out/ \
  --secret Hello
```
This will save both the StegaStamp and the residual that was applied to the original image.

Or run the bash code directly:
```bash=
bash encode_image.sh
```

## Decoding a Message
The script `decode_image.py` can be used to decode a message from a StegaStamp.

Example usage:
```bash=
python decode_image.py \
  saved_models/stegastamp_pretrained \
  --image out/test_hidden.png
```

Samely, or run the bash code directly:
```bash=
bash decode_image.sh
```

## Detecting and Decoding
The script `detector.py` can be used to detect and decode StegaStamps in an image. This is useful in cases where there are multiple StegaStamps are present or the StegaStamp does not fill the frame of the image.

To use the detector, make sure to download the detector model as described in the installation section. The recomended input video resolution is 1920x1080.

```bash=
python detector.py \
  --detector_model detector_models/stegastamp_detector \
  --decoder_model saved_models/stegastamp_pretrained \
  --video test_vid.mp4
```

<strong>Discription:</strong>
- Add the `--save_video FILENAME` flag to save out the results.

- The `--visualize_detector` flag can be used to visualize the output of the detector network. The mask corresponds to the segmentation mask, the colored polygons are fit to this segmentation mask using a set of heuristics. The detector outputs can noisy and are sensitive to size of the stegastamp. Further optimization of the detection network is not explored in this paper.

Samely, or run the bash code directly:
```bash=
bash detector.sh
```
in the terminal.

### Detecting and Decoding from Webcam
In browser 

```bash=
localhost:8080/detecdecode.html
```

## Example Encoded Images

<table>
  <tbody>
		<tr>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece3a76f9674cb331922_original_0.png" id="w-node-ac7c80dba393-64fa863e" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece369dd90202b3b9d7a_hidden_0.png" id="w-node-36366b9dbcb2-64fa863e" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece30eb6b33f8d4267ef_residual_0.png" id="w-node-e80884ffd5de-64fa863e" alt=""/></td>
		</tr>
		<tr>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece30eb6b39b324267f0_original_28.png" id="w-node-9c2eeb0f55f3-64fa863e" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece369dd9061a43b9d7c_hidden_28.png" id="w-node-3fd6c27963f0-64fa863e" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece39ef4082d8c846fd6_residual_28.png" id="w-node-f172e4fdbcf9-64fa863e" alt=""/></td>
		</tr>
		<tr>		
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece30eb6b3070b4267f1_original_25.png" id="w-node-1cb605591b2c-64fa863e" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece30eb6b30f6e4267f2_hidden_25.png" id="w-node-cc09a898c4fe-64fa863e" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4ece33f994729c3605a22_residual_25.png" id="w-node-0c2b7ac3257b-64fa863e" alt=""/></td>
		</tr>
		<tr>
			<td><div id="w-node-c76dcfc62cf8-64fa863e" class="stega_example_label">Original Image</div></td>
			<td><div id="w-node-62d402cd6de4-64fa863e" class="stega_example_label">StegaStamp</div></td>
			<td><div id="w-node-3acde7b9475e-64fa863e" class="stega_example_label">Residual</div></td>
		</tr>
	</tbody>
</table>
<p class="paragraph-3 stega_text">Here are examples of images that have been converted to StegaStamps. The residual depicts the difference between the original image and the StegaStamp.</p>
	
## Results

<table>
  <tbody>
		<tr>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4f5095181343d9eac9d81_oblique.gif" alt="" class="image-7"/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4fcb6b1119cca3212a112_lighting.gif" alt=""/></td>
		</tr>
		<tr>
			<td><div class="captions">Oblique Angles</div></td>
			<td><div class="captions">Variable Lighting</div></td>
		</tr>
		<tr>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4f8dd79305788c84a1319_occlusion.gif" alt=""/></td>
			<td><img src="https://uploads-ssl.webflow.com/51e0d73d83d06baa7a00000f/5ca4f6dc518134364eacd8d8_reflection.gif" alt=""/></td>
		</tr>
		<tr>
			<td><div class="captions">Occlusion</div></td>
			<td><div class="captions">Reflections</div></td>
		</tr>
	</tbody>
</table>
<p class="paragraph-3 stega_text stega_results_text">Here are examples of detection and decoding. The percentage corresponds to the number of bits correctly decoded. Each of these examples encode 100 bits. </p>

## Getting Started Yourself

The project is still a work in progress, but I want to put it out so that I get some good suggestions.

The easiest way to get started is to simply try out on Colab: 

[<img src="https://colab.research.google.com/assets/colab-badge.svg" align="center">](https://colab.research.google.com/github/Charmve/StegaStamp/blob/master/StegaStamp_train_model.ipynb)  

The secret.len is limited 7 characters (56 bit).

## Disclaimer

Thanks to the excilent open source work from Matthew Tancik, Ben Mildenhall, et.al !

> Any interest disputes and social consequences arising from using this method have nothing to do with the open source author of this project.
