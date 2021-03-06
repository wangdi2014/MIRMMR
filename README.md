# MIRMMR (murmur)
**M**icrosatellite **I**nstability **R**egression using **M**ethylation and **M**utations in **R**

MIRMMR uses logistic regression model building to predict microsatellite instability (MSI) status. Model building modules include **penalized**, **stepwise**, and **univarite** regression techniques. Once a model is built, the **predict** module allows new data to be scored quickly. The **compare** module lets users compare the performance of multiple tools. Remember,

> All models are wrong but some are useful. --George Box, statistician

---
# Usage
```
Rscript murmur.R -m module -f data.frame -i msi.status -c first.data.column -o output.prefix -d output.directory [options]
```

### Parameter specification
With single character flags (e.g. `-m`, `-f`), do not use an equals sign. With full word flags (e.g. `--plots`, `--xlabel`) use with an equals sign like `--plots=TRUE` or `--xlabel="My X label"`.

### Help 
To generate a help message for more details and options, use --help.
```
Rscript murmur.R --help
``` 

### R packages 

+ doMC (suggested)
+ ggplot2 (suggested)
+ glmnet (suggested)
+ grid (suggested)
+ MASS (suggested)
+ optparse (required)

### Main inputs
There are 6 major inputs required by most modules.

| Input | Explanation |
| --- | --- |
| `-m`,<br>`--module` | The module to be run, must be one of "compare", "penalized", "predict", "stepwise", or "univariate" |
| `-f`,<br>`--data_frame` | A file that R can read as a data.frame containing a header row, one row per sample, and a group of meta information columns (columns 1:(c-1)) followed by a group of data columns (columns c:end) |
| `-i`,<br>`--msi_status` | Name of the column with binary 'known truth' status calls. For the data in this column, things work better if TRUE corresponds to having whatever condition is being tested, but it also works if the data is stored as a binary vector that can be coerced to TRUE/FALSE. |
| `-c`,<br>`--first_data_column` | The number of the first data column that will be used as a regression predictor (assumes the remaining columns greater than it are also data columns that will be used as regression predictors) | 
| `-o`,<br>`--output_prefix` | File name prefix to use when writing output files |
| `-d`,<br>`--output_directory` | Directory name (relative or absolute path) to use when writing output files |

### Overwriting
The default behavior is to not overwrite existing files. Set `--overwrite=TRUE` to overwrite existing files.

### Plotting
There are several options relevant to plotting (only in compare and penalized modules)

| Option | Default | Explanation |
| --- | --- | --- |
| `--plots` | TRUE | Generate plots |
| `--xlabel` | NULL | Set x-label text |
| `--ylabel` | NULL | Set y-label text |
| `--title` | NULL | Set plot title |
| `--color_indicates` | NULL | Legend title, corresponds to `--group` option in penalized module and `--msi_status` column in compare module |
| `--theme_bw` | FALSE | Set the ggplot2 theme to bw and increase font size (for publications) |

---
# Modules

### Compare
Compare the results obtained through various methods with the compare module. Use `--plots=TRUE` to visualize results. Data columns could include MIRMMR scores, other quantitive method outcomes, and other binary (TRUE/FALSE or two factor vectors) method outcomes.

```
Rscript murmur.R -m compare -f data.frame -c first.data.column -o output.prefix -d output.directory [options]
```

### Penalized
The penalized module uses penalized regression to fit a logistic model. There are many command line options relevant to only this module.

```
Rscript murmur.R -m penalized -f data.frame -i msi.status -c first.data.column -o output.prefix -d output.directory [options]
```

| Option | Default | Explanation |
| --- | --- | --- |
| `--alpha` | 0.9 | Desired alpha level for penalized regression (0 is ridge, 1 is lasso) |
| `--consensus` | FALSE | Perform consenses variable finding for comparison to optimal lambda approach |
| `--group` | NULL | Identify the column name specifying the group a sample belongs to (e.g. cancer type) |
| `--lambda` | lambda.min | Procedure used by glmnet::cv.glmnet to report lambda (options: "lambda.min", "lambda.1se") |
| `--nfolds` | 10 | Number of folds to divide the data into for cross validation | 
| `--parallel` | FALSE | glmnet has built-in parallelization you can access if you have multiple cores | 
| `--par_cores` | 1 | Number of parallel cores to use; detect number of cores with parallel::detectCores() |
| `--repeats` | 1000 | Number of times to perform cross validation when selecting lambda or performing consensus variable finding |
| `--set_seed` | 0 (not set) | Seed value at the beginning to replicate previous results (cross validation is random) |
| `--train` | FALSE | Select a subset of data to train model and test the model with the remaining data | 
| `--train_proportion` | 0.8 | Proportion of samples to put in your training set with --train=TRUE |
| `--type_measure` | class | Type of cross validation error that is used to find the optimal lambda (options: "mse", "deviance", "mae", "class", and "auc") |

### Predict
The predict module predicts MSI status of new data (`-f`, `--data_frame`) based on a given prediction model. Identify the prediction model to use with `--model` (model should be saved as a unique object in an .Robj file).

```
Rscript murmur.R -m predict -f data.frame -c first.data.column -o output.prefix -d output.directory --model="model.Robj"
```

### Stepwise
The stepwise module uses MASS::stepAIC() to find an optimal model using both forward and backward steps. 

```
Rscript murmur.R -m stepwise -f data.frame -i msi.status -c first.data.column -o output.prefix -d output.directory
```

### Univariate
For each variable in the input, the univariate module fits a logistic regression model using that variable only (with intercept). 

```
Rscript murmur.R -m univariate -f data.frame -i msi.status -c first.data.column -o output.prefix -d output.directory
```
