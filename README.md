# DVC - Get Started

## Get Started
```bash
git init
python3 -m venv venv
source venv/bin/activate
pip install dvc
echo ".venv">>.gitignore
git add .gitignore
dvc init
git status
 	# new file:   .dvc/.gitignore
 	# new file:   .dvc/config
 	# new file:   .dvcignore
 	# new file:   .gitignore
git commit -m "Initialize DVC"
```
## Data Versioning
```bash
# Download the latest version of the data.xml as samples
dvc get https://github.com/iterative/dataset-registry \
           get-started/data.xml -o data/data.xml
# To start tracking a file or directory
dvc add data/data.xml
# DVC stores information about the added file in .dvc
# This metadata file is a placeholder for the original data
git add data/data.xml.dvc data/.gitignore
git status
 	# new file:   data/data.xml.dvc
 	# new file:   data/.gitignore
git commit -m "Add raw data"
```
## Storing and sharing
```bash
# dvc remote add -d storage gdrive://...
dvc remote add -d myremote /tmp/dvcstore
dvc push
git add .dvc/config
dvc remote list
	# myremote        /tmp/dvcstore
git commit -m "Configure local remote"
```
## Retrieving
```bash
rm -rf .dvc/cache
rm -f data/data.xml
dvc pull
```
## Making changes
```bash
cp data/data.xml /tmp/data.xml
cat /tmp/data.xml >> data/data.xml
dvc add data/data.xml
git commit data/data.xml.dvc -m "Dataset updates"
dvc push
```
## Switching between versions
```bash
git checkout HEAD~1 data/data.xml.dvc
dvc checkout
git commit data/data.xml.dvc -m "Revert dataset updates"
```

# How can we use these artifacts outside of the project?

Remember those .dvc files dvc add generates? Those files have their history in Git. DVC's remote storage config is also saved in Git, and contains all the information needed to access and download any version of datasets, files, and models. It means that a Git repository with DVC files becomes an entry point, and can be used instead of accessing files directly.

## List files and directories
```bash
dvc list https://github.com/iterative/dataset-registry get-started
```

## Download
```bash
dvc get https://github.com/iterative/dataset-registry use-cases/cats-dogs
```

Import to yout project
```bash
dvc import https://github.com/iterative/dataset-registry \
             use-cases/cats-dogs -o data/cats-dogs
```

## Python API

It's also possible to integrate your data or models directly in source code with DVC's Python API. This lets you access the data contents directly from within an application at runtime. For example:
```python
import dvc.api

with dvc.api.open(
    'get-started/data.xml',
    repo='https://github.com/iterative/dataset-registry'
) as fd:
    # fd is a file descriptor which can be processed normally
```

# Data Pipelines

```bash
wget https://code.dvc.org/get-started/code.zip
unzip code.zip
rm -f code.zip
pip install -r src/requirements.txt
```
```bash
dvc stage add -n prepare \
                -p prepare.seed,prepare.split \
                -d src/prepare.py -d data/data.xml \
                -o data/prepared \
                python src/prepare.py data/data.xml
```
A dvc.yaml file is generated. It includes information about the command we want to run (python src/prepare.py data/data.xml), its dependencies, and outputs.

The command options used above mean the following:

- **-n** prepare specifies a name for the stage. If you open the dvc.yaml file you will see a section named prepare.

- **-p** prepare.seed,prepare.split defines special types of dependencies — parameters. We'll get to them later in the Metrics, Parameters, and Plots page, but the idea is that the stage can depend on field values from a parameters file (params.yaml by default):

- **-d** src/prepare.py and **-d** data/data.xml mean that the stage depends on these files to work. Notice that the source code itself is marked as a dependency. If any of these files change later, DVC will know that this stage needs to be reproduced.

- **-o** data/prepared specifies an output directory for this script, which writes two files in it. This is how the workspace should look like after the run:

The last line, python src/prepare.py data/data.xml is the command to run in this stage, and it's saved to dvc.yaml, as shown below.

By using dvc stage add multiple times, and specifying outputs of a stage as dependencies of another one, we can describe a sequence of commands which gets to a desired result. This is what we call a data pipeline or dependency graph.

``` bash
dvc stage add -n featurize \
                -p featurize.max_features,featurize.ngrams \
                -d src/featurization.py -d data/prepared \
                -o data/features \
                python src/featurization.py data/prepared data/features
```

``` bash
dvc stage add -n train \
                -p train.seed,train.n_est,train.min_split \
                -d src/train.py -d data/features \
                -o model.pkl \
                python src/train.py data/features model.pkl
```

The whole point of creating this dvc.yaml file is the ability to easily reproduce a pipeline:
command can be used to compare this state with an actual state of the workspace.
``` bash
dvc status 
dvc params diff
dvc metrics diff
dvc plots diff
```

## Reproduce
The whole point of creating this dvc.yaml file is the ability to easily reproduce a pipeline:
``` bash
dvc repro
```

## Visualize
Having built our pipeline, we need a good way to understand its structure. Seeing a graph of connected stages would help. DVC lets you do so without leaving the terminal!
``` bash
dvc dag
```