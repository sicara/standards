# [MO] How to have repeatable experiments (10 min + time to run your experiment and upload inputs/outputs)

### Owner: [Alexandre Chaintreuil](https://github.com/achntrl)

## Why

On one project, we made several experiments, sometimes coding directly on the AWS GPU instance. When we needed to come back to a previous result we couldn't reproduce it. We lost several days getting back to a previous situation.

## Prerequisites

- I have an `experiments` folder in my project Google Drive folder
- I have a Mode of Operation to run an experiment (idealy one script commited in the repository)

## Steps

### Create a subfolder for your experiment (1 min)

In google drive, go to the `experiments` folder and create a sub folder

> ✅ Check : the experiment sub-folder is in the `experiment` folder


### Create a google doc file called `information` with the summary of the experiment : what are you trying to test and why (5 min)

Create a doc file. Write down the experiment you're going to run, and why you're going to run it.

> ✅ Check : your co developper or agile coach knows what experiment you're going to run and why


### Tag the code your going to run in git and note down the tag in the `information` file (2 min)

Commit the code you are going to run, possibly in a new branch to avoid polluting production code. Tag it with `git tag -a <your-tag-name> -m "<your tag message>"`. Then push your tag and write its name in the `information` file

> ✅ Check : you can see the tag on GitHub
>
> ✅ Check : you can see the tag in the `information` file

### Add all the inputs of the experiment in the folder (variable)

Your algorithm is going to run on some data. Store them in the drive. If all your experiments runs on the same data, it's OK to store it once.

> ✅ Check : Your algorithm inputs are stored in the drive.

### Run your experiment (variable)

You should have a Mode of operations to run your experiments. If you ran specific commands for this experiments, write them down in `information`.

> ✅ Check : Your experiment has ran

### Record your results (2 minutes)

Store the key metrics (score of your algorithm) in the `information` file

> ✅ Check : The `information` file contains the key metrics

### Store the output files (variable)

If you trained a neural network, you have weights you might want to reuse. Store them so you can reuse them without re-training the network.

If you use Keras, you can save your model using [this tutorial](https://keras.io/getting-started/faq/#how-can-i-save-a-keras-model). If the file is too big, you can same the model architecture to a json/yaml file, and save the weights in a separate file. We went from a 1.5 Go model to a ~500 Mo model using this trick.

> ✅ Check : I can reuse the model without training it

