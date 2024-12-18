# ADL-WS-2024
Repository for ADL project for WS24
Author: Viktoriia Ovsianik (12217985)

## Assignment 1 - Initiate

**Task:** Define the scope of the project & relevant resources

### 1. Topic Selected

Artistic Insights: Classifying and Interpreting Artwork Styles Using Deep Learning

### 2. Project Type

Bring your own method

### 3. Relevant articles

1. Jordan J. Bird, Ahmad Lotfi. CIFAKE: Image Classification and Explainable Identification of AI-Generated Synthetic Images, 2023. (https://arxiv.org/abs/2303.14126)
2. Lingquan Zeng. Improved Painting Image Style Classification of ResNet based on Attention Mechanism, IEEE, 2021. (https://ieeexplore.ieee.org/document/10512005)
3. Wentao Zhao, Dalin Zhou, Xinguo Qiu, and Wei Jiang. Compare the performance of the models in art classification, 2021 (https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0248414)
4. Ravidu Suien Rammuni Silva, Ahmad Lotfi, Isibor Kennedy Ihianle, Golnaz Shahtahmassebi, Jordan J. Bird. ArtBrain: An Explainable end-to-end Toolkit for Classification and Attribution of AI-Generated Art and Style (https://arxiv.org/abs/2412.01512)


### 4. Project Summary

 **4.1. Description**

This project aims to develop a deep learning model capable of classifying artwork into various styles and predicting whether it was created by a human artist or generated by AI. The model will be trained on a dataset comprising paintings from the 14th to the 21st century, as well as AI-generated artworks. These artworks will span ten distinct artistic styles: Art Nouveau, Baroque, Expressionism, Impressionism, Post-Impressionism, Realism, Renaissance, Romanticism, Surrealism, and Ukiyo-e. The  goal is to create a system that not only classifies artistic styles but also verifies the authenticity of paintings and evaluates their origin.

 **4.2. Dataset**

AI-ArtBench (https://www.kaggle.com/datasets/ravidussilva/real-ai-art)  
AI-ArtBench is a dataset that contains 180,000+ art images. 60,000 of them are human-drawn art that was directly taken from ArtBench-10 dataset and the rest is generated equally using Latent Diffusion and Standard Diffusion models. Dataset also contains information about art style.

 **4.3. Work breakdown**
| Task                                             | Expected Effort  | Real Effort   | Status        |
|--------------------------------------------------|------------------|---------------|---------------|
| Dataset collection                               | 2 hours           | 2 hours | Completed |
| Dataset preprocessing                            | 4 hours           | 12 hours| Completed |
| Designing & building network                     | 32 hours         | 24 hours| Completed |
| Fine-Tuning                                      | 32 hours          | xx| In Progress|
| Demo App                                         | 32 hours         | xx | Not Started |
| Report & Presentation                            | 10 hours           | xx | Not Started |

-----

## Assignment 2 - Initiate

**Task:** Build baseline model & optimize it

### 1. Repository structure

```bash
ADL-WS-2024/
│
├── README.md                     # Project overview, setup, and instructions
├── requirements.txt              # Python dependencies
├── .gitignore                    # Ignore unnecessary files (e.g., __pycache__)
│
├── datasets/                     # Folder for dataset preprocessors
│   ├── preprocessing.py          # Dataset preprocessing
│   ├── dataset.py                # Base class for Dataset creation
│   └── ALArtBench.py             # Class that organizes dataset in suitable format for pytorch.DataLoadr
│
├── models/                       # Models for the project
│   ├── resnet18.py               # ResNet18 fine-tuned implementation
│   └── simple_cnn.py             # Simple CNN baseline
│
├── notebooks/                    # Jupyter notebooks for exploration
│   ├── CNN.ipynb                 # Notebook for baseline model training
│   ├── Data.ipynb                # Notebook for data exploration and image printing
│   ├── ResNet.ipynb              # Notebook for basic ResNet18 model training
│   ├── Resnet_augmentation.ipynb # Notebook for ResNet18 model training with data augmentation
│   ├── Resnet_best_model.ipynb   # Notebook for ResNet18 model training with data augmentation and improved resolution
│   └── train_images_grid.png     # Image grid of sample artworks
│
├── scripts/                      # Standalone scripts
│   ├── evaluation.py             # Test model evaluation metrics
│   ├── metrics.py                # Train and validation tracking metrics
│   ├── run_cnn.py                # Script to run CNN training pipeline
│   ├── viz.py                    # Visualization helpers
│   └── wandb_logger.py           # Logging training metrics to Weights & Biases
│
├── trainers/                     # Training utilities
│   ├── __init__.py
│   └── ImgClassification.py      # Main image classification trainer

```
### 2. Installation

Clone the repository:

```bash
git clone https://github.com/ovsvc/ADL-WS-2024.git
cd ADL-WS-2024
```

Set up a Python environment (e.g., using Conda):

```bash
conda create --name adl-ws-2024 python=3.11.11 -y
conda activate adl-ws-2024
```

Install the required dependencies:

```bash
pip install -r requirements.txt
```
All models can be found and tested using `notebooks` section.

------------

### 3. Assignment progress discussion

**1. Dataset collection & preprocessing**

Dataset was collected from Kaggle. AI-ArtBench is a dataset that contains 180,000+ art images. 60,000 of them are human-drawn art that was directly taken from ArtBench-10 dataset and the rest is generated equally using Latent Diffusion and Standard Diffusion models. The human-drawn art is in 256x256 resolution and images generated using Latent Diffusion and Standard Diffusion has 256x256 and 768x768 resolutions respectively. Dataset already split into train and test parts and contains artworks from 10 different artistic styles. 

Processing: `datasets.preprocessing.py`
* Since the number of AI-generated artworks was higher in comparison to human ones, to balance the dataset I decided to select 5.000 images for each artistic style for each creation type (human, AI). Therefore, the final size of the training subset was (5.000 x 10 x2). Then I split the training subset into train and validation (90/10) setting the seed to provide reproducibility and resized all the images to 256x256.
* Labels were created with the logic {AI/human}_{artistic_style_name}

Processing: `datasets.AIArtbench.py`
* To seamlessly use pytorch.DataLoader later dataset was prepared in a special way using `datasets.AIArtbench.py` that inherits the logic of the base class `datasets.dataset.py`

**2. Task & models**

The main goal of the project was to predict the artistic style and creation type (human/AI) of the artwork (= Image Classification Problem).
To solve the task the following models were selected:

* 2.1. Simple_CNN (baseline)
I decided to start with a lightweight convolutional neural network designed for image classification. It features two convolutional layers with ReLU activation, followed by max pooling, and a fully connected layer with 20 output units. This simple structure served as a way to test if the whole pipeline works as expected as well as get a baseline measurement that I can improve throughout the project. 

* 2.2. Fine-tuned ResNet18 model
In most articles on this topic, authors often focus on training the ResNet50 model. However, its complexity can lead to overfitting and requires substantial computational resources. Therefore, I opted for the simpler ResNet18 model, as more complex architectures often yield only marginal improvements in accuracy. During training, I froze the layers `['conv1', 'bn1', 'layer1', 'layer2']` to concentrate fine-tuning on the deeper layers, enhancing feature extraction and classification.

* 2.3. Fine-tuned ResNet18 model with data augmentation
Since when fine-tuning Resnet18 I experienced overfitting in the model around 4-5 epoch, I decided to apply some transformations to the data so that the model could improve generalization.

**First variation**: 
Firstly, I applied the following transformations to the training set:
- RandomHorizontalFlip(): Randomly flips the image horizontally with a 50% chance.
- RandomRotation(30): Randomly rotates the image by a maximum of 30 degrees.
- RandomResizedCrop(32): Randomly crops a portion of the image and resizes it to 32x32 pixels.
- ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.2): Randomly adjusts the brightness, contrast, saturation, and hue of the image to simulate lighting changes.

However, the training results worsened further. I realized that predicting artistic styles might require preserving the original appearance of the image, so I removed **ColorJitter** as it alters the image significantly.

**Second  variation**: 
Then I changed towards the following transformations:
- RandomHorizontalFlip: Randomly flips the image horizontally with a 50% chance.
- RandomVerticalFlip: Randomly flips the image vertically with a 50% chance.
- RandomRotation(30): Randomly rotates the image by a maximum of 30 degrees.

Additionally, I decided to unfreeze all layers.

**Third  variation**: 
Finally, I chose to increase the image resolution from 32x32 to 224x224 because the model struggled to distinguish between similar art styles. Small details, which are crucial for differentiation, might have disappeared when working with low-resolution images.

**3. Training setup**

The training setup was similar for all models:
* epochs: 10
* patience: 3
* validation frequency: 1
* learning rate: 0.001
* optimizer: SGD Optimizer
* loss function: Cross entropy loss function
* training resources: Google Colab GPU

The logic for model training and testing can be found in `trainers.ImgClassification.py`.
Helper functions to initialize the training and testing process are in `scripts.run_cnn.py`.
The training and validation process has been stored and tracked using WandB Logger  `scripts.wandb_logger.py`.

**4. Metrics**

Since the main task was related to multi-class classification the following metrics were used:

* For tracking training process: accuracy, multi-class accuracy
* For evaluating results of the test set: accuracy, multi-class accuracy, precision, recall, f1-score, confusion matrix

The main metric was overall accuracy and the goal was to find an approach that would outperform the baseline.

**5. Models & Results**

**5.1 Simple CNN (baseline)**


| **Metric**              | **Value**  |
|-------------------------|------------|
| **Test Loss**            | 1.7142     |
| **Test Accuracy**        | 45.12%     |
| **Overall Precision**    | 44.83%     |
| **Overall Recall**       | 45.12%     |
| **Overall F1-Score**     | 44.03%     |

The Simple CNN baseline model achieved a test accuracy of 45.12% on the classification task across various artistic styles and authenticity labels. On average, the model is correct about 44.83% of the time when it predicts a class. Even though on average the model performance is quite poor for some cases it showed satisfactory results:
* Best performing classes: AI_renaissance (f1-score: 0.84), AI_baroque (f1-score: 0.69)
* Worst performing classes: AI_post_impressionism (f1-score: 0.11), AI_surrealism (f1-score: 0.14) & human_surrealism (f1-fcore: 0.14), human_art_nouveau (f1-score: 0.16)
* This model struggles not only to distinguish between artistic style but also to identify AI-generated and human-created artworks. 

**5.2. ResNet18 (fine tuning)**

| **Metric**              | **Value**  |
|-------------------------|------------|
| **Test Loss**            | 1.0641     |
| **Test Accuracy**        | 63.36%     |
| **Overall Precision**    | 63.60%     |
| **Overall Recall**       | 63.36%     |
| **Overall F1-Score**     | 63.21%     |

The ResNet18 model with transfer learning and frozen layers ['conv1', 'bn1', 'layer1', 'layer2'] achieved a Test Accuracy of 63.36% on the classification task across various artistic styles and authenticity labels. On average, the model is correct about 63.60% of the time when it predicts a class. Even though this model performs better than simple CNN it still has uneven performance across classes, thus:
* Best performing Classes: AI_renaissance (f-1 score: 0.95) & human_romanticism (f-1 score: 0.83)
* Worst performing Classes: AI_surrealism (f-1 score: 0.24), AI_post_impressionism (f-1 score: 0.22)  & human_expressionism (f-1 score: 0.36), human_impressionism (f-1 score: 0.27)
* The model faces challenges with classes that exhibit subtle or overlapping features, likely due to similarities in artistic elements. Notably, the model's primary difficulty lies in identifying the correct artistic style rather than distinguishing originality. This means AI-generated images are generally recognized as AI-generated but are often misclassified into a different style. This knowledge can potentially help with further performance improvement.
* Also it is important to mention that the training process for the model was stopped before reaching 10th epoch since the validation loss started to increase which is the sing of overfitting. 

**5.3. ResNet18 (fine tuning) with data augmentation**

| **Metric**               | **Value**  |
|--------------------------|------------|
| **Test Loss**            | 0.9344     |
| **Test Accuracy**        | 67.04%     |
| **Overall Precision**    | 66.55%     |
| **Overall Recall**       | 67.04%     |
| **Overall F1-Score**     | 66.56%     |

The ResNet18 model with data augmentation and non frozen parameter achieved a Test Accuracy of 67.04% on the classification task across various artistic styles and authenticity labels. On average, the model is correct about 66.55% of the time when it predicts a class. The model performs better than the previous version of ResNet18 because it avoids overfitting.
* Best performing Classes: AI_renaissance (f-1 score: 0.98), human_romanticism (f-1 score: 0.88), AI_baroque (f1-score: 0.86)
* Worst performing Classes: AI_post_impressionism (f-1 score: 0.27), human_expressionism (f-1 score: 0.39), human_impressionism (f-1 score: 0.21), human_baroque (f1-score: 0.31)
* The model struggles to classify correctly the same classes as model's previous version.

**5.4. ResNet18 (fine tuning) with data augmentation and increased image resolution (224x224)**

| **Metric**               | **Value**  |
|--------------------------|------------|
| **Test Loss**            | 0.4600     |
| **Test Accuracy**        | 82.90%     |
| **Overall Precision**    | 82.87%     |
| **Overall Recall**       | 82.90%     |
| **Overall F1-Score**     | 82.90%     |

Based on the evaluation results of the previous models it became clear that the model struggles to identify correctly such movements as impressionism, post-impressionism and expressionism as they usually have some overlapping visual traits and might not be easy to distinguish even for humans. Therefore, in attempt to improve the quality of results I decided to increase the resolution of the images to 224x224 as this would allow me to provide more details that might be crucial in distinguishing between the styles. This decidion led to significant improvement of the results:

* Best performing Classes: AI_renaissance (f-1 score: 1.0), human_romanticism (f-1 score: 0.98), AI_baroque (f1-score: 0.97)
* Worst performing Classes:
- human_impressionism (f-1 score: 0.42): usually mixed with human_baroque & human_surrealism
- AI_post_impressionism (f-1 score: 0.51): usually mixed with AI_surrealism & human_art_nouveau

**Incorrectly classified images**
![image](https://github.com/user-attachments/assets/6c0d113a-d6f1-4700-9010-783f6aaf4422)

**Correctly classified images**
![image](https://github.com/user-attachments/assets/da380adc-9498-4b19-a46f-3735328d5178)


**5.3. Train vs Validation Results**

**Train Progress Overview**

<img width="1079" alt="Screenshot 2024-12-18 at 13 15 29" src="https://github.com/user-attachments/assets/572ec28a-f1bb-445b-b404-0e646e446832" />

**Validation Progress Overview**

<img width="1080" alt="Screenshot 2024-12-18 at 13 16 21" src="https://github.com/user-attachments/assets/2e58efa5-4916-417e-8043-6ef17a647d06" />

Legend description:
* Simple_CNN (baseline) - The Simple CNN baseline model, description: section 2.1.
* ResNet18 (fine tuning) - Fine-tuning with frozen layers ['conv1', 'bn1', 'layer1', 'layer2'], description: section 2.2.
* ResNet18 (fine tuning + data augmentation) -  Fine-tuning with frozen layers ['conv1', 'bn1', 'layer1', 'layer2'] and data augmentation, description: section 2.3 (first variation)
* ResNet18 (fine tuning + data augmentation + no freeze) -  Fine-tuning with unfrozen layers and less aggressive data augmentation, description: section 2.3 (second variation)
* ResNet18 (fine tuning + data augmentation + no freeze + improved resolution) -  Fine-tuning with unfrozen layers, less aggressive data augmentation and improved resolution, description: section 2.3 (third variation)

For some model early stopping criteria was triggered, therefore, thaining was stopped before reaching 10th epoch. 

**6. Future Work**

* Get More Insight into Model Predictions (XRAI):

Implement XRAI or other explainability tools to better understand how the model makes predictions, particularly focusing on regions of images that are most influential for classification. This can help identify patterns and errors, particularly for complex class.
