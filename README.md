# DLCBL Hans classification 

Conceived to predict the Hans classification using multiplexed images from DLBCL patients. 

Adaptation of the code for the prediction of HER2 overexpression status and IHC score as seen in "Predicting the HER2 status in esophageal cancer from tissue microarrays using convolutional neural networks".

## Usage:  

### Quick intro
To train a resnet on example images available in _data/example_images_ run:  
```console
$ conda env create --file environment.yml
$ conda activate her2
$ python3 train.py --model=resnet34 --batch_size=4 --img_size=1024
$ python3 train_abmil.py --model=abmil --batch_size=1 --img_size=5468
```

### Recommended steps:

- Preprocessed your images: remove background, normalized the images if needed, you can see an example code in **data_preprocessing.py**
- Visualise your preprocessing and check if the dataset function are working: use the notebooks. 
- Generate two or three csv with the training, validation and test files 
- Adapt the training code to your experiment: change the task, change the cropping transformation, etc
- I recommend to use git and wand when you change something in the training code, this way you can see which commit was working and when it stopped working (easier to understand what you broke). Git also allows to go back in time to a working version if needed. 

It is usually easier to make the ResNet work first and then move to the abmil. At first, you can also train and validate with only few examples to make sure the code is running (see val_hans_test.csv). 

### Notebooks: 

- **abmil_check.ipynb**: test the **get_valid_patches** function from **utils.py** and adapt it to your images. Currently working for images with 0 as background.
- **data_exploration.ipynb**: visualize every image of your training or validation set to see if your preprocessing is working well.
- **generate_training_csv.ipynb**: code I used to generate the train and validation csv, to adapt to your data and problem. 
- **train_notebook.ipynb**: training notebook, original version by Juan -> is not adapted to multiplexed images with more than 3 channels. 

### Data format
Each dataset should be in a .csv file with the following columns:
- **filename**: paths of the images (directory + file name). 
- **task**: Numeric labels for your task (0, 1, 2)

### Scripts:
**train.py** or **train_abmil.py**: Train the CNNs. 

Arguments:

    --model: resnet34 or abmil
    --img_size or --patch_size: 1024 or 2048 for ResNet, 224 for abmil
    --channels: list of the channel you want to use, [list(range(14))] for all of them or [[0,1,6,8]] to select only some 
    --pretrained: True or False, if more than 3 channels, the first layer will not be pretrained 
    --frozen: True or False, if more than 3 channels, the first layer will not be frozen
    --task': name of your task, here relapse or hans_binary
    --epochs: number of epochs, default is 100
    --learning_rate: 0.001 for resnet and 0.0001 for abmil but to adapt to your experiments and to the batch size
    --weight_decay: 1e-8 to adapt to your experiments 
    --scheduler_factor:  0.1
    --scheduler_patience: 10
    --batch_size: currently 4 for ResNet (but could be increased depending on your number of channels) and 1 for ABMIL
    --train_csv: csv file containing file names and labels of training images 
    --val_csv: csv file containing file names and labels of validation images 
    --data_path: can be used if you want to change the file path for training but not redo training and validation csv
    --data_suffixe: can be used if you want to change the file name for training but not redo training and validation csv
    --checkpoints_dir: Path to save model checkpoints
    --num_workers: Number of workers for loading data, default = 0

**inference.py**: **_Was not modified to work with multiplexed images_**. 
Run a trained CNN on a test dataset, and export a .csv with the class probabilities for each image. 

Arguments:  
  --model MODEL         resnet34 or abmil (default: resnet34).  
  --task TASK           ihc-score or her2-status (default: her2-status).  
  --batch_size [BATCH_SIZE]
                        Batch size (default: 32).  
  --test_csv TEST_CSV   .csv file containing the test examples (default: test.csv).  
  --checkpoint_dir CHECKPOINT_DIR
                        Path to load the model checkpoint from (default: ./checkpoints/checkpoint_99.pth.tar).  
  --out_dir OUT_DIR     Path to save the inference results (default: ./out).  
  --num_workers NUM_WORKERS
                        Number of workers for loading data (default: 0).  
  --img_size IMG_SIZE   Input image size for the model (default: 1024).  

**staining_intensity_classifier.py**: **_Was not modified to work with multiplexed images_**.
Train and test the staining intensity classifier. 

Arguments:  
  --task TASK           ihc-score or her2-status (default: her2-status).  
  --train_csv TRAIN_CSV
                        .csv file containing the training examples (default: train.csv).  
  --test_csv TEST_CSV   .csv file containing the test examples (default: None).  
  --pickle_dir PICKLE_DIR
                        Folder to store the pickled classifier (default: ./).  
  --out_dir OUT_DIR     Path to save the inference results (default: ./out).  

### Training with Google Colab
 **train.py** makes use of **CUDA** to accelerate the training of the model. In case CUDA is not available to you, we provide **train_notebook.ipynb** that can be opened with [**Google Colab**](https://colab.research.google.com/)
