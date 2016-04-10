A platform for training and testing convolutional neural network (CNN) based on [Caffe](http://caffe.berkeleyvision.org/)

## Prerequisite
[Caffe and pycaffe](http://caffe.berkeleyvision.org/installation.html) (already included for mri-wrapper)

## Quick run
We provide some toy data for you to quickly play around.

+ First replace all $REPO_HOME$ in the following files as the actual directory of the repository:

	+ example/param.list
	+ example/data/train.txt
	+ example/data/valid.txt
	+ example/data/test.txt


+ Train a simple neural network on the toy training data
	
	```
python run.py train example/param.list
```

+ Use the trained network to make prediction on the toy test set

	```
python run.py test example/param.list
```

+ Evaluate the testing result

	```
python run.py test_eval example/param.list
```

	The output will be under $REPO_HOME$/example/basicmodel/version0/best_trial/


## Data preparation


#### For DNA Sequence data
+ Prepare your sequence of training, validation and testing set separately in [TSV](https://github.com/gifford-lab/caffe-cnn/tree/master/example/sequence/sample.tsv) format.
+ Prepare the target (the label or real value you want to fit) for your training, validation and testing set separately. One line for each sample.
+ Use this [script](https://github.com/gifford-lab/caffe-cnn/tree/master/embedH5.py) to transform the sequence and target to the data format that Caffe can take. The "outfile" argument for training, validation and testing set should be the following respectively:
 
 	```
 $output_topdir$/data/train.h5
 $output_topdir$/data/valid.h5
 $output_topdir$/data/test.h5
 ```
 Type the following for more details on how to use the script:
 
	```
 python embedH5.py -h
 	```
 	
#### For other data type
You will need to manually prepare the following data under $output_topdir$/data/ ([Example](https://github.com/gifford-lab/caffe-cnn/tree/master/example/data))

+ Data matirx

	Training, validating and testing data should be each prepared as 4-D matrix with dimension _number * channel * width * height_ in HDF5 format. A good practice is to partition the dataset into small batches. 

+ Data manifest

	For each of train/valid/test set,  a [train.txt](https://github.com/gifford-lab/caffe-cnn/tree/master/example/data/train.txt)/[valid.txt](https://github.com/gifford-lab/caffe-cnn/tree/master/example/data/valid.txt)/[test.txt](https://github.com/gifford-lab/caffe-cnn/tree/master/example/data/test.txt) file is required to specify the ABSOLUTE PATH of all the HDF5 batches in the set.


## Caffe model file preparation
Refer to [here](http://caffe.berkeleyvision.org/) and [here](https://github.com/BVLC/caffe/tree/master/models) for instructions and examples in generating the following three files: 


+ [trainval.prototxt](https://github.com/gifford-lab/caffe-cnn/blob/master/example/trainval.prototxt): architecture for training
+ [solver.prototxt](https://github.com/gifford-lab/caffe-cnn/blob/master/example/solver.prototxt): solver parameters
+ [deploy.prototxt](https://github.com/gifford-lab/caffe-cnn/blob/master/example/deploy.prototxt): architecture for testing

Note in trainval.prototxt

+ Data must be input through a hdf5 layer
+ Data source (for training as an example) should be either "../../../data/train.txt" or an absolute path $output_topdir$/data/train.txt.

## Prepare [param.list](https://github.com/gifford-lab/caffe-cnn/blob/master/example/param.list)


```
output_topdir $REPO_HOME$/example/
gpunum 2
caffemodel_topdir example/
model_name basicmodel
batch_name version0
batch2predict version0
trial_num 3
optimwrt accuracy
outputlayer prob
```

+ `output_topdir`: The top output folder
+ `gpunum`: The GPU device number to run on
+ `caffemodel_topdir`: The directory where you put all the caffe model files
+ `model_name`: The name of the model. 
+ `batch_name`: The name of the specific model version. All the output will be therefore generated under $output_topdir$/$modelname$/$model_batchname$/
+ `batch2predict`: The model version to make prediction with.
+ `trial_num`: The number of training trial.
+ `optimwrt`: choose the best trial wrt this metric (accuracy or loss)
+ `outputlayer`: The name of the output blob that will be used as prediction in test phase


## Ready to go!


#### Train a neural network
$trial_num$ number of independent training will be performed using the same parameters. The output will be under $output_topdir$/$model_name$/$batch_name$/


```
python run.py train param.list
```

#### Make prediction with the trained neural network
First, from all the model files saved across different trials and iterations, the one with best validation accuracy will be picked. Then input the test data into this model to generate prediction output. All the ouput in test phase are under /$output_topdir$/$model_name$/$batch_name$/best_trial/

```
python run.py test param.list
```

#### Evaluate the prediction performance
Evaluate the performance on test set by accuracy and area under receiver operating curve (auROC). The output is in the same output folder as test phase.


```
python run.py test_eval param.list
```
