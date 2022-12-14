Niche Modeling
================
Marcos V. Dantas-Queiroz
7/13/2022

# Ecological Niche Modeling

In this notebook I include all steps needed to run an Ecological Niche
Modeling (ENM), for the comparative phylogeographic paper of the BQM
species. Note that some steps require a large amount of processing, so
they will not perform adequately in a regular desktop/notebook. I
expressly indicate when are those cases with a begin sentence. These
computationally intensive analyses were ran in GridUnesp, a
High-perfomance computing cluster.

Also, some steps were conducted in other program languages (e.g., bash,
Python). I also pointed out when this is the case.

## 1. Getting the data

One of the reviewers suggested that we should use a more finer time
scale in our ENM in order to show if the flickering connectivity could
be taking place in the BQMs. The reviewer suggested the
[CHELSA-TraCE21k](https://chelsa-climate.org/chelsa-trace21k/) database,
a long term transient climate data at 1km resolution in 100-yeartime
steps from today to the LGM (\~ 21,000).

However, for our purposes (and to reduce the huge amount of data), a
100-yeartime steps scale is too much. A **1000-yeartime** steps would be
a more adequate scenario to see any signal of demographic changes in our
species.

The [download
page](https://envicloud.wsl.ch/#/?prefix=chelsa%2Fchelsa_V1%2Fchelsa_trace%2F)
of the CHELSA-TraCE21k provides a list of files for each Bioclimatic
variable [(BIO1-BIO19)](https://www.worldclim.org/data/bioclim.html),
starting in 0 (current time) and going back to -200 (20,000 years ago).
The file names are sorted something like this:

`CHELSA_TraCE21k_bio01_0_V1.0.tif` \| **the current time slice for the
bio01 variable**  
`CHELSA_TraCE21k_bio01_-1_V1.0.tif` \| the 100-year slice for the bio01
variable  
`CHELSA_TraCE21k_bio01_-2_V1.0.tif` \| the 200-year slice for the bio01
variable  
`CHELSA_TraCE21k_bio01_-3_V1.0.tif` \| the 300-year slice for the bio01
variable  
`...`  
`CHELSA_TraCE21k_bio01_-10_V1.0.tif` \| **the 1,000-year slice for the
bio01 variable**  
`CHELSA_TraCE21k_bio01_-11_V1.0.tif` \| the 1,100-year slice for the
bio01 variable  
`CHELSA_TraCE21k_bio01_-12_V1.0.tif` \| the 1,200-year slice for the
bio01 variable  
`CHELSA_TraCE21k_bio01_-20_V1.0.tif` \| **the 2,000-year slice for the
bio01 variable**  
`CHELSA_TraCE21k_bio01_-30_V1.0.tif` \| **the 3,000-year slice for the
bio01 variable**  
`...`  
`CHELSA_TraCE21k_bio01_-100_V1.0.tif` \| **the 10,000-year slice for the
bio01 variable**  
`CHELSA_TraCE21k_bio01_-101_V1.0.tif` \| the 10,100-year slice for the
bio01 variable  
`...`  
`CHELSA_TraCE21k_bio01_-110_V1.0.tif` \| **the 11,000-year slice for the
bio01 variable**  
`CHELSA_TraCE21k_bio01_-120_V1.0.tif` \| **the 12,000-year slice for the
bio01 variable**  
`...`  
`CHELSA_TraCE21k_bio01_-190_V1.0.tif` \| **the 19,000-year slice for the
bio01 variable**  
`CHELSA_TraCE21k_bio01_-200_V1.0.tif` \| **the 20,000-year slice for the
bi01 variable**

Since we are interested in a 1000-yeartime steps, we need to filter only
the files that have this time slices (in bold, in the example above.
Thus, we want all files that has the `_0_` (current time), the whole
tens (i.e., `_-10_` \| 1,000 years; `_-20_` \| 2,000 years, etc), and
the hundreds with whole tens (i.e., `-_100_` \| 10,000 years; `_-110_`
\| 11,000 years; `_-200_` \| 20,000 years, etc).

So, for each bioclimatic variable (19), we???ll have 21 files, ending up
with 399 files.

1.  current
2.  1,000 years
3.  2,000 years
4.  3,000 years
5.  4,000 years
6.  5,000 years
7.  6,000 years
8.  7,000 years
9.  8,000 years
10. 9,000 years
11. 10,000 years
12. 11,000 years
13. 12,000 years
14. 13,000 years
15. 14,000 years
16. 15,000 years
17. 16,000 years
18. 17,000 years
19. 18,000 years
20. 19,000 years
21. 20,000 years

Instead of checking each of the 399 boxes on the download page, we
submit a file with the link of each of one of the files, using the
`wget` command in `bash`. To create the file, follow the code below.

### 1.1 Creating a file to download all bioclim files using `wget`

The following script creates a simple txt file, where each line will be
a link with the desired bioclim file.

``` r
bioclims <- c("bio01", "bio02", "bio03", "bio04", "bio05", "bio06", "bio07", "bio08", "bio09", "bio10", "bio11", "bio12", "bio13", "bio14", "bio15", "bio16", "bio17", "bio18", "bio19")

years <- c("_0_", "_-10_", "_-20_", "_-30_", "_-40_", "_-50_", "_-60_", "_-70_", "_-80_", "_-90_", "_-100_", "_-110_", "_-120_", "_-130_", "_-140_", "_-150_", "_-160_", "_-170_", "_-180_", "_-190_", "_-200_")

file.create("my_wget.txt")

#for each bioclimatic file
for(b in bioclims){
  #for each year
  for(y in years){
    #create a line withe the entire link + the bioclimatic file + the year + the end of the file
    line <- paste0("https://os.zhdk.cloud.switch.ch/envicloud/chelsa/chelsa_V1/chelsa_trace/bio/CHELSA_TraCE21k_", b, y, "V1.0.tif")
    #write that line in a file
    write(line, file = 'my_wget.txt', append = TRUE)
  }
}
```

With the `my_wget.txt` file, you can submit it via `wget` operator in
`bash`. I did this step in GridUnesp, since the files are huge. However,
we will crop the files to our desire region (South America), yielding
smaller files.

<p style="color:red">
**this is a `bash` command**
</p>

``` bash
wget -i my_wget.txt
```

### 1.2 Cropping the bioclim files to desired extent

Now that all bioclimatic files are available in my computer/cluster,
we???ll crop the files to our desired extent, i.e., instead of using the
entire world as a variable, we???ll only use a region. In our case is the
South American continent. This script takes sometime to run, so I
recommend to submit it as a job instead of running it in the frontend
(if you are using some cluster service, of course).

**`script_crop_and_mask_brazil_grid.R`**

``` r
#import some libraries
library(raster)
library(rnaturalearth)

#Read all files ended with *.tif (the bioclim files)
#Make sure that you are in the correct directory and there are no other tif files beside the bioclims
raw_chelsa_files <- list.files(pattern = '*.tif')

#The South America extent
sa <- ne_countries(continent = 'south america', returnclass = 'sf')

#for each bioclim file
for(f in raw_chelsa_files){
  #import it as a raster file
  a <- raster(f)
  #mask it by the south america continent and then crop it, creating a new file
  #with a 'sa_' in the beginning of the file name
  mask(crop(a, sa), sa, filename = paste0("sa_", f))
}
```

A example of a submiting job file (using the GridUnesp parameters) is
given below. Since we are using libraries that are not natively
installed in the GridUnesp, I create a new environment called ENM, where
I installed the required R libraries (`raster` and `rnaturalearth`).
More information about how it works, check the [GridUnesp
page](https://www.ncc.unesp.br/gridunesp/docs/v2/manual_do_usuario.html)
(in Portuguese).

<p style="color:red">
**this is a `bash` file**
</p>

**`submit_script_crop_and_mask_sa_grid.sh`**

``` bash
#!/bin/bash
#SBATCH -t 48:00:00 -c 4

export INPUT="script_crop_and_mask_sa_grid.R CHELSA_TraCE21k_bio*V1.0.tif"
export OUTPUT="sa_CHELSA_TraCE21k_bio*V1.0.tif"

module load miniconda/3
source activate ENM

job-nanny srun Rscript script_crop_and_mask_sa_grid.R --input=CHELSA_TraCE21k_bio*V1.0.tif
```

After that I recommend to move the `sa_` files to a new folder and
**delete the original CHELSA data** (or moving them to a backup HD)
after the cropping as they take up a lot of space and can easily be
downloaded again later.

## 2. Variable selection

Depending of the studied region, some bioclimatic variables might be
high correlated between each other. Using such correlated data is
redundant, leading to poor results and unnecessarily increasing analyses
processing time. Thus, it???s important to evaluate if there are any
correlation and keeping only those variables with low correlation
values. The script below calculate the correlation of the current-time
variables, using distinct methods.

**`script_correlations_grid.R`**

``` r
# packages
library(raster)
library(rgdal)
library(corrplot)
library(vegan)
library(psych)
library(ncdf4)

#grab all file names starting with "sa_"
clim_sa_tif <- list.files(pattern = 'sa_CHELSA*')

#Current
cur <- grep('_0_', clim_sa_tif, value = T)
y_cur <- stack(cur)

#Extract values of cells
y_values_cur <- values(y_cur)

#omit NAs
y_values_cur <- na.omit(y_values_cur)

##Correlation
corr_cur <- cor(y_values_cur)

write.table(abs(round(corr_cur, 2)), 'correlation_cur.xls', row.names = T, sep = '\t')
write.table(ifelse(corr_cur >= 0.7, 'sim', 'nao'), 'correlation_cur_afirmation.xls', row.names = T, sep = '\t')

#plot of correlation
png("cor_cur.png", w = 18, he = 18, units = "cm", res = 300)
corrplot(corr_cur, type = "lower", diag = F, tl.srt = 45, mar = c(3, 0.5, 2, 1),
         title = "Correlacao entre variaveis Bioclimaticas (presente)")

dev.off()

##PCA

pca_cur <- prcomp(y_values_cur, scale = T)

#screenplot
png("pca_present_screeplot.tif", wi = 18, he = 18, un = "cm", res = 300)
par(mar = c(3, 5, 5, 2))
screeplot(pca_cur, main = "Contribuicao de cada PC (presente)", ylab = "Autovalores", cex.lab = 1.3)
abline(h = 1, col = "red", lty = 2)
dev.off()

#export table with each contribution
write.table(abs(round(pca_cur$rotation[, 1:5], 2)), "contr_cur_pca.xls", row.names = T, sep = "\t")

##Factorial analysis

png("screeplot_fatorial_cur.tif", wi = 18, he = 18, un = "cm", res = 300)
fa.parallel(y_values_cur, fm = "ml", fa = "fa") 
dev.off()

fa_cur <- fa(y_values_cur, nfactors = 5, rotate = "varimax", fm = "ml")
fa_cur_loadings <- loadings(fa_cur)

write.table(abs(round(fa_cur_loadings, 2)), "fa_cur_loadings.xls", row.names = T, sep = "\t")
```

Our factorial analysis (the method that I choose to select the
correlated variables) shows that the most uncorrelated variables are:
`bio07`, `bio10`, `bio13`, `bio14` and `bio19`. So, we keep working only
with them.

## 3. Modeling

Now it???s time to model the ecological niche for our species for all time
slices that we selected before. For this, we???ll need two inputs: the
bioclimatic variables and the coordinate points of our species. Check
below how the coordinate file looks like.

**`Vriesea_oligantha_coords.txt`**

| decimalLatitude | decimalLongitude |
|-----------------|------------------|
| -20.50505558    | -43.63914144     |
| -19.9475        | -43.751111       |
| -19.5733        | -43.2956         |
| -19.3333        | -43.5167         |
| -19.26854205    | -43.54895258     |
| -19.24772191    | -43.50991404     |
| -18.962778      | -43.767222       |
| -18.46232308    | -43.34785782     |
| -18.461389      | -43.464167       |
| -18.358         | -43.6836         |

The modeling itself is the step the request most processing power, so I
suggest that you split each algorithm and each replicate in a different
job, as I did below.

#### **Bioclim**

**`Bioclim_script_Bokermannohyla_alvarengai_1.R`**

``` r
library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#Number of replicate (check below how I did this)
i <- 1

#check the pathname
points <- read.table('Bokermannohyla_alvarengai.txt', header = T, sep = '\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

for(j in seq(10, 200, by = 10)){
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Bioclim <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Bioclim <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ BioClim ############
print(paste('Bioclim',i))
#if(rep>1){bc_ant <- bc_model}
bc_model <- bioclim(cur_bios,pres_train)
#if(rep>1){bc_mean <- mean(bc_ant,bc_model)}

#dismo::evaluate_models
e_bc <- dismo::evaluate(pres_test,backgr_test,bc_model,cur_bios)
tr_bc <- dismo::threshold(e_bc,'spec_sens')

id_bc <- which(e_bc@t == tr_bc)
eval_bc_rep <- c(e_bc@t[id_bc], e_bc@auc, (e_bc@TPR[id_bc]+e_bc@TNR[id_bc]-1),i)
eval_Bioclim <- rbind(eval_Bioclim, eval_bc_rep)

write.csv(eval_Bioclim, file = 'eval_Bioclim_Bokermannohyla_alvarengai_1.csv',row.names = F)

#projections
pred_cur_bc <- predict(cur_bios,bc_model,progress='')
writeRaster(pred_cur_bc, filename = 'cur_bios_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_bc)

pred_y_10_bc <- predict(y_10, bc_model, progress = '')
writeRaster(pred_y_10_bc, filename='y_010_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_bc)

pred_y_20_bc <- predict(y_20, bc_model, progress = '')
writeRaster(pred_y_20_bc, filename='y_020_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_bc)

pred_y_30_bc <- predict(y_30, bc_model, progress = '')
writeRaster(pred_y_30_bc, filename='y_030_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_bc)

pred_y_40_bc <- predict(y_40, bc_model, progress = '')
writeRaster(pred_y_40_bc, filename='y_040_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_bc)

pred_y_50_bc <- predict(y_50, bc_model, progress = '')
writeRaster(pred_y_50_bc, filename='y_050_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_bc)

pred_y_60_bc <- predict(y_60, bc_model, progress = '')
writeRaster(pred_y_60_bc, filename='y_060_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_bc)

pred_y_70_bc <- predict(y_70, bc_model, progress = '')
writeRaster(pred_y_70_bc, filename='y_070_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_bc)

pred_y_80_bc <- predict(y_80, bc_model, progress = '')
writeRaster(pred_y_80_bc, filename='y_080_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_bc)

pred_y_90_bc <- predict(y_90, bc_model, progress = '')
writeRaster(pred_y_90_bc, filename='y_090_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_bc)

pred_y_100_bc <- predict(y_100, bc_model, progress = '')
writeRaster(pred_y_100_bc, filename='y_100_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_bc)

pred_y_110_bc <- predict(y_110, bc_model, progress = '')
writeRaster(pred_y_110_bc, filename='y_110_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_bc)

pred_y_120_bc <- predict(y_120, bc_model, progress = '')
writeRaster(pred_y_120_bc, filename='y_120_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_bc)

pred_y_130_bc <- predict(y_130, bc_model, progress = '')
writeRaster(pred_y_130_bc, filename='y_130_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_bc)

pred_y_140_bc <- predict(y_140, bc_model, progress = '')
writeRaster(pred_y_140_bc, filename='y_140_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_bc)

pred_y_150_bc <- predict(y_150, bc_model, progress = '')
writeRaster(pred_y_150_bc, filename='y_150_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_bc)

pred_y_160_bc <- predict(y_160, bc_model, progress = '')
writeRaster(pred_y_160_bc, filename='y_160_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_bc)

pred_y_170_bc <- predict(y_170, bc_model, progress = '')
writeRaster(pred_y_170_bc, filename='y_170_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_bc)

pred_y_180_bc <- predict(y_180, bc_model, progress = '')
writeRaster(pred_y_180_bc, filename='y_180_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_bc)

pred_y_190_bc <- predict(y_190, bc_model, progress = '')
writeRaster(pred_y_190_bc, filename='y_190_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_bc)

pred_y_200_bc <- predict(y_200, bc_model, progress = '')
writeRaster(pred_y_200_bc, filename='y_200_Bioclim_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_bc)
```

#### **Gower**

**`Gower_script_Bokermannohyla_alvarengai_1.R`**

``` r
library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('Bokermannohyla_alvarengai.txt', header = T, sep = '\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}

i <- 1

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Gower <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Gower <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ Gower ############ 
#if(rep>1){gower_ant <- gower_model}
print(paste('Gower',i))
gower_model <- domain(cur_bios,pres_train)
#if(rep>1){gower_mean <- mean(gower_ant,gower_model)}

#dismo::evaluate_models
e_gower <- dismo::evaluate(pres_test,backgr_test,gower_model,cur_bios)
tr_gower <- dismo::threshold(e_gower,'spec_sens')

id_gower <- which(e_gower@t == tr_gower)
eval_gower_rep <- c(e_gower@t[id_gower], e_gower@auc, (e_gower@TPR[id_gower]+e_gower@TNR[id_gower]-1),i)
eval_Gower <- rbind(eval_Gower, eval_gower_rep)
write.csv(eval_Gower, file = 'eval_Gower_Bokermannohyla_alvarengai_1.csv', row.names = F)

#projections
pred_cur_gower <- predict(cur_bios,gower_model,progress='')
writeRaster(pred_cur_gower, filename = 'cur_bios_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_gower)

pred_y_10_gower <- predict(y_10, gower_model, progress = '')
writeRaster(pred_y_10_gower, filename='y_010_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_gower)

pred_y_20_gower <- predict(y_20, gower_model, progress = '')
writeRaster(pred_y_20_gower, filename='y_020_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_gower)

pred_y_30_gower <- predict(y_30, gower_model, progress = '')
writeRaster(pred_y_30_gower, filename='y_030_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_gower)

pred_y_40_gower <- predict(y_40, gower_model, progress = '')
writeRaster(pred_y_40_gower, filename='y_040_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_gower)

pred_y_50_gower <- predict(y_50, gower_model, progress = '')
writeRaster(pred_y_50_gower, filename='y_050_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_gower)

pred_y_60_gower <- predict(y_60, gower_model, progress = '')
writeRaster(pred_y_60_gower, filename='y_060_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_gower)

pred_y_70_gower <- predict(y_70, gower_model, progress = '')
writeRaster(pred_y_70_gower, filename='y_070_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_gower)

pred_y_80_gower <- predict(y_80, gower_model, progress = '')
writeRaster(pred_y_80_gower, filename='y_080_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_gower)

pred_y_90_gower <- predict(y_90, gower_model, progress = '')
writeRaster(pred_y_90_gower, filename='y_090_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_gower)

pred_y_100_gower <- predict(y_100, gower_model, progress = '')
writeRaster(pred_y_100_gower, filename='y_100_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_gower)

pred_y_110_gower <- predict(y_110, gower_model, progress = '')
writeRaster(pred_y_110_gower, filename='y_110_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_gower)

pred_y_120_gower <- predict(y_120, gower_model, progress = '')
writeRaster(pred_y_120_gower, filename='y_120_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_gower)

pred_y_130_gower <- predict(y_130, gower_model, progress = '')
writeRaster(pred_y_130_gower, filename='y_130_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_gower)

pred_y_140_gower <- predict(y_140, gower_model, progress = '')
writeRaster(pred_y_140_gower, filename='y_140_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_gower)

pred_y_150_gower <- predict(y_150, gower_model, progress = '')
writeRaster(pred_y_150_gower, filename='y_150_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_gower)

pred_y_160_gower <- predict(y_160, gower_model, progress = '')
writeRaster(pred_y_160_gower, filename='y_160_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_gower)

pred_y_170_gower <- predict(y_170, gower_model, progress = '')
writeRaster(pred_y_170_gower, filename='y_170_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_gower)

pred_y_180_gower <- predict(y_180, gower_model, progress = '')
writeRaster(pred_y_180_gower, filename='y_180_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_gower)

pred_y_190_gower <- predict(y_190, gower_model, progress = '')
writeRaster(pred_y_190_gower, filename='y_190_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_gower)

pred_y_200_gower <- predict(y_200, gower_model, progress = '')
writeRaster(pred_y_200_gower, filename='y_200_Gower_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_gower)
```

#### **Mahalanobis**

**`Mahalanobis_script_Bokermannohyla_alvarengai_1.R`**

``` r
library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('Bokermannohyla_alvarengai.txt', header = T, sep = '\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}

i <- 1

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Mahalanobis <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Mahalanobis <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ mahalanobis ############

#if(rep>1){mahalanobis_ant <- mahalanobis_model}
print(paste('Mahalanobis',i))
mahalanobis_model <- mahal(cur_bios,pres_train)
#if(rep>1){mahalanobis_mean <- mean(mahalanobis_ant,mahalanobis_model)}

#dismo::evaluate_models
e_mahalanobis <- dismo::evaluate(pres_test,backgr_test,mahalanobis_model,cur_bios)
tr_mahalanobis <- dismo::threshold(e_mahalanobis,'spec_sens')

id_mahalanobis <- which(e_mahalanobis@t == tr_mahalanobis)
eval_mahalanobis_rep <- c(e_mahalanobis@t[id_mahalanobis], e_mahalanobis@auc, (e_mahalanobis@TPR[id_mahalanobis]+e_mahalanobis@TNR[id_mahalanobis]-1),i)
eval_Mahalanobis <- rbind(eval_Mahalanobis, eval_mahalanobis_rep)
write.csv(eval_Mahalanobis, file = 'eval_Mahalanobis_Bokermannohyla_alvarengai_1.csv', row.names = F)

#projections
pred_cur_mahalanobis <- predict(cur_bios,mahalanobis_model,progress='')
writeRaster(pred_cur_mahalanobis, filename = 'cur_bios_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_mahalanobis)

pred_y_10_mahalanobis <- predict(y_10, mahalanobis_model, progress = '')
writeRaster(pred_y_10_mahalanobis, filename='y_010_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_mahalanobis)

pred_y_20_mahalanobis <- predict(y_20, mahalanobis_model, progress = '')
writeRaster(pred_y_20_mahalanobis, filename='y_020_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_mahalanobis)

pred_y_30_mahalanobis <- predict(y_30, mahalanobis_model, progress = '')
writeRaster(pred_y_30_mahalanobis, filename='y_030_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_mahalanobis)

pred_y_40_mahalanobis <- predict(y_40, mahalanobis_model, progress = '')
writeRaster(pred_y_40_mahalanobis, filename='y_040_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_mahalanobis)

pred_y_50_mahalanobis <- predict(y_50, mahalanobis_model, progress = '')
writeRaster(pred_y_50_mahalanobis, filename='y_050_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_mahalanobis)

pred_y_60_mahalanobis <- predict(y_60, mahalanobis_model, progress = '')
writeRaster(pred_y_60_mahalanobis, filename='y_060_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_mahalanobis)

pred_y_70_mahalanobis <- predict(y_70, mahalanobis_model, progress = '')
writeRaster(pred_y_70_mahalanobis, filename='y_070_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_mahalanobis)

pred_y_80_mahalanobis <- predict(y_80, mahalanobis_model, progress = '')
writeRaster(pred_y_80_mahalanobis, filename='y_080_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_mahalanobis)

pred_y_90_mahalanobis <- predict(y_90, mahalanobis_model, progress = '')
writeRaster(pred_y_90_mahalanobis, filename='y_090_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_mahalanobis)

pred_y_100_mahalanobis <- predict(y_100, mahalanobis_model, progress = '')
writeRaster(pred_y_100_mahalanobis, filename='y_100_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_mahalanobis)

pred_y_110_mahalanobis <- predict(y_110, mahalanobis_model, progress = '')
writeRaster(pred_y_110_mahalanobis, filename='y_110_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_mahalanobis)

pred_y_120_mahalanobis <- predict(y_120, mahalanobis_model, progress = '')
writeRaster(pred_y_120_mahalanobis, filename='y_120_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_mahalanobis)

pred_y_130_mahalanobis <- predict(y_130, mahalanobis_model, progress = '')
writeRaster(pred_y_130_mahalanobis, filename='y_130_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_mahalanobis)

pred_y_140_mahalanobis <- predict(y_140, mahalanobis_model, progress = '')
writeRaster(pred_y_140_mahalanobis, filename='y_140_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_mahalanobis)

pred_y_150_mahalanobis <- predict(y_150, mahalanobis_model, progress = '')
writeRaster(pred_y_150_mahalanobis, filename='y_150_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_mahalanobis)

pred_y_160_mahalanobis <- predict(y_160, mahalanobis_model, progress = '')
writeRaster(pred_y_160_mahalanobis, filename='y_160_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_mahalanobis)

pred_y_170_mahalanobis <- predict(y_170, mahalanobis_model, progress = '')
writeRaster(pred_y_170_mahalanobis, filename='y_170_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_mahalanobis)

pred_y_180_mahalanobis <- predict(y_180, mahalanobis_model, progress = '')
writeRaster(pred_y_180_mahalanobis, filename='y_180_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_mahalanobis)

pred_y_190_mahalanobis <- predict(y_190, mahalanobis_model, progress = '')
writeRaster(pred_y_190_mahalanobis, filename='y_190_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_mahalanobis)

pred_y_200_mahalanobis <- predict(y_200, mahalanobis_model, progress = '')
writeRaster(pred_y_200_mahalanobis, filename='y_200_Mahalanobis_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_mahalanobis)
```

#### **Random Forest**

**`RandomForest_script_Bokermannohyla_alvarengai_1.R`**

``` r
library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('Bokermannohyla_alvarengai.txt', header = T, sep = '\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}

i <- 1

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Randomforest <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Randomforest <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

## Random Forest
#if(rep>1){randforest_ant <- randforest_model}
print(paste('Random Forest',i))
randforest_model <- randomForest(pa ~ sa_CHELSA_TraCE21k_bio07_0_V1.0 + sa_CHELSA_TraCE21k_bio10_0_V1.0 + sa_CHELSA_TraCE21k_bio13_0_V1.0 + sa_CHELSA_TraCE21k_bio14_0_V1.0 +sa_CHELSA_TraCE21k_bio19_0_V1.0,data=envtrain)
#if(rep>1){randforest_mean <- mean(randforest_ant,randforest_model)}

#dismo::evaluate_models
e_randomforest <- dismo::evaluate(pres_test,backgr_test,randforest_model,cur_bios)
tr_randomforest <- dismo::threshold(e_randomforest,'spec_sens')

id_randomforest <- which(e_randomforest@t == tr_randomforest)
eval_randomforest_rep <- c(e_randomforest@t[id_randomforest], e_randomforest@auc, (e_randomforest@TPR[id_randomforest]+e_randomforest@TNR[id_randomforest]-1),i)
eval_Randomforest <- rbind(eval_Randomforest, eval_randomforest_rep)
write.csv(eval_Randomforest, file = 'eval_RandomForest_Bokermannohyla_alvarengai_1.csv',row.names = F)


#projections
pred_cur_randforest <- predict(cur_bios,randforest_model,progress='')
writeRaster(pred_cur_randforest, filename = 'cur_bios_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_randforest)

pred_y_10_randforest <- predict(y_10, randforest_model, progress = '')
writeRaster(pred_y_10_randforest, filename='y_010_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_randforest)

pred_y_20_randforest <- predict(y_20, randforest_model, progress = '')
writeRaster(pred_y_20_randforest, filename='y_020_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_randforest)

pred_y_30_randforest <- predict(y_30, randforest_model, progress = '')
writeRaster(pred_y_30_randforest, filename='y_030_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_randforest)

pred_y_40_randforest <- predict(y_40, randforest_model, progress = '')
writeRaster(pred_y_40_randforest, filename='y_040_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_randforest)

pred_y_50_randforest <- predict(y_50, randforest_model, progress = '')
writeRaster(pred_y_50_randforest, filename='y_050_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_randforest)

pred_y_60_randforest <- predict(y_60, randforest_model, progress = '')
writeRaster(pred_y_60_randforest, filename='y_060_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_randforest)

pred_y_70_randforest <- predict(y_70, randforest_model, progress = '')
writeRaster(pred_y_70_randforest, filename='y_070_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_randforest)

pred_y_80_randforest <- predict(y_80, randforest_model, progress = '')
writeRaster(pred_y_80_randforest, filename='y_080_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_randforest)

pred_y_90_randforest <- predict(y_90, randforest_model, progress = '')
writeRaster(pred_y_90_randforest, filename='y_090_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_randforest)

pred_y_100_randforest <- predict(y_100, randforest_model, progress = '')
writeRaster(pred_y_100_randforest, filename='y_100_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_randforest)

pred_y_110_randforest <- predict(y_110, randforest_model, progress = '')
writeRaster(pred_y_110_randforest, filename='y_110_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_randforest)

pred_y_120_randforest <- predict(y_120, randforest_model, progress = '')
writeRaster(pred_y_120_randforest, filename='y_120_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_randforest)

pred_y_130_randforest <- predict(y_130, randforest_model, progress = '')
writeRaster(pred_y_130_randforest, filename='y_130_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_randforest)

pred_y_140_randforest <- predict(y_140, randforest_model, progress = '')
writeRaster(pred_y_140_randforest, filename='y_140_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_randforest)

pred_y_150_randforest <- predict(y_150, randforest_model, progress = '')
writeRaster(pred_y_150_randforest, filename='y_150_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_randforest)

pred_y_160_randforest <- predict(y_160, randforest_model, progress = '')
writeRaster(pred_y_160_randforest, filename='y_160_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_randforest)

pred_y_170_randforest <- predict(y_170, randforest_model, progress = '')
writeRaster(pred_y_170_randforest, filename='y_170_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_randforest)

pred_y_180_randforest <- predict(y_180, randforest_model, progress = '')
writeRaster(pred_y_180_randforest, filename='y_180_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_randforest)

pred_y_190_randforest <- predict(y_190, randforest_model, progress = '')
writeRaster(pred_y_190_randforest, filename='y_190_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_randforest)

pred_y_200_randforest <- predict(y_200, randforest_model, progress = '')
writeRaster(pred_y_200_randforest, filename='y_200_RandomForest_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_randforest)
```

#### **SVM**

**`SVM_script_Bokermannohyla_alvarengai_1.R`**

``` r
library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('Bokermannohyla_alvarengai.txt', header = T, sep = '\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}

i <- 1

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_SVM <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_SVM <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ SVM ############
#if(rep>1){svm_ant <- svm_model}
print(paste('SVM',i))
svm_model <- ksvm(pa ~ sa_CHELSA_TraCE21k_bio07_0_V1.0 + sa_CHELSA_TraCE21k_bio10_0_V1.0 + sa_CHELSA_TraCE21k_bio13_0_V1.0 + sa_CHELSA_TraCE21k_bio14_0_V1.0 +sa_CHELSA_TraCE21k_bio19_0_V1.0,data=envtrain)
#if(rep>1){svm_mean <- mean(svm_ant,svm_model)}

#dismo::evaluate_models
e_svm <- dismo::evaluate(pres_test,backgr_test,svm_model,cur_bios)
tr_svm <- dismo::threshold(e_svm,'spec_sens')

id_svm <- which(e_svm@t == tr_svm)
eval_svm_rep <- c(e_svm@t[id_svm], e_svm@auc, (e_svm@TPR[id_svm]+e_svm@TNR[id_svm]-1),i)
eval_SVM <- rbind(eval_SVM, eval_svm_rep)
write.csv(eval_SVM, file = 'eval_SVM_Bokermannohyla_alvarengai_1.csv',row.names = F)


#projections
pred_cur_svm <- predict(cur_bios,svm_model,progress='')
writeRaster(pred_cur_svm, filename = 'cur_bios_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_svm)

pred_y_10_svm <- predict(y_10, svm_model, progress = '')
writeRaster(pred_y_10_svm, filename='y_010_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_svm)

pred_y_20_svm <- predict(y_20, svm_model, progress = '')
writeRaster(pred_y_20_svm, filename='y_020_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_svm)

pred_y_30_svm <- predict(y_30, svm_model, progress = '')
writeRaster(pred_y_30_svm, filename='y_030_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_svm)

pred_y_40_svm <- predict(y_40, svm_model, progress = '')
writeRaster(pred_y_40_svm, filename='y_040_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_svm)

pred_y_50_svm <- predict(y_50, svm_model, progress = '')
writeRaster(pred_y_50_svm, filename='y_050_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_svm)

pred_y_60_svm <- predict(y_60, svm_model, progress = '')
writeRaster(pred_y_60_svm, filename='y_060_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_svm)

pred_y_70_svm <- predict(y_70, svm_model, progress = '')
writeRaster(pred_y_70_svm, filename='y_070_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_svm)

pred_y_80_svm <- predict(y_80, svm_model, progress = '')
writeRaster(pred_y_80_svm, filename='y_080_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_svm)

pred_y_90_svm <- predict(y_90, svm_model, progress = '')
writeRaster(pred_y_90_svm, filename='y_090_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_svm)

pred_y_100_svm <- predict(y_100, svm_model, progress = '')
writeRaster(pred_y_100_svm, filename='y_100_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_svm)

pred_y_110_svm <- predict(y_110, svm_model, progress = '')
writeRaster(pred_y_110_svm, filename='y_110_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_svm)

pred_y_120_svm <- predict(y_120, svm_model, progress = '')
writeRaster(pred_y_120_svm, filename='y_120_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_svm)

pred_y_130_svm <- predict(y_130, svm_model, progress = '')
writeRaster(pred_y_130_svm, filename='y_130_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_svm)

pred_y_140_svm <- predict(y_140, svm_model, progress = '')
writeRaster(pred_y_140_svm, filename='y_140_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_svm)

pred_y_150_svm <- predict(y_150, svm_model, progress = '')
writeRaster(pred_y_150_svm, filename='y_150_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_svm)

pred_y_160_svm <- predict(y_160, svm_model, progress = '')
writeRaster(pred_y_160_svm, filename='y_160_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_svm)

pred_y_170_svm <- predict(y_170, svm_model, progress = '')
writeRaster(pred_y_170_svm, filename='y_170_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_svm)

pred_y_180_svm <- predict(y_180, svm_model, progress = '')
writeRaster(pred_y_180_svm, filename='y_180_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_svm)

pred_y_190_svm <- predict(y_190, svm_model, progress = '')
writeRaster(pred_y_190_svm, filename='y_190_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_svm)

pred_y_200_svm <- predict(y_200, svm_model, progress = '')
writeRaster(pred_y_200_svm, filename='y_200_SVM_Bokermannohyla_alvarengai_1', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_svm)
```

To submit several jobs, I create a Python script to create multiple R
scripts and multiple sh files. Check how it works. Maybe there is a
better a way to do this, but it worked for me.

``` r
import os
import io

sp_list = ["Bokermannohyla_alvarengai","Bokermannohyla_oxente","Bokermannohyla_saxicola","Euphorbia_attastoma","Lychnophora_ericoides","Neoregelia_bahiana","Pilosocereus_aurisetus","Pithecopus_megacephalus","Pleurodema_alium","Richterago_discoidea","Tibouchina_papyrus","Vellozia_auriculata","Vriesea_oligantha"]

#Bioclim scripts

for sp in sp_list:
    for i in range(1, 6):
        file_bioclim = io.open(f"Bioclim_script_{sp}_{i}.R", "w", newline = "\n")
        file_bioclim.write ("""library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('/home/marcosvdqueiroz/ENM/chelsa_files/ENM/coords/{0}.txt', header = T, sep = '\\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){{
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}}

i <- {1}

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Bioclim <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Bioclim <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ BioClim ############
print(paste('Bioclim',i))
#if(rep>1){{bc_ant <- bc_model}}
bc_model <- bioclim(cur_bios,pres_train)
#if(rep>1){{bc_mean <- mean(bc_ant,bc_model)}}

#dismo::evaluate_models
e_bc <- dismo::evaluate(pres_test,backgr_test,bc_model,cur_bios)
tr_bc <- dismo::threshold(e_bc,'spec_sens')

id_bc <- which(e_bc@t == tr_bc)
eval_bc_rep <- c(e_bc@t[id_bc], e_bc@auc, (e_bc@TPR[id_bc]+e_bc@TNR[id_bc]-1),i)
eval_Bioclim <- rbind(eval_Bioclim, eval_bc_rep)

write.csv(eval_Bioclim, file = 'eval_Bioclim_{0}_{1}.csv',row.names = F)

#projections
pred_cur_bc <- predict(cur_bios,bc_model,progress='')
writeRaster(pred_cur_bc, filename = 'cur_bios_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_bc)

pred_y_10_bc <- predict(y_10, bc_model, progress = '')
writeRaster(pred_y_10_bc, filename='y_010_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_bc)

pred_y_20_bc <- predict(y_20, bc_model, progress = '')
writeRaster(pred_y_20_bc, filename='y_020_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_bc)

pred_y_30_bc <- predict(y_30, bc_model, progress = '')
writeRaster(pred_y_30_bc, filename='y_030_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_bc)

pred_y_40_bc <- predict(y_40, bc_model, progress = '')
writeRaster(pred_y_40_bc, filename='y_040_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_bc)

pred_y_50_bc <- predict(y_50, bc_model, progress = '')
writeRaster(pred_y_50_bc, filename='y_050_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_bc)

pred_y_60_bc <- predict(y_60, bc_model, progress = '')
writeRaster(pred_y_60_bc, filename='y_060_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_bc)

pred_y_70_bc <- predict(y_70, bc_model, progress = '')
writeRaster(pred_y_70_bc, filename='y_070_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_bc)

pred_y_80_bc <- predict(y_80, bc_model, progress = '')
writeRaster(pred_y_80_bc, filename='y_080_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_bc)

pred_y_90_bc <- predict(y_90, bc_model, progress = '')
writeRaster(pred_y_90_bc, filename='y_090_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_bc)

pred_y_100_bc <- predict(y_100, bc_model, progress = '')
writeRaster(pred_y_100_bc, filename='y_100_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_bc)

pred_y_110_bc <- predict(y_110, bc_model, progress = '')
writeRaster(pred_y_110_bc, filename='y_110_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_bc)

pred_y_120_bc <- predict(y_120, bc_model, progress = '')
writeRaster(pred_y_120_bc, filename='y_120_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_bc)

pred_y_130_bc <- predict(y_130, bc_model, progress = '')
writeRaster(pred_y_130_bc, filename='y_130_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_bc)

pred_y_140_bc <- predict(y_140, bc_model, progress = '')
writeRaster(pred_y_140_bc, filename='y_140_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_bc)

pred_y_150_bc <- predict(y_150, bc_model, progress = '')
writeRaster(pred_y_150_bc, filename='y_150_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_bc)

pred_y_160_bc <- predict(y_160, bc_model, progress = '')
writeRaster(pred_y_160_bc, filename='y_160_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_bc)

pred_y_170_bc <- predict(y_170, bc_model, progress = '')
writeRaster(pred_y_170_bc, filename='y_170_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_bc)

pred_y_180_bc <- predict(y_180, bc_model, progress = '')
writeRaster(pred_y_180_bc, filename='y_180_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_bc)

pred_y_190_bc <- predict(y_190, bc_model, progress = '')
writeRaster(pred_y_190_bc, filename='y_190_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_bc)

pred_y_200_bc <- predict(y_200, bc_model, progress = '')
writeRaster(pred_y_200_bc, filename='y_200_Bioclim_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_bc)

""".format(sp, i))

del(sp)
del(i)

file_bioclim.close()

#Gower scripts

for sp in sp_list:
    for i in range(1, 6):
        file_gower = io.open(f"Gower_script_{sp}_{i}.R", "w", newline = "\n")
        file_gower.write ("""library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('/home/marcosvdqueiroz/ENM/chelsa_files/ENM/coords/{0}.txt', header = T, sep = '\\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){{
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}}

i <- {1}

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Gower <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Gower <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ Gower ############ 
#if(rep>1){{gower_ant <- gower_model}}
print(paste('Gower',i))
gower_model <- domain(cur_bios,pres_train)
#if(rep>1){{gower_mean <- mean(gower_ant,gower_model)}}

#dismo::evaluate_models
e_gower <- dismo::evaluate(pres_test,backgr_test,gower_model,cur_bios)
tr_gower <- dismo::threshold(e_gower,'spec_sens')

id_gower <- which(e_gower@t == tr_gower)
eval_gower_rep <- c(e_gower@t[id_gower], e_gower@auc, (e_gower@TPR[id_gower]+e_gower@TNR[id_gower]-1),i)
eval_Gower <- rbind(eval_Gower, eval_gower_rep)
write.csv(eval_Gower, file = 'eval_Gower_{0}_{1}.csv', row.names = F)

#projections
pred_cur_gower <- predict(cur_bios,gower_model,progress='')
writeRaster(pred_cur_gower, filename = 'cur_bios_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_gower)

pred_y_10_gower <- predict(y_10, gower_model, progress = '')
writeRaster(pred_y_10_gower, filename='y_010_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_gower)

pred_y_20_gower <- predict(y_20, gower_model, progress = '')
writeRaster(pred_y_20_gower, filename='y_020_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_gower)

pred_y_30_gower <- predict(y_30, gower_model, progress = '')
writeRaster(pred_y_30_gower, filename='y_030_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_gower)

pred_y_40_gower <- predict(y_40, gower_model, progress = '')
writeRaster(pred_y_40_gower, filename='y_040_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_gower)

pred_y_50_gower <- predict(y_50, gower_model, progress = '')
writeRaster(pred_y_50_gower, filename='y_050_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_gower)

pred_y_60_gower <- predict(y_60, gower_model, progress = '')
writeRaster(pred_y_60_gower, filename='y_060_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_gower)

pred_y_70_gower <- predict(y_70, gower_model, progress = '')
writeRaster(pred_y_70_gower, filename='y_070_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_gower)

pred_y_80_gower <- predict(y_80, gower_model, progress = '')
writeRaster(pred_y_80_gower, filename='y_080_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_gower)

pred_y_90_gower <- predict(y_90, gower_model, progress = '')
writeRaster(pred_y_90_gower, filename='y_090_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_gower)

pred_y_100_gower <- predict(y_100, gower_model, progress = '')
writeRaster(pred_y_100_gower, filename='y_100_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_gower)

pred_y_110_gower <- predict(y_110, gower_model, progress = '')
writeRaster(pred_y_110_gower, filename='y_110_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_gower)

pred_y_120_gower <- predict(y_120, gower_model, progress = '')
writeRaster(pred_y_120_gower, filename='y_120_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_gower)

pred_y_130_gower <- predict(y_130, gower_model, progress = '')
writeRaster(pred_y_130_gower, filename='y_130_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_gower)

pred_y_140_gower <- predict(y_140, gower_model, progress = '')
writeRaster(pred_y_140_gower, filename='y_140_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_gower)

pred_y_150_gower <- predict(y_150, gower_model, progress = '')
writeRaster(pred_y_150_gower, filename='y_150_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_gower)

pred_y_160_gower <- predict(y_160, gower_model, progress = '')
writeRaster(pred_y_160_gower, filename='y_160_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_gower)

pred_y_170_gower <- predict(y_170, gower_model, progress = '')
writeRaster(pred_y_170_gower, filename='y_170_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_gower)

pred_y_180_gower <- predict(y_180, gower_model, progress = '')
writeRaster(pred_y_180_gower, filename='y_180_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_gower)

pred_y_190_gower <- predict(y_190, gower_model, progress = '')
writeRaster(pred_y_190_gower, filename='y_190_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_gower)

pred_y_200_gower <- predict(y_200, gower_model, progress = '')
writeRaster(pred_y_200_gower, filename='y_200_Gower_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_gower)

""".format(sp, i))
        
del(sp)
del(i)
file_gower.close()

## Mahalanobis scripts

for sp in sp_list:
    for i in range(1, 6):
        file_maha = io.open(f"Mahalanobis_script_{sp}_{i}.R", "w", newline = "\n")
        file_maha.write ("""library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('/home/marcosvdqueiroz/ENM/chelsa_files/ENM/coords/{0}.txt', header = T, sep = '\\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){{
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}}

i <- {1}

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Mahalanobis <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Mahalanobis <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ mahalanobis ############

#if(rep>1){{mahalanobis_ant <- mahalanobis_model}}
print(paste('Mahalanobis',i))
mahalanobis_model <- mahal(cur_bios,pres_train)
#if(rep>1){{mahalanobis_mean <- mean(mahalanobis_ant,mahalanobis_model)}}

#dismo::evaluate_models
e_mahalanobis <- dismo::evaluate(pres_test,backgr_test,mahalanobis_model,cur_bios)
tr_mahalanobis <- dismo::threshold(e_mahalanobis,'spec_sens')

id_mahalanobis <- which(e_mahalanobis@t == tr_mahalanobis)
eval_mahalanobis_rep <- c(e_mahalanobis@t[id_mahalanobis], e_mahalanobis@auc, (e_mahalanobis@TPR[id_mahalanobis]+e_mahalanobis@TNR[id_mahalanobis]-1),i)
eval_Mahalanobis <- rbind(eval_Mahalanobis, eval_mahalanobis_rep)
write.csv(eval_Mahalanobis, file = 'eval_Mahalanobis_{0}_{1}.csv', row.names = F)

#projections
pred_cur_mahalanobis <- predict(cur_bios,mahalanobis_model,progress='')
writeRaster(pred_cur_mahalanobis, filename = 'cur_bios_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_mahalanobis)

pred_y_10_mahalanobis <- predict(y_10, mahalanobis_model, progress = '')
writeRaster(pred_y_10_mahalanobis, filename='y_010_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_mahalanobis)

pred_y_20_mahalanobis <- predict(y_20, mahalanobis_model, progress = '')
writeRaster(pred_y_20_mahalanobis, filename='y_020_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_mahalanobis)

pred_y_30_mahalanobis <- predict(y_30, mahalanobis_model, progress = '')
writeRaster(pred_y_30_mahalanobis, filename='y_030_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_mahalanobis)

pred_y_40_mahalanobis <- predict(y_40, mahalanobis_model, progress = '')
writeRaster(pred_y_40_mahalanobis, filename='y_040_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_mahalanobis)

pred_y_50_mahalanobis <- predict(y_50, mahalanobis_model, progress = '')
writeRaster(pred_y_50_mahalanobis, filename='y_050_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_mahalanobis)

pred_y_60_mahalanobis <- predict(y_60, mahalanobis_model, progress = '')
writeRaster(pred_y_60_mahalanobis, filename='y_060_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_mahalanobis)

pred_y_70_mahalanobis <- predict(y_70, mahalanobis_model, progress = '')
writeRaster(pred_y_70_mahalanobis, filename='y_070_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_mahalanobis)

pred_y_80_mahalanobis <- predict(y_80, mahalanobis_model, progress = '')
writeRaster(pred_y_80_mahalanobis, filename='y_080_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_mahalanobis)

pred_y_90_mahalanobis <- predict(y_90, mahalanobis_model, progress = '')
writeRaster(pred_y_90_mahalanobis, filename='y_090_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_mahalanobis)

pred_y_100_mahalanobis <- predict(y_100, mahalanobis_model, progress = '')
writeRaster(pred_y_100_mahalanobis, filename='y_100_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_mahalanobis)

pred_y_110_mahalanobis <- predict(y_110, mahalanobis_model, progress = '')
writeRaster(pred_y_110_mahalanobis, filename='y_110_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_mahalanobis)

pred_y_120_mahalanobis <- predict(y_120, mahalanobis_model, progress = '')
writeRaster(pred_y_120_mahalanobis, filename='y_120_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_mahalanobis)

pred_y_130_mahalanobis <- predict(y_130, mahalanobis_model, progress = '')
writeRaster(pred_y_130_mahalanobis, filename='y_130_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_mahalanobis)

pred_y_140_mahalanobis <- predict(y_140, mahalanobis_model, progress = '')
writeRaster(pred_y_140_mahalanobis, filename='y_140_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_mahalanobis)

pred_y_150_mahalanobis <- predict(y_150, mahalanobis_model, progress = '')
writeRaster(pred_y_150_mahalanobis, filename='y_150_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_mahalanobis)

pred_y_160_mahalanobis <- predict(y_160, mahalanobis_model, progress = '')
writeRaster(pred_y_160_mahalanobis, filename='y_160_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_mahalanobis)

pred_y_170_mahalanobis <- predict(y_170, mahalanobis_model, progress = '')
writeRaster(pred_y_170_mahalanobis, filename='y_170_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_mahalanobis)

pred_y_180_mahalanobis <- predict(y_180, mahalanobis_model, progress = '')
writeRaster(pred_y_180_mahalanobis, filename='y_180_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_mahalanobis)

pred_y_190_mahalanobis <- predict(y_190, mahalanobis_model, progress = '')
writeRaster(pred_y_190_mahalanobis, filename='y_190_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_mahalanobis)

pred_y_200_mahalanobis <- predict(y_200, mahalanobis_model, progress = '')
writeRaster(pred_y_200_mahalanobis, filename='y_200_Mahalanobis_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_mahalanobis)

""".format(sp, i))
                         
del(sp)
del(i)
file_maha.close()

#SVM scripts

for sp in sp_list:
    for i in range(1, 6):
        file_svm = io.open(f"SVM_script_{sp}_{i}.R", "w", newline = "\n")
        file_svm.write ("""library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('/home/marcosvdqueiroz/ENM/chelsa_files/ENM/coords/{0}.txt', header = T, sep = '\\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){{
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}}

i <- {1}

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_SVM <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_SVM <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

############ SVM ############
#if(rep>1){{svm_ant <- svm_model}}
print(paste('SVM',i))
svm_model <- ksvm(pa ~ sa_CHELSA_TraCE21k_bio07_0_V1.0 + sa_CHELSA_TraCE21k_bio10_0_V1.0 + sa_CHELSA_TraCE21k_bio13_0_V1.0 + sa_CHELSA_TraCE21k_bio14_0_V1.0 +sa_CHELSA_TraCE21k_bio19_0_V1.0,data=envtrain)
#if(rep>1){{svm_mean <- mean(svm_ant,svm_model)}}

#dismo::evaluate_models
e_svm <- dismo::evaluate(pres_test,backgr_test,svm_model,cur_bios)
tr_svm <- dismo::threshold(e_svm,'spec_sens')

id_svm <- which(e_svm@t == tr_svm)
eval_svm_rep <- c(e_svm@t[id_svm], e_svm@auc, (e_svm@TPR[id_svm]+e_svm@TNR[id_svm]-1),i)
eval_SVM <- rbind(eval_SVM, eval_svm_rep)
write.csv(eval_SVM, file = 'eval_SVM_{0}_{1}.csv',row.names = F)


#projections
pred_cur_svm <- predict(cur_bios,svm_model,progress='')
writeRaster(pred_cur_svm, filename = 'cur_bios_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_svm)

pred_y_10_svm <- predict(y_10, svm_model, progress = '')
writeRaster(pred_y_10_svm, filename='y_010_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_svm)

pred_y_20_svm <- predict(y_20, svm_model, progress = '')
writeRaster(pred_y_20_svm, filename='y_020_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_svm)

pred_y_30_svm <- predict(y_30, svm_model, progress = '')
writeRaster(pred_y_30_svm, filename='y_030_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_svm)

pred_y_40_svm <- predict(y_40, svm_model, progress = '')
writeRaster(pred_y_40_svm, filename='y_040_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_svm)

pred_y_50_svm <- predict(y_50, svm_model, progress = '')
writeRaster(pred_y_50_svm, filename='y_050_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_svm)

pred_y_60_svm <- predict(y_60, svm_model, progress = '')
writeRaster(pred_y_60_svm, filename='y_060_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_svm)

pred_y_70_svm <- predict(y_70, svm_model, progress = '')
writeRaster(pred_y_70_svm, filename='y_070_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_svm)

pred_y_80_svm <- predict(y_80, svm_model, progress = '')
writeRaster(pred_y_80_svm, filename='y_080_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_svm)

pred_y_90_svm <- predict(y_90, svm_model, progress = '')
writeRaster(pred_y_90_svm, filename='y_090_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_svm)

pred_y_100_svm <- predict(y_100, svm_model, progress = '')
writeRaster(pred_y_100_svm, filename='y_100_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_svm)

pred_y_110_svm <- predict(y_110, svm_model, progress = '')
writeRaster(pred_y_110_svm, filename='y_110_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_svm)

pred_y_120_svm <- predict(y_120, svm_model, progress = '')
writeRaster(pred_y_120_svm, filename='y_120_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_svm)

pred_y_130_svm <- predict(y_130, svm_model, progress = '')
writeRaster(pred_y_130_svm, filename='y_130_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_svm)

pred_y_140_svm <- predict(y_140, svm_model, progress = '')
writeRaster(pred_y_140_svm, filename='y_140_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_svm)

pred_y_150_svm <- predict(y_150, svm_model, progress = '')
writeRaster(pred_y_150_svm, filename='y_150_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_svm)

pred_y_160_svm <- predict(y_160, svm_model, progress = '')
writeRaster(pred_y_160_svm, filename='y_160_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_svm)

pred_y_170_svm <- predict(y_170, svm_model, progress = '')
writeRaster(pred_y_170_svm, filename='y_170_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_svm)

pred_y_180_svm <- predict(y_180, svm_model, progress = '')
writeRaster(pred_y_180_svm, filename='y_180_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_svm)

pred_y_190_svm <- predict(y_190, svm_model, progress = '')
writeRaster(pred_y_190_svm, filename='y_190_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_svm)

pred_y_200_svm <- predict(y_200, svm_model, progress = '')
writeRaster(pred_y_200_svm, filename='y_200_SVM_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_svm)

""".format(sp, i))
        
del(sp)
del(i)
file_svm.close()

#Scripts random forest

for sp in sp_list:
    for i in range(1, 6):
        file_rdf = io.open(f"RandomForest_script_{sp}_{i}.R", "w", newline = "\n")
        file_rdf.write ("""library(raster)
library(dismo)
library(maptools)
library(sp)
library(gam)
library(randomForest)
library(kernlab)
library(vegan)

#check the pathname
points <- read.table('/home/marcosvdqueiroz/ENM/chelsa_files/ENM/coords/{0}.txt', header = T, sep = '\\t')

data(wrld_simpl)

coordinates(points) <- ~decimalLongitude+decimalLatitude
crs(points) <- crs(wrld_simpl)

my_files <- list.files(pattern = '*.tif', full.names = TRUE)

a_0 <- grep(paste('_0_', sep = ''), my_files, value = TRUE)

#By 1000 years
for(j in seq(10, 200, by = 10)){{
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_-', j, '_', sep = ''), my_files, value = TRUE))
}}

i <- {1}

cur_bios <- stack(a_0)
y_10 <- stack(a_10)
y_20 <- stack(a_20)
y_30 <- stack(a_30)
y_40 <- stack(a_40)
y_50 <- stack(a_50)
y_60 <- stack(a_60)
y_70 <- stack(a_70)
y_80 <- stack(a_80)
y_90 <- stack(a_90)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

names(y_10) <-names(y_20) <-names(y_30) <-names(y_40) <-names(y_50) <-names(y_60) <-names(y_70) <-names(y_80) <-names(y_90) <-names(y_100) <-names(y_110) <-names(y_120) <-names(y_130) <-names(y_140) <-names(y_150) <-names(y_160) <-names(y_170) <-names(y_180) <-names(y_190) <-names(y_200) <- names(cur_bios)


val_cur_bios <- as.data.frame(extract(cur_bios,points))
dup_cur_bios <- duplicated(val_cur_bios)

eval_Randomforest <- data.frame(matrix(ncol = 4,nrow = 0))

eval_names <- c('thr','AU','TS','rep')

eval_Randomforest <- eval_names

backgr <- randomPoints(cur_bios,100000)
absvals <- extract(cur_bios, backgr)
dim(backgr)
dim(absvals)

## Train and test precences and background
#####
pres_group <- kfold(points,5)
pres_train <- data.frame(points[pres_group != 1,])
pres_test <- data.frame(points[pres_group == 1,])

backgr_group <- kfold(backgr,5)
backgr_train <- data.frame(backgr[backgr_group != 1,]); names(backgr_train) <- names(pres_train)
backgr_test <- backgr[backgr_group == 1,]

train <- rbind(pres_train,backgr_train)
pb_train <- c(rep(1, nrow(pres_train)), rep(0, nrow(backgr_train)))
envtrain <- extract(cur_bios, train)
envtrain <- data.frame(cbind(pa=pb_train, envtrain))

## Random Forest
#if(rep>1){{randforest_ant <- randforest_model}}
print(paste('Random Forest',i))
randforest_model <- randomForest(pa ~ sa_CHELSA_TraCE21k_bio07_0_V1.0 + sa_CHELSA_TraCE21k_bio10_0_V1.0 + sa_CHELSA_TraCE21k_bio13_0_V1.0 + sa_CHELSA_TraCE21k_bio14_0_V1.0 +sa_CHELSA_TraCE21k_bio19_0_V1.0,data=envtrain)
#if(rep>1){{randforest_mean <- mean(randforest_ant,randforest_model)}}

#dismo::evaluate_models
e_randomforest <- dismo::evaluate(pres_test,backgr_test,randforest_model,cur_bios)
tr_randomforest <- dismo::threshold(e_randomforest,'spec_sens')

id_randomforest <- which(e_randomforest@t == tr_randomforest)
eval_randomforest_rep <- c(e_randomforest@t[id_randomforest], e_randomforest@auc, (e_randomforest@TPR[id_randomforest]+e_randomforest@TNR[id_randomforest]-1),i)
eval_Randomforest <- rbind(eval_Randomforest, eval_randomforest_rep)
write.csv(eval_Randomforest, file = 'eval_RandomForest_{0}_{1}.csv',row.names = F)


#projections
pred_cur_randforest <- predict(cur_bios,randforest_model,progress='')
writeRaster(pred_cur_randforest, filename = 'cur_bios_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_cur_randforest)

pred_y_10_randforest <- predict(y_10, randforest_model, progress = '')
writeRaster(pred_y_10_randforest, filename='y_010_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_10_randforest)

pred_y_20_randforest <- predict(y_20, randforest_model, progress = '')
writeRaster(pred_y_20_randforest, filename='y_020_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_20_randforest)

pred_y_30_randforest <- predict(y_30, randforest_model, progress = '')
writeRaster(pred_y_30_randforest, filename='y_030_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_30_randforest)

pred_y_40_randforest <- predict(y_40, randforest_model, progress = '')
writeRaster(pred_y_40_randforest, filename='y_040_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_40_randforest)

pred_y_50_randforest <- predict(y_50, randforest_model, progress = '')
writeRaster(pred_y_50_randforest, filename='y_050_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_50_randforest)

pred_y_60_randforest <- predict(y_60, randforest_model, progress = '')
writeRaster(pred_y_60_randforest, filename='y_060_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_60_randforest)

pred_y_70_randforest <- predict(y_70, randforest_model, progress = '')
writeRaster(pred_y_70_randforest, filename='y_070_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_70_randforest)

pred_y_80_randforest <- predict(y_80, randforest_model, progress = '')
writeRaster(pred_y_80_randforest, filename='y_080_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_80_randforest)

pred_y_90_randforest <- predict(y_90, randforest_model, progress = '')
writeRaster(pred_y_90_randforest, filename='y_090_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_90_randforest)

pred_y_100_randforest <- predict(y_100, randforest_model, progress = '')
writeRaster(pred_y_100_randforest, filename='y_100_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_100_randforest)

pred_y_110_randforest <- predict(y_110, randforest_model, progress = '')
writeRaster(pred_y_110_randforest, filename='y_110_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_110_randforest)

pred_y_120_randforest <- predict(y_120, randforest_model, progress = '')
writeRaster(pred_y_120_randforest, filename='y_120_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_120_randforest)

pred_y_130_randforest <- predict(y_130, randforest_model, progress = '')
writeRaster(pred_y_130_randforest, filename='y_130_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_130_randforest)

pred_y_140_randforest <- predict(y_140, randforest_model, progress = '')
writeRaster(pred_y_140_randforest, filename='y_140_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_140_randforest)

pred_y_150_randforest <- predict(y_150, randforest_model, progress = '')
writeRaster(pred_y_150_randforest, filename='y_150_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_150_randforest)

pred_y_160_randforest <- predict(y_160, randforest_model, progress = '')
writeRaster(pred_y_160_randforest, filename='y_160_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_160_randforest)

pred_y_170_randforest <- predict(y_170, randforest_model, progress = '')
writeRaster(pred_y_170_randforest, filename='y_170_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_170_randforest)

pred_y_180_randforest <- predict(y_180, randforest_model, progress = '')
writeRaster(pred_y_180_randforest, filename='y_180_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_180_randforest)

pred_y_190_randforest <- predict(y_190, randforest_model, progress = '')
writeRaster(pred_y_190_randforest, filename='y_190_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_190_randforest)

pred_y_200_randforest <- predict(y_200, randforest_model, progress = '')
writeRaster(pred_y_200_randforest, filename='y_200_RandomForest_{0}_{1}', bylayer = TRUE, format = 'GTiff', overwrite = TRUE)
rm(pred_y_200_randforest)

""".format(sp, i))
        
del(sp)
del(i)
file_rdf.close()

#Submit jobs sh

algorithms = ['Bioclim', 'Gower', 'Mahalanobis', 'SVM', 'RandomForest']

for sp in sp_list:
    for i in range(1, 6):
        for a in algorithms:
            file_sh = io.open(f"submit_{a}_{sp}_{i}.sh", "w", newline = "\n")
            #print(f'submit_{sp}_{i}_{a}.sh')
            if a == 'Mahalanobis':
                file_sh.write("""#!/bin/bash
#SBATCH -t 120:00:00 -c 4

export INPUT="{0}_script_{1}_{2}.R sa_CHELSA_TraCE21k_bio07_-10_V1.0.tif sa_CHELSA_TraCE21k_bio07_-100_V1.0.tif sa_CHELSA_TraCE21k_bio07_-110_V1.0.tif sa_CHELSA_TraCE21k_bio07_-120_V1.0.tif sa_CHELSA_TraCE21k_bio07_-130_V1.0.tif sa_CHELSA_TraCE21k_bio07_-140_V1.0.tif sa_CHELSA_TraCE21k_bio07_-150_V1.0.tif sa_CHELSA_TraCE21k_bio07_-160_V1.0.tif sa_CHELSA_TraCE21k_bio07_-170_V1.0.tif sa_CHELSA_TraCE21k_bio07_-180_V1.0.tif sa_CHELSA_TraCE21k_bio07_-190_V1.0.tif sa_CHELSA_TraCE21k_bio07_-20_V1.0.tif sa_CHELSA_TraCE21k_bio07_-200_V1.0.tif sa_CHELSA_TraCE21k_bio07_-30_V1.0.tif sa_CHELSA_TraCE21k_bio07_-40_V1.0.tif sa_CHELSA_TraCE21k_bio07_-50_V1.0.tif sa_CHELSA_TraCE21k_bio07_-60_V1.0.tif sa_CHELSA_TraCE21k_bio07_-70_V1.0.tif sa_CHELSA_TraCE21k_bio07_-80_V1.0.tif sa_CHELSA_TraCE21k_bio07_-90_V1.0.tif sa_CHELSA_TraCE21k_bio07_0_V1.0.tif sa_CHELSA_TraCE21k_bio10_-10_V1.0.tif sa_CHELSA_TraCE21k_bio10_-100_V1.0.tif sa_CHELSA_TraCE21k_bio10_-110_V1.0.tif sa_CHELSA_TraCE21k_bio10_-120_V1.0.tif sa_CHELSA_TraCE21k_bio10_-130_V1.0.tif sa_CHELSA_TraCE21k_bio10_-140_V1.0.tif sa_CHELSA_TraCE21k_bio10_-150_V1.0.tif sa_CHELSA_TraCE21k_bio10_-160_V1.0.tif sa_CHELSA_TraCE21k_bio10_-170_V1.0.tif sa_CHELSA_TraCE21k_bio10_-180_V1.0.tif sa_CHELSA_TraCE21k_bio10_-190_V1.0.tif sa_CHELSA_TraCE21k_bio10_-20_V1.0.tif sa_CHELSA_TraCE21k_bio10_-200_V1.0.tif sa_CHELSA_TraCE21k_bio10_-30_V1.0.tif sa_CHELSA_TraCE21k_bio10_-40_V1.0.tif sa_CHELSA_TraCE21k_bio10_-50_V1.0.tif sa_CHELSA_TraCE21k_bio10_-60_V1.0.tif sa_CHELSA_TraCE21k_bio10_-70_V1.0.tif sa_CHELSA_TraCE21k_bio10_-80_V1.0.tif sa_CHELSA_TraCE21k_bio10_-90_V1.0.tif sa_CHELSA_TraCE21k_bio10_0_V1.0.tif sa_CHELSA_TraCE21k_bio13_-10_V1.0.tif sa_CHELSA_TraCE21k_bio13_-100_V1.0.tif sa_CHELSA_TraCE21k_bio13_-110_V1.0.tif sa_CHELSA_TraCE21k_bio13_-120_V1.0.tif sa_CHELSA_TraCE21k_bio13_-130_V1.0.tif sa_CHELSA_TraCE21k_bio13_-140_V1.0.tif sa_CHELSA_TraCE21k_bio13_-150_V1.0.tif sa_CHELSA_TraCE21k_bio13_-160_V1.0.tif sa_CHELSA_TraCE21k_bio13_-170_V1.0.tif sa_CHELSA_TraCE21k_bio13_-180_V1.0.tif sa_CHELSA_TraCE21k_bio13_-190_V1.0.tif sa_CHELSA_TraCE21k_bio13_-20_V1.0.tif sa_CHELSA_TraCE21k_bio13_-200_V1.0.tif sa_CHELSA_TraCE21k_bio13_-30_V1.0.tif sa_CHELSA_TraCE21k_bio13_-40_V1.0.tif sa_CHELSA_TraCE21k_bio13_-50_V1.0.tif sa_CHELSA_TraCE21k_bio13_-60_V1.0.tif sa_CHELSA_TraCE21k_bio13_-70_V1.0.tif sa_CHELSA_TraCE21k_bio13_-80_V1.0.tif sa_CHELSA_TraCE21k_bio13_-90_V1.0.tif sa_CHELSA_TraCE21k_bio13_0_V1.0.tif sa_CHELSA_TraCE21k_bio14_-10_V1.0.tif sa_CHELSA_TraCE21k_bio14_-100_V1.0.tif sa_CHELSA_TraCE21k_bio14_-110_V1.0.tif sa_CHELSA_TraCE21k_bio14_-120_V1.0.tif sa_CHELSA_TraCE21k_bio14_-130_V1.0.tif sa_CHELSA_TraCE21k_bio14_-140_V1.0.tif sa_CHELSA_TraCE21k_bio14_-150_V1.0.tif sa_CHELSA_TraCE21k_bio14_-160_V1.0.tif sa_CHELSA_TraCE21k_bio14_-170_V1.0.tif sa_CHELSA_TraCE21k_bio14_-180_V1.0.tif sa_CHELSA_TraCE21k_bio14_-190_V1.0.tif sa_CHELSA_TraCE21k_bio14_-20_V1.0.tif sa_CHELSA_TraCE21k_bio14_-200_V1.0.tif sa_CHELSA_TraCE21k_bio14_-30_V1.0.tif sa_CHELSA_TraCE21k_bio14_-40_V1.0.tif sa_CHELSA_TraCE21k_bio14_-50_V1.0.tif sa_CHELSA_TraCE21k_bio14_-60_V1.0.tif sa_CHELSA_TraCE21k_bio14_-70_V1.0.tif sa_CHELSA_TraCE21k_bio14_-80_V1.0.tif sa_CHELSA_TraCE21k_bio14_-90_V1.0.tif sa_CHELSA_TraCE21k_bio14_0_V1.0.tif sa_CHELSA_TraCE21k_bio19_-10_V1.0.tif sa_CHELSA_TraCE21k_bio19_-100_V1.0.tif sa_CHELSA_TraCE21k_bio19_-110_V1.0.tif sa_CHELSA_TraCE21k_bio19_-120_V1.0.tif sa_CHELSA_TraCE21k_bio19_-130_V1.0.tif sa_CHELSA_TraCE21k_bio19_-140_V1.0.tif sa_CHELSA_TraCE21k_bio19_-150_V1.0.tif sa_CHELSA_TraCE21k_bio19_-160_V1.0.tif sa_CHELSA_TraCE21k_bio19_-170_V1.0.tif sa_CHELSA_TraCE21k_bio19_-180_V1.0.tif sa_CHELSA_TraCE21k_bio19_-190_V1.0.tif sa_CHELSA_TraCE21k_bio19_-20_V1.0.tif sa_CHELSA_TraCE21k_bio19_-200_V1.0.tif sa_CHELSA_TraCE21k_bio19_-30_V1.0.tif sa_CHELSA_TraCE21k_bio19_-40_V1.0.tif sa_CHELSA_TraCE21k_bio19_-50_V1.0.tif sa_CHELSA_TraCE21k_bio19_-60_V1.0.tif sa_CHELSA_TraCE21k_bio19_-70_V1.0.tif sa_CHELSA_TraCE21k_bio19_-80_V1.0.tif sa_CHELSA_TraCE21k_bio19_-90_V1.0.tif sa_CHELSA_TraCE21k_bio19_0_V1.0.tif"
export OUTPUT="eval_{0}_{1}_{2}.csv y_010_{0}_{1}_{2}.tif y_020_{0}_{1}_{2}.tif y_030_{0}_{1}_{2}.tif y_040_{0}_{1}_{2}.tif y_050_{0}_{1}_{2}.tif y_060_{0}_{1}_{2}.tif y_070_{0}_{1}_{2}.tif y_080_{0}_{1}_{2}.tif y_090_{0}_{1}_{2}.tif y_100_{0}_{1}_{2}.tif y_110_{0}_{1}_{2}.tif y_120_{0}_{1}_{2}.tif y_130_{0}_{1}_{2}.tif y_140_{0}_{1}_{2}.tif y_150_{0}_{1}_{2}.tif y_160_{0}_{1}_{2}.tif y_170_{0}_{1}_{2}.tif y_180_{0}_{1}_{2}.tif y_190_{0}_{1}_{2}.tif y_200_{0}_{1}_{2}.tif cur_bios_{0}_{1}_{2}.tif"

module load miniconda/3
source activate ENM

job-nanny srun Rscript {0}_script_{1}_{2}.R --input=sa_CHELSA_TraCE21k_bio07_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio07_0_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio10_0_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio13_0_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio14_0_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio19_0_V1.0.tif
""".format(a, sp, i))
            else:
                file_sh.write("""#!/bin/bash
#SBATCH -t 60:00:00 -c 4

export INPUT="{0}_script_{1}_{2}.R sa_CHELSA_TraCE21k_bio07_-10_V1.0.tif sa_CHELSA_TraCE21k_bio07_-100_V1.0.tif sa_CHELSA_TraCE21k_bio07_-110_V1.0.tif sa_CHELSA_TraCE21k_bio07_-120_V1.0.tif sa_CHELSA_TraCE21k_bio07_-130_V1.0.tif sa_CHELSA_TraCE21k_bio07_-140_V1.0.tif sa_CHELSA_TraCE21k_bio07_-150_V1.0.tif sa_CHELSA_TraCE21k_bio07_-160_V1.0.tif sa_CHELSA_TraCE21k_bio07_-170_V1.0.tif sa_CHELSA_TraCE21k_bio07_-180_V1.0.tif sa_CHELSA_TraCE21k_bio07_-190_V1.0.tif sa_CHELSA_TraCE21k_bio07_-20_V1.0.tif sa_CHELSA_TraCE21k_bio07_-200_V1.0.tif sa_CHELSA_TraCE21k_bio07_-30_V1.0.tif sa_CHELSA_TraCE21k_bio07_-40_V1.0.tif sa_CHELSA_TraCE21k_bio07_-50_V1.0.tif sa_CHELSA_TraCE21k_bio07_-60_V1.0.tif sa_CHELSA_TraCE21k_bio07_-70_V1.0.tif sa_CHELSA_TraCE21k_bio07_-80_V1.0.tif sa_CHELSA_TraCE21k_bio07_-90_V1.0.tif sa_CHELSA_TraCE21k_bio07_0_V1.0.tif sa_CHELSA_TraCE21k_bio10_-10_V1.0.tif sa_CHELSA_TraCE21k_bio10_-100_V1.0.tif sa_CHELSA_TraCE21k_bio10_-110_V1.0.tif sa_CHELSA_TraCE21k_bio10_-120_V1.0.tif sa_CHELSA_TraCE21k_bio10_-130_V1.0.tif sa_CHELSA_TraCE21k_bio10_-140_V1.0.tif sa_CHELSA_TraCE21k_bio10_-150_V1.0.tif sa_CHELSA_TraCE21k_bio10_-160_V1.0.tif sa_CHELSA_TraCE21k_bio10_-170_V1.0.tif sa_CHELSA_TraCE21k_bio10_-180_V1.0.tif sa_CHELSA_TraCE21k_bio10_-190_V1.0.tif sa_CHELSA_TraCE21k_bio10_-20_V1.0.tif sa_CHELSA_TraCE21k_bio10_-200_V1.0.tif sa_CHELSA_TraCE21k_bio10_-30_V1.0.tif sa_CHELSA_TraCE21k_bio10_-40_V1.0.tif sa_CHELSA_TraCE21k_bio10_-50_V1.0.tif sa_CHELSA_TraCE21k_bio10_-60_V1.0.tif sa_CHELSA_TraCE21k_bio10_-70_V1.0.tif sa_CHELSA_TraCE21k_bio10_-80_V1.0.tif sa_CHELSA_TraCE21k_bio10_-90_V1.0.tif sa_CHELSA_TraCE21k_bio10_0_V1.0.tif sa_CHELSA_TraCE21k_bio13_-10_V1.0.tif sa_CHELSA_TraCE21k_bio13_-100_V1.0.tif sa_CHELSA_TraCE21k_bio13_-110_V1.0.tif sa_CHELSA_TraCE21k_bio13_-120_V1.0.tif sa_CHELSA_TraCE21k_bio13_-130_V1.0.tif sa_CHELSA_TraCE21k_bio13_-140_V1.0.tif sa_CHELSA_TraCE21k_bio13_-150_V1.0.tif sa_CHELSA_TraCE21k_bio13_-160_V1.0.tif sa_CHELSA_TraCE21k_bio13_-170_V1.0.tif sa_CHELSA_TraCE21k_bio13_-180_V1.0.tif sa_CHELSA_TraCE21k_bio13_-190_V1.0.tif sa_CHELSA_TraCE21k_bio13_-20_V1.0.tif sa_CHELSA_TraCE21k_bio13_-200_V1.0.tif sa_CHELSA_TraCE21k_bio13_-30_V1.0.tif sa_CHELSA_TraCE21k_bio13_-40_V1.0.tif sa_CHELSA_TraCE21k_bio13_-50_V1.0.tif sa_CHELSA_TraCE21k_bio13_-60_V1.0.tif sa_CHELSA_TraCE21k_bio13_-70_V1.0.tif sa_CHELSA_TraCE21k_bio13_-80_V1.0.tif sa_CHELSA_TraCE21k_bio13_-90_V1.0.tif sa_CHELSA_TraCE21k_bio13_0_V1.0.tif sa_CHELSA_TraCE21k_bio14_-10_V1.0.tif sa_CHELSA_TraCE21k_bio14_-100_V1.0.tif sa_CHELSA_TraCE21k_bio14_-110_V1.0.tif sa_CHELSA_TraCE21k_bio14_-120_V1.0.tif sa_CHELSA_TraCE21k_bio14_-130_V1.0.tif sa_CHELSA_TraCE21k_bio14_-140_V1.0.tif sa_CHELSA_TraCE21k_bio14_-150_V1.0.tif sa_CHELSA_TraCE21k_bio14_-160_V1.0.tif sa_CHELSA_TraCE21k_bio14_-170_V1.0.tif sa_CHELSA_TraCE21k_bio14_-180_V1.0.tif sa_CHELSA_TraCE21k_bio14_-190_V1.0.tif sa_CHELSA_TraCE21k_bio14_-20_V1.0.tif sa_CHELSA_TraCE21k_bio14_-200_V1.0.tif sa_CHELSA_TraCE21k_bio14_-30_V1.0.tif sa_CHELSA_TraCE21k_bio14_-40_V1.0.tif sa_CHELSA_TraCE21k_bio14_-50_V1.0.tif sa_CHELSA_TraCE21k_bio14_-60_V1.0.tif sa_CHELSA_TraCE21k_bio14_-70_V1.0.tif sa_CHELSA_TraCE21k_bio14_-80_V1.0.tif sa_CHELSA_TraCE21k_bio14_-90_V1.0.tif sa_CHELSA_TraCE21k_bio14_0_V1.0.tif sa_CHELSA_TraCE21k_bio19_-10_V1.0.tif sa_CHELSA_TraCE21k_bio19_-100_V1.0.tif sa_CHELSA_TraCE21k_bio19_-110_V1.0.tif sa_CHELSA_TraCE21k_bio19_-120_V1.0.tif sa_CHELSA_TraCE21k_bio19_-130_V1.0.tif sa_CHELSA_TraCE21k_bio19_-140_V1.0.tif sa_CHELSA_TraCE21k_bio19_-150_V1.0.tif sa_CHELSA_TraCE21k_bio19_-160_V1.0.tif sa_CHELSA_TraCE21k_bio19_-170_V1.0.tif sa_CHELSA_TraCE21k_bio19_-180_V1.0.tif sa_CHELSA_TraCE21k_bio19_-190_V1.0.tif sa_CHELSA_TraCE21k_bio19_-20_V1.0.tif sa_CHELSA_TraCE21k_bio19_-200_V1.0.tif sa_CHELSA_TraCE21k_bio19_-30_V1.0.tif sa_CHELSA_TraCE21k_bio19_-40_V1.0.tif sa_CHELSA_TraCE21k_bio19_-50_V1.0.tif sa_CHELSA_TraCE21k_bio19_-60_V1.0.tif sa_CHELSA_TraCE21k_bio19_-70_V1.0.tif sa_CHELSA_TraCE21k_bio19_-80_V1.0.tif sa_CHELSA_TraCE21k_bio19_-90_V1.0.tif sa_CHELSA_TraCE21k_bio19_0_V1.0.tif"
export OUTPUT="eval_{0}_{1}_{2}.csv y_010_{0}_{1}_{2}.tif y_020_{0}_{1}_{2}.tif y_030_{0}_{1}_{2}.tif y_040_{0}_{1}_{2}.tif y_050_{0}_{1}_{2}.tif y_060_{0}_{1}_{2}.tif y_070_{0}_{1}_{2}.tif y_080_{0}_{1}_{2}.tif y_090_{0}_{1}_{2}.tif y_100_{0}_{1}_{2}.tif y_110_{0}_{1}_{2}.tif y_120_{0}_{1}_{2}.tif y_130_{0}_{1}_{2}.tif y_140_{0}_{1}_{2}.tif y_150_{0}_{1}_{2}.tif y_160_{0}_{1}_{2}.tif y_170_{0}_{1}_{2}.tif y_180_{0}_{1}_{2}.tif y_190_{0}_{1}_{2}.tif y_200_{0}_{1}_{2}.tif cur_bios_{0}_{1}_{2}.tif"

module load miniconda/3
source activate ENM

job-nanny srun Rscript {0}_script_{1}_{2}.R --input=sa_CHELSA_TraCE21k_bio07_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio07_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio07_0_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio10_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio10_0_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio13_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio13_0_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio14_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio14_0_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-10_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-100_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-110_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-120_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-130_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-140_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-150_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-160_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-170_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-180_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-190_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-20_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-200_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-30_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-40_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-50_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-60_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-70_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-80_V1.0.tif,sa_CHELSA_TraCE21k_bio19_-90_V1.0.tif,sa_CHELSA_TraCE21k_bio19_0_V1.0.tif
""".format(a, sp, i))
            
del(a)
del(sp)
del(i)

file_sh.close()
```

# 3. Ensemble

After all modeling is done, we can combine the multiple types of
algorithms to produce a ???average??? model for each time slice. This can be
done using the script below (for each species, of course).

``` r
library(raster)

#Grep all evaluation files

eval_bioclim_files <- Sys.glob('*Bioclim*csv')
eval_gower_files <- Sys.glob('*Gower*csv')
eval_mahalanobis_files <- Sys.glob('*Mahalanobis*csv')
eval_randomforest_files <- Sys.glob('*RandomForest*csv')
eval_svm_files <- Sys.glob('*SVM*csv')

#Create new empty dataframes

eval_Bioclim <- data.frame()
eval_Gower <- data.frame()
eval_Mahalanobis <- data.frame()
eval_Randomforest <- data.frame()
eval_SVM <- data.frame()

#For each one of the evaluate files, populate the empty dataframes

#Bioclim
for(i in 1:length(eval_bioclim_files)){
  f <- read.csv(eval_bioclim_files[i], skip = 1)
  eval_Bioclim <- rbind(eval_Bioclim, f)
}

#Gower
for(i in 1:length(eval_gower_files)){
  f <- read.csv(eval_gower_files[i], skip = 1)
  eval_Gower <- rbind(eval_Gower, f)
}

#Mahalanobis
for(i in 1:length(eval_mahalanobis_files)){
  f <- read.csv(eval_mahalanobis_files[i], skip = 1)
  eval_Mahalanobis <- rbind(eval_Mahalanobis, f)
}

#Randomforest
for(i in 1:length(eval_randomforest_files)){
  f <- read.csv(eval_randomforest_files[i], skip = 1)
  eval_Randomforest <- rbind(eval_Randomforest, f)
}

#SVM
for(i in 1:length(eval_svm_files)){
  f <- read.csv(eval_svm_files[i], skip = 1)
  eval_SVM <- rbind(eval_SVM, f)
}

#Make sure that all names are the same
names(eval_Bioclim) <- names(eval_Gower) <- names(eval_Mahalanobis)  <- names(eval_Randomforest) <- names(eval_SVM) <- c('thrs','AUC','TSS','rep')


#Bind all evaluation in one single dataframe
eval <- data.frame(rbind(as.data.frame(eval_Bioclim),as.data.frame(eval_Gower), as.data.frame(eval_Mahalanobis),
                         as.data.frame(eval_Randomforest),as.data.frame(eval_SVM)))

eval$thrs <- as.numeric(as.character(eval$thrs))
eval$AUC <- as.numeric(as.character(eval$AUC))
eval$TSS <- as.numeric(as.character(eval$TSS))

#Lets begin the ensemble process. First, by the current predictions

### current ####
cur <- list.files(pattern='cur_', full.names=TRUE )
cur <- stack(cur)
cur_mean_tss <- weighted.mean(cur > eval$thrs,eval$TSS)
#plot(cur_mean_tss)
writeRaster(cur_mean_tss, filename='ensemble_cur_weighted_mean_Bokermannohyla_alvarengai', format='GTiff', overwrite=TRUE)
rm(cur_mean_tss)

#creating a seq with leading zeros

my_seq <- seq(10, 200, by = 10)
my_seq <- sprintf('%0.3d', my_seq)

#Listing all files
my_files <- list.files(pattern = '*.tif')


#By 1000 years
for(j in my_seq){
  nam <- paste('a', j, sep = '_')
  assign(nam, grep(paste('_', j, '_', sep = ''), my_files, value = TRUE))
}

y_010 <- stack(a_010)
y_020 <- stack(a_020)
y_030 <- stack(a_030)
y_040 <- stack(a_040)
y_050 <- stack(a_050)
y_060 <- stack(a_060)
y_070 <- stack(a_070)
y_080 <- stack(a_080)
y_090 <- stack(a_090)
y_100 <- stack(a_100)
y_110 <- stack(a_110)
y_120 <- stack(a_120)
y_130 <- stack(a_130)
y_140 <- stack(a_140)
y_150 <- stack(a_150)
y_160 <- stack(a_160)
y_170 <- stack(a_170)
y_180 <- stack(a_180)
y_190 <- stack(a_190)
y_200 <- stack(a_200)

y_010_mean_tss <- weighted.mean(y_010 > eval$thrs, eval$TSS)
writeRaster(y_010_mean_tss, filename='ensemble_y_010_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_010_mean_tss)

y_020_mean_tss <- weighted.mean(y_020 > eval$thrs, eval$TSS)
writeRaster(y_020_mean_tss, filename='ensemble_y_020_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_020_mean_tss)

y_030_mean_tss <- weighted.mean(y_030 > eval$thrs, eval$TSS)
writeRaster(y_030_mean_tss, filename='ensemble_y_030_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_030_mean_tss)

y_040_mean_tss <- weighted.mean(y_040 > eval$thrs, eval$TSS)
writeRaster(y_040_mean_tss, filename='ensemble_y_040_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_040_mean_tss)

y_050_mean_tss <- weighted.mean(y_050 > eval$thrs, eval$TSS)
writeRaster(y_050_mean_tss, filename='ensemble_y_050_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_050_mean_tss)

y_060_mean_tss <- weighted.mean(y_060 > eval$thrs, eval$TSS)
writeRaster(y_060_mean_tss, filename='ensemble_y_060_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_060_mean_tss)

y_070_mean_tss <- weighted.mean(y_070 > eval$thrs, eval$TSS)
writeRaster(y_070_mean_tss, filename='ensemble_y_070_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_070_mean_tss)

y_080_mean_tss <- weighted.mean(y_080 > eval$thrs, eval$TSS)
writeRaster(y_080_mean_tss, filename='ensemble_y_080_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_080_mean_tss)

y_090_mean_tss <- weighted.mean(y_090 > eval$thrs, eval$TSS)
writeRaster(y_090_mean_tss, filename='ensemble_y_090_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_090_mean_tss)

y_100_mean_tss <- weighted.mean(y_100 > eval$thrs, eval$TSS)
writeRaster(y_100_mean_tss, filename='ensemble_y_100_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_100_mean_tss)

y_110_mean_tss <- weighted.mean(y_110 > eval$thrs, eval$TSS)
writeRaster(y_110_mean_tss, filename='ensemble_y_110_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_110_mean_tss)

y_120_mean_tss <- weighted.mean(y_120 > eval$thrs, eval$TSS)
writeRaster(y_120_mean_tss, filename='ensemble_y_120_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_120_mean_tss)

y_130_mean_tss <- weighted.mean(y_130 > eval$thrs, eval$TSS)
writeRaster(y_130_mean_tss, filename='ensemble_y_130_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_130_mean_tss)

y_140_mean_tss <- weighted.mean(y_140 > eval$thrs, eval$TSS)
writeRaster(y_140_mean_tss, filename='ensemble_y_140_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_140_mean_tss)

y_150_mean_tss <- weighted.mean(y_150 > eval$thrs, eval$TSS)
writeRaster(y_150_mean_tss, filename='ensemble_y_150_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_150_mean_tss)

y_160_mean_tss <- weighted.mean(y_160 > eval$thrs, eval$TSS)
writeRaster(y_160_mean_tss, filename='ensemble_y_160_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_160_mean_tss)

y_170_mean_tss <- weighted.mean(y_170 > eval$thrs, eval$TSS)
writeRaster(y_170_mean_tss, filename='ensemble_y_170_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_170_mean_tss)

y_180_mean_tss <- weighted.mean(y_180 > eval$thrs, eval$TSS)
writeRaster(y_180_mean_tss, filename='ensemble_y_180_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_180_mean_tss)

y_190_mean_tss <- weighted.mean(y_190 > eval$thrs, eval$TSS)
writeRaster(y_190_mean_tss, filename='ensemble_y_190_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_190_mean_tss)

y_200_mean_tss <- weighted.mean(y_200 > eval$thrs, eval$TSS)
writeRaster(y_200_mean_tss, filename='ensemble_y_200_weighted_mean_Bokermannohyla_alvarengai', format = 'GTiff', overwrite = TRUE)
rm(y_200_mean_tss)

print('Bokermannohyla_alvarengai is done!')
```

# 4. Area calculation

Since we are interested in the putative role of the flickering
connectivity in the BQMs, maybe we could calculate the suitable area
size for each species in each time slice. If the flickering connectivity
is a phenomenom that has occured in the BQMs, we would see a variation
in the suitable area, mostly decreasing towards the LGM. Or a
correlation with mean temperature range.

To do so, we need the ensemble maps for each species, and for each time
slice.

``` r
library(raster)
library(ggplot2)
library(gganimate)

#read the files
files <- list.files(pattern = '*.tif')

#put the files in a rasterstack
my_stack <- stack(files)

#create an empty vector
my_areas <- c()

#set a threshold of at which level of suitability should we calculate the area
p <- 0.5

#for 21 rasters (current - LGM, by 1000 years)
for(r in 1:21){
  #select the raster
  my_raster <- my_stack[[r]]
  
  #set the values below the threshold as NA
  my_raster[my_raster < p] <- NA
  
  #create a raster with the size values
  ar <- area(my_raster, na.rm = TRUE, weights = FALSE)
  
  #sum the values
  sum_area <- sum(getValues(ar), na.rm=TRUE)
  
  #put the value in a vector
  my_areas[r] <- sum_area
  
  #remove everything and start again
  rm(my_raster)
  rm(ar)
  rm(sum_area)
}

area_df <- as.data.frame(cbind(files, my_areas))
area_df$my_areas <- as.numeric(area_df$my_areas)

my_seq <- seq(0, 20)




area_df$files <- my_seq
#area_df$files <- factor(area_df$files, levels = names_df)

write.csv(area_df, file = 'area_df.csv', row.names = F)
#area_df <- read.csv('./Bokermannohyla_alvarengai/area_df.csv')


area_plot <- ggplot(data = area_df, aes(x = files, y = my_areas)) +
  geom_bar(stat="identity") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 90, hjust = 0.95, vjust = 0.2, size = 10)) +
  theme(axis.text.y = element_text(size = 10)) +
  labs(x = 'time before present (1000 years)', y = 'area (m??)')

area_plot

ggsave(area_plot, filename = 'area_plot.png')

area_plot2 <- ggplot(data = area_df, aes(x = files, y = my_areas)) +
  geom_line() +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 90, hjust = 0.95, vjust = 0.2, size = 10)) +
  theme(axis.text.y = element_text(size = 10)) +
  labs(x = 'time before present (1000 years)', y = 'area (m??)') +
  transition_reveal(files)

animate(area_plot2, nframes = 21, fps = 10)

anim_save(animation = area_plot2, filename = 'area_plot2.gif')
```

Here are the barplots for each species:

![](/Users/marcosqueiroz/Library/CloudStorage/GoogleDrive-marvin.danque@gmail.com/My%20Drive/Doutorado/Tese_Marcos/Comparada/Analyses/Modelagem/meco/area_rasters/Bokermannohyla_oxente/area_plot_scaled.svg)

# 5. Map animations

Since we are dealing with 21 time slices, plot all of them, for all
species, can generate some confusing. Maybe animations could be more
appropriate. Let???s do some.

``` r
library(raster)
library(tmap)
library(rnaturalearth)
library(magick)

sa <- ne_countries(continent = 'south america', returnclass = 'sf')

files <- list.files(pattern = '*.tif')

my_stack <- stack(files)

my_map <- tm_shape(sa) +
  tm_polygons(alpha = 0)

my_rev_col <- c("#FAFAFA",
                "#2B83BA",
                "#ABDDA4",
                "#FFFFBF",
                "#FDAE61",
                "#D7191C")


names_df <- c('0_Current')
my_seq <- seq(1000, 20000, by = 1000)
my_seq <- sprintf('%d_years', my_seq)
names_df <- append(names_df, my_seq)

ordering_files <- matrix(nrow = 21, ncol =2)

for(i in 1:length(names_df)){
  my_plot <- my_map +
    tm_shape(my_stack[[i]]) +
    tm_raster(style = 'cont', midpoint = NA, palette = my_rev_col) +
    tm_layout(title = paste0(names_df[i]),
              title.position = c('center', 'bottom'),
              title.size = .8,
              legend.show = FALSE)
  filename <- paste0(names_df[i], '_Bokermannohyla_alvarengai.png')
  ordering_files[i,] <- c(i, filename)
  tmap_save(tm = my_plot,
            filename = filename)
}

ordering_files <- as.data.frame(ordering_files)

png_list <- lapply(ordering_files$V2, image_read)
png_joined <- image_join(png_list)
png_animated <- image_animate(png_joined, fps = 2)
#png_animated

image_write(png_animated, "Bokermannohyla_oxente_animated_ENM.gif")
```

***Bokermannohyla\_saxicola***

![](/Users/marcosqueiroz/Library/CloudStorage/GoogleDrive-marvin.danque@gmail.com/My%20Drive/Doutorado/Tese_Marcos/Comparada/Analyses/Modelagem/meco/area_rasters/Bokermannohyla_oxente/Bokermannohyla_oxente_animated_ENM.gif)
