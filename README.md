# DVC - Get Started

## Get Started
```bash
$ git init
$ python3 -m venv venv
$ source venv/bin/activate
$ pip install dvc
$ echo ".venv">>.gitignore
$ git add .gitignore
$ dvc init
$ git status
 	# new file:   .dvc/.gitignore
 	# new file:   .dvc/config
 	# new file:   .dvcignore
 	# new file:   .gitignore
$ git commit -m "Initialize DVC"
```
## Data Versioning
```bash
# Download the latest version of the data.xml as samples
$ dvc get https://github.com/iterative/dataset-registry \
           get-started/data.xml -o data/data.xml
# To start tracking a file or directory
$ dvc add data/data.xml
# DVC stores information about the added file in .dvc
# This metadata file is a placeholder for the original data
$ git add data/data.xml.dvc data/.gitignore
$ git status
 	# new file:   data/data.xml.dvc
 	# new file:   data/.gitignore
$ git commit -m "Add raw data"
```
## Storing and sharing
```bash
# dvc remote add -d storage gdrive://...
$ dvc remote add -d myremote /tmp/dvcstore
$ dvc push
$ git add .dvc/config
$ dvc remote list
	# myremote        /tmp/dvcstore
$ git commit -m "Configure local remote"
```
## Retrieving
```bash
$ rm -rf .dvc/cache
$ rm -f data/data.xml
$ dvc pull
```
## Making changes
```bash
$ cp data/data.xml /tmp/data.xml
$ cat /tmp/data.xml >> data/data.xml
$ dvc add data/data.xml
$ git commit data/data.xml.dvc -m "Dataset updates"
$ dvc push
```
## Switching between versions
```bash
$ git checkout HEAD~1 data/data.xml.dvc
$ dvc checkout
$ git commit data/data.xml.dvc -m "Revert dataset updates"
```

# How can we use these artifacts outside of the project?

Remember those .dvc files dvc add generates? Those files have their history in Git. DVC's remote storage config is also saved in Git, and contains all the information needed to access and download any version of datasets, files, and models. It means that a Git repository with DVC files becomes an entry point, and can be used instead of accessing files directly.

## List files and directories
```bash
$ dvc list https://github.com/iterative/dataset-registry get-started
```

## Download
```bash
$ dvc get https://github.com/iterative/dataset-registry use-cases/cats-dogs
```

Import to yout project
```bash
$ dvc import https://github.com/iterative/dataset-registry \
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


