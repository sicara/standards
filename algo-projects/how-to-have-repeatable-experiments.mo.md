# [MO] How to have repeatable experiments (10 min + time to run your experiment and upload inputs/outputs)

### Owner: [Alexandre Chaintreuil](https://github.com/achntrl)

## Why

On one project, we made several experiments, sometimes coding directly on the AWS GPU instance. When we needed to come back to a previous result we couldn't reproduce it. We lost several days getting back to a previous situation.

## Prerequisites

- I have an `experiments` folder in my project Google Drive folder. You will have to store the training set and the validation set, and they can be big. It's better to save them in Google Drive (vs. in GitHub where they would bloat the repository).
- I have a script and/or a routine to run the experiment. Someone (who is not you) with your data set should be able to run the experiment.

## Steps

### Create a subfolder for your experiment (1 min)

In google drive, go to the `experiments` folder and create a sub folder.

> ✅ Check: the experiment sub-folder is in the `experiment` folder


### Create a google doc file called `information` with the summary of the experiment: what are you trying to test and why (5 min)

Create a doc file. Write down the experiment you're going to run, and why you're going to run it.

> ✅ Check: your co developper or agile coach knows what experiment you're going to run and why


### Make the code you are going to run available and note that down in the `information` file (2 min)

Commit and push the code you are going to run. You can add an `experiment` folder in the repo, or open a new branch to avoid polluting production code, or make a Jupyter notebook. Regardless how you choose to do it, the important thing is to make the code available in the repository. Write how to access the code in the `information` file.

> ✅ Check: you can see your experiment code on GitHub
>
> ✅ Check: you can see how to access the code in the `information` file

### Add all the inputs of the experiment in the folder (variable)

Your algorithm is going to run on some data. Store them in the drive. If all your experiments runs on the same data, it's OK to store it once.

> ✅ Check: Your algorithm inputs are stored in the drive.

### Run your experiment (variable)

You should have a Mode of operations to run your experiments. If you ran specific commands for this experiment, write them down in `information`.

> ✅ Check: Your experiment has ran

### Record your results (2 minutes)

Store the key metrics (score of your algorithm) in the `information` file.

> ✅ Check: The `information` file contains the key metrics

### Store the output files (variable)

If you trained a neural network, you have weights you might want to reuse. Store them so you can reuse them without re-training the network.

If you use Keras, you can save your model using [this tutorial](https://keras.io/getting-started/faq/#how-can-i-save-a-keras-model) or use [model checkpoints](https://keras.io/callbacks/#example-model-checkpoints). If the file is too big, you can same the model architecture to a json/yaml file, and save the weights in a separate file. We went from a 1.5 Go model to a ~500 Mo model using this trick.

If you use SKLearn, [it is also possible](http://scikit-learn.org/stable/modules/model_persistence.html).

> ✅ Check: I can reuse the model without training it

