# Cloudera Data Science Workbench

Basic tour of Cloudera Data Science Workbench (CDSW).

## 1. Platform Overview

### 1.1. Architecture

*TODO* 
- relationship with CDH, 
- admin account, 
- kubernates and docker, 
- engines (Mem, CPUs and GPUs).

### 1.2. Website - User Interface
Upon login into CDSW, you are presented with the Projects page. On the left menu, you have 6 tabs:

**Projects**
List of all your open Projects, plus on the top you have a quick glance at stats and available resources in the CDSW cluster. 

**Sessions**
List of all your open and closed Sessions. A Session is a compute environment loaded with Python/R/Scala libraries and with access to the filesystem.

**Experiments**
List of all Experiments run and the collected metrics. 

**Models**
List of all deployed Models.

**Jobs**
List of all Jobs run across all your projects.

**Settings**
The Settings page gives you access to:
- your Profile information.
- SSH Keys.
- Hadoop Authentication via Kerberos.
- API Keys.


## 2. Demo Excercises

### 2.1. Initial setup

Create a new project using this GitHub project as your initial setup. The project main page is presented to you. Here you have 8 tabs:

**Overview**: quick overview of the project, including displaying the README.md file.

**Sessions**: list of open session for this project.

**Experiments**: list of project experimets. 

**Models**: list of deployed project models

**Jobs**: list of project jobs

**Files**: list of files included in this project.

**Team**: this page allows you to add collaborators to this project.

**Settings**: this page allows you to select the Engine, set the visibility, and delete the project.

Open the Workbench and start a new Python2 Session, then run the following command to install some required libraries:
```
!pip2 install --upgrade pip dask keras matplotlib pandas_highcharts protobuf tensorflow seaborn sklearn
```

Open the Workbench and start a Python3 Session, then run the following command to install some required libraries:
```
!pip3 install --upgrade pip dask keras matplotlib pandas_highcharts protobuf tensorflow seaborn sklearn
```

Please note: as libraries are updated, dependancies might break: please check the output log and fix the dependancies. For example, you might have to install a specific version of setuptools or numpy for tensorflow to install properly.

Start a new R Session, and run these commands:
```
install.packages('sparklyr')
install.packages('plotly')
install.packages("nycflights13")
install.packages("Lahman")
install.packages("mgcv")
install.packages('shiny') 
```

Stop and restart all sessions for changes to take effect.

### 2.2. Workbench

In the project you'll find 4 scripts which walk you through the interactive capabilities of CDSW.

`1_python.py` demonstrates:
  - Markdown via comments
  - Jupyter-compatible visualizations
  - Simple console sharing

`2_pyspark.py` demonstrates:
  - Easy connectivity to (kerberized) Spark in YARN client mode.
  - Access to Hadoop HDFS CLI (e.g. `hdfs dfs -ls /`).

`3_tensorflow.py` demonstrates:
  - Ability to install and use custom packages (e.g. `pip search tensorflow`)

`4_sparklyr.R` demonstrates:
  - Use familiar dplyr with Spark using [Sparklyr](http://spark.rstudio.com)

`5_shiny.R` demonstrates:
  - Use of shiny to provide interactive graphics inside CDSW

*TODO: add scala example*

### 2.3. Experiments

## Experiment


Starting with version 1.4, Cloudera Data Science Workbench allows data scientists 
to run batch experiments that track different versions of code, input parameters, 
and output (both metrics and files).

### Challenge

As data scientists iteratively develop models, they often experiment with datasets, 
features, libraries, algorithms, and parameters. 
Even small changes can significantly impact the resulting model. 
This means data scientists need the ability to iterate and repeat similar experiments 
in parallel and on demand, as they rely on differences in output and scores to tune parameters 
until they obtain the best fit for the problem at hand. Such a training workflow requires 
versioning of the file system, input parameters, and output of each training run.

Without versioned experiments you would need intense process rigor to consistently 
track training artifacts (data, parameters, code, etc.), and even then it might be 
impossible to reproduce and explain a given result. This can lead to wasted 
time/effort during collaboration, not to mention the compliance risks introduced.

### Solution

Starting with version 1.4, Cloudera Data Science Workbench uses experiments 
to facilitate ad-hoc batch execution and model training. 
Experiments are batch executed workloads where the code, input parameters, 
and output artifacts are versioned. This feature also provides a lightweight 
ability to track output data, including files, metrics, and metadata for comparison.

### Concepts

The term experiment refers to a non interactive batch execution script that is 
versioned across input parameters, project files, and output. 
Batch experiments are associated with a specific project (much like sessions 
or jobs) and have no notion of scheduling; they run at creation time. 
To support versioning of the project files and retain run-level artifacts 
and metadata, each experiment is executed in an isolated container.

### Instructions

**STEP 1**

Start a workbench with a Python 3, open a terminal window:

```
chmod +x setup.sh
./setup.sh
```

**STEP 2** Examine `dsfortelco_sklearn_exp.py`

Open the file `dsfortelco_sklearn_exp.py`. This is a python program that builds a 
churn model to predict customer churn (the likelihood that this customer is going 
to stop his subscription with his telecom operator). There is a dataset available 
on hdfs with customer data, including a churn indicator field.

The program is going to build a churn prediction model using the Random Forest 
algorithm. Random forests are ensembles of decision trees. Random forests are one 
of the most successful machine learning models for classification and regression. 
They combine many decision trees in order to reduce the risk of overfitting. 
Like decision trees, random forests handle categorical features, extend to the 
multiclass classification setting, do not require feature scaling, and are able 
to capture non-linearities and feature interactions.

`spark.mllib` supports random forests for binary and multiclass classification and 
for regression, using both continuous and categorical features. spark.mllib 
implements random forests using the existing decision tree implementation. 
Please see the decision tree guide for more information on trees.

The Random Forest algorithm expects a couple of parameters:

- numTrees: Number of trees in the forest.

Increasing the number of trees will decrease the variance in predictions, 
improving the model’s test-time accuracy.
Training time increases roughly linearly in the number of trees.

- maxDepth: Maximum depth of each tree in the forest.

Increasing the depth makes the model more expressive and powerful. However, 
deep trees take longer to train and are also more prone to overfitting.
In general, it is acceptable to train deeper trees when using random forests 
than when using a single decision tree. One tree is more likely to overfit than 
a random forest (because of the variance reduction from averaging multiple trees 
in the forest).

In the `dsfortelco_sklearn_exp.py` program, these parameters can be passed to the 
program at runtime. In the lines 40 and 41, these parameters are passed to 
python variables:

```
param_numTrees = int(sys.argv[1])
param_maxDepth = int(sys.argv[2])
```

Also note that at the lines 71 and 72, the quality indicator for the Random Forest model, 
are written back to the Data Science Workbench repository:

```
cdsw.track_metric("auroc", auroc)
cdsw.track_metric("ap", ap)
```

These indicators will show up later in the **Experiments** dashboard.


**STEP 3** Run the experiment for the first time

Now, run the experiment using the following parameters:

numTrees = 40
numDepth = 20

From the menu, select Run -> Experiments. Now, in the background, the Data Science Workbench 
environment will spin up a new docker container, where this program will run.

Go back to the ‘Projects’ page in CDSW, and hit the ‘Experiments’ button.

If the Status indicates ‘Running’, you have to wait till the run is completed.
In case the status is ‘Build Failed’ or ‘Failed’, check the log information. 
This is accessible by clicking on the run number of your experiments. 
There you can find the session log, as well as the build information.

In case your status indicates ‘Success’, you should be able to see the auroc (Area Under the Curve) 
model quality indicator. It might be that this value is hidden by the CDSW user interface. 
In that case, click on the ‘3 metrics’ links, and select the auroc field. 
It might be needed to de-select some other fields, since the interface can only show 3 metrics 
at the same time.

In this example, 0.871. Not bad, but maybe there are better hyper parameter values available. 

**STEP 4** : Re run the experiment several times

Now, re-run the experiment 3 more times and try different values for NumTrees and NumDepth.
Try the following values:

```
NumTrees NumDepth
15       25
25       20
```


When all runs have completed successfully, check which parameters had the best quality 
(best predictive value). This is represented by the highest ‘area under the curve’, auroc metric.

**STEP 5** : Save the best model to your environment

Select the run number with the best predictive value. In the Overview screen of the experiment, 
you can see that the model in spark format, is captured in the file ‘sklearn_rf.pkl’. 
Select this file and hit the ‘Add to Project’ button. This will copy the model to your project 
directory.



### 2.3. Models


Starting with version 1.4, Cloudera Data Science Workbench allows data scientists to build, deploy, 
and manage models as REST APIs to serve predictions.


### Challenge
 
Data scientists often develop models using a variety of Python/R open source packages. 
The challenge lies in actually exposing those models to stakeholders who can test the model. 
In most organizations, the model deployment process will require assistance from a separate 
DevOps team who likely have their own policies about deploying new code.

For example, a model that has been developed in Python by data scientists might be rebuilt in 
another language by the devops team before it is actually deployed. This process can be slow and 
error-prone. It can take months to deploy new models, if at all. This also introduces compliance 
risks when you take into account the fact that the new re-developed model might not be even be an 
accurate reproduction of the original model.

Once a model has been deployed, you then need to ensure that the devops team has a way to rollback 
the model to a previous version if needed. This means the data science team also needs a reliable 
way to retain history of the models they build and ensure that they can rebuild a specific version 
if needed. At any time, data scientists (or any other stakeholders) must have a way to accurately 
identify which version of a model is/was deployed.


### Solution


Starting with version 1.4, Cloudera Data Science Workbench allows data scientists to build and 
deploy their own models as REST APIs. Data scientists can now select a Python or R function within
a project file, and Cloudera Data Science Workbench will:


Create a snapshot of model code, model parameters, and dependencies.
Package a trained model into an immutable artifact and provide basic serving code.
Add a REST endpoint that automatically accepts input parameters matching the function, and that 
returns a data structure that matches the function’s return type.
Save the model along with some metadata.
Deploy a specified number of model API replicas, automatically load balanced.


### Instructions

**STEP 1** : Examine the program `predic_churn_sklearn.py`

Open the project you created in the previous lab, and examine the file. This PySpark program uses 
the pickle.load mechanism to deploy models. The model it refers to the `sklearn_rf.pkl` file, was 
saved in the previous lab from the experiment with the best predictive model. 

There is a predict definition which is the function that calls the model, using features, 
and will return a result variable.


**STEP 2** : Deploy the model

From the projects page of your project, select the ‘Models’ button. Select ‘New Model’, and populate 
specify the following configuration:

```
Name:	“My Churn Prediction Model”
Description:	Anything you want
File:		predict_churn_sklearn.py
Function:	predict	
Example Input:
{
    "feature": "0, 65, 0, 137, 21.95, 83, 19.42, 111, 9.4, 6, 3.43, 4"
}
Kernal:		Python 3
Engine:	2 vCPU / 4 GB Memory
Replicas:	1
```

If all parameters are set, you can hit the ‘Deploy Model’ button. Wait till the model is deployed. 
This will take several minutes.


**STEP 3** : Test the deployed model

After the several minutes, your model should get to the ‘Deployed’ state. Now, click on the Model 
Name link, to go to the Model Overview page. From the that page, hit the ‘Test’ button to check 
if the model is working.

The green color with success is telling that our REST call to the model is technically working. 
And if you examine the response: {“result”: 1}, it returns a 1, which mean that customer with these 
features is likely to churn.

Now, lets change the input parameters and call the predict function again. Put the following values 
in the Input field:

```
{
  "feature": "0, 95, 0, 88, 26.62, 75, 21.05, 115, 8.65, 5, 3.32, 3"
}
```

With these input parameters, the model returns 0, which mean that the customer is not likely to churn.



### 2.5. Jobs

CDSW allows you to schedule a job - run some code - at a specific time or interval. For example, you can schedule `7_fit.py` to run every week on new data that you have collected, so that the model it builds becomes more precise with the updated data.
*TODO examples*
