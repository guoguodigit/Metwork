---
title: "HiTMaP"
output:
  html_document: 
    keep_md: yes
    toc: yes
    toc_float: yes
    collapsed: no
    theme: spacelab
    df_print: tibble
    highlight: zenburn
    fig_width: 10
    fig_height: 10
    number_sections: yes
  word_document: default
  pdf_document: default
  md_document:
    variant: markdown_github
bibliography: references.bib
vignette: >
  %\VignetteIndexEntry{1. HiTMaP}
  %\VignetteEncoding{UTF-8}
  %\VignetteEngine{knitr::rmarkdown}
---




-- An R package of High-resolution Informatics Toolbox for Maldi-imaging Proteomics

![](https://zenodo.org/badge/187550066.svg)

# Package installation
This is a tutorial for the use of HiTMaP (An R package of High-resolution Informatics Toolbox for Maldi-imaging Proteomics). User's may run HiTMaP using Docker, or through R, however Docker is recommended to avoid issues with package dependency.

## Installation of docker image

HiTMaP has been encapsulated into a docker image. Using a bash terminal, user's can download the latest version by using the code as below.


```bash
docker pull mashuoa/hitmap
```

Seting up and running the docker container:

```bash
# For windows user's, run the image with a local user\Documents\expdata folder mapped to the docker container:
docker run --name hitmap -v %userprofile%\Documents\expdata:/root/expdata -a stdin -a stdout -i -t mashuoa/hitmap /bin/bash 

# For linux or mac user's, run the image with a local user/expdata folder mapped to the docker container:
docker run --name hitmap -v ~/expdata:/root/expdata -a stdin -a stdout -i -t mashuoa/hitmap /bin/bash 

#Run the R console
R

```

Revoke Docker terminal:

```bash
#use ctrl+d to exit the docker container shell 

#Restart the container and connect to the shell 
docker restart hitmap
docker container exec -it hitmap /bin/bash
```

Stop/remove docker container (warning: if no local disk is mapped to "~/expdata", please backup your existing result files from the container before you remove it):

```bash
docker stop hitmap
docker rm hitmap
```



If you are using docker GUI, pull the docker image using the codes above and follow the image as below to setup the container.

![Docker GUI setting](Resource/docker_gui_setting.png)

## Installation code for R console installation

The code below is used for an experienced R user to build a local R/HiTMaP running environment. 
Major dependencies to note:

* R base
* java running library (for linux, additional configuration is needed: *R CMD javareconf*)
* orca for plotly (https://github.com/plotly/orca/releases/tag/v1.3.1)
* magick++ (for Linux, additional configuration is needed to expand the pixel limitation)



```r
#install the git package
install.packages("remotes")
install.packages("devtools")
#library(devtools)
library(remotes)
Sys.setenv("R_REMOTES_NO_ERRORS_FROM_WARNINGS" = "true")
remotes::install_github("MASHUOA/HiTMaP",auth_token ="cf6d877f8b6ada1865987b13f9e9996c1883014a",force=T)
3
no
#Update all dependencies
BiocManager::install(ask = F)
yes
library(HiTMaP)
```

For windows users, Rtools (*https://cran.r-project.org/bin/windows/Rtools/*) is required.

## Codes for Linux OS building enviornment
Run the codes as below to enable the required components in Linux console.


```bash
sudo apt-get install tcl-dev tk-dev
sudo apt-get install r-cran-ncdf4
sudo apt-get install libz-dev
sudo apt install libxml2-dev
sudo apt install libssl-dev
sudo apt install libcurl4-openssl-dev
sudo apt-get install libnss-winbind winbind
sudo apt install dirmngr gnupg apt-transport-https ca-certificates software-properties-common

sudo add-apt-repository ppa:webupd8team/y-ppa-manager
sudo apt-get update
sudo apt-get install y-ppa-manager
sudo y-ppa-manager

sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys
sudo add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu focal-cran40/'
sudo apt-cache policy r-base
sudo apt-get purge r-base
sudo apt-get install r-base-core="4.0.2-1.2004.0"
sudo apt-get install libmagick++-dev

sudo apt-get install libfftw3-dev
sudo apt-get install r-base-dev texlive-full
sudo apt-get install libudunits2-dev
sudo apt-get install libgdal-dev
```

## Codes for Mac OS building enviornment (optional)

The following code is for a local GUI purpose. Hitmap now has been built on the shiny server system. You can skip this step in the later version. You may need to update the Xcode. Go to your Mac OS terminal and input:


```bash
xcode-select --install
```

You'll then receive:
*xcode-select: note: install requested for command line developer tools*
You will be prompted at this point in a window to update Xcode Command Line tools. 

You may also need to install the X11.app and tcl/tk support for Mac system:

+ X11.app:
https://www.xquartz.org/

+ Use the following link to download and install the correct tcltk package for your OS version.
https://cran.r-project.org/bin/macosx/tools/


# Example data and source code
The HitMaP comes with a series of maldi-imaging datasets acquired by FT-ICR mass spectromety. With the following code, you can download these raw data set into a local folder.  

You can download the example data manually through this link: "https://github.com/MASHUOA/HiTMaP/releases/download/1.0/Data.tar.gz"


Or download the files in a R console:

```r
if(!require(piggyback)) install.packages("piggyback")
library(piggyback)

Sys.setenv(GITHUB_TOKEN="cf6d877f8b6ada1865987b13f9e9996c1883014a")

#made sure that this folder has enough space
wd="~/expdata/"
dir.create(wd)
setwd(wd)

pb_download("HiTMaP-master.zip", repo = "MASHUOA/HiTMaP", dest = ".",.token ="cf6d877f8b6ada1865987b13f9e9996c1883014a")

pb_download("Data.tar.gz", repo = "MASHUOA/HiTMaP", dest = ".",.token ="cf6d877f8b6ada1865987b13f9e9996c1883014a")

untar('Data.tar.gz',exdir =".",  tar="tar")

#unlink('Data.tar.gz')
list.dirs()
```
The example data contains three folders for three individual IMS datasets, which each contain a configuration file, and the fasta database, respectively:
*"./Bovinlens_Trypsin_FT" 
"./MouseBrain_Trypsin_FT"
"./Peptide_calibrants_FT"*

# Proteomics identification on maldi-imaging dataset 

To perform false-discovery rate controlled  peptide and protein annotation, run the following script below in your R session:


```r
#create candidate list
library(HiTMaP)
#set project folder that contains imzML, .ibd and fasta files
#wd=paste0(file.path(path.package(package="HiTMaP")),"/data/")
#set a series of imzML files to be processed
datafile=c("Bovinlens_Trypsin_FT/Bovin_lens.imzML")
wd="~/expdata/"


imaging_identification(
#==============Choose the imzml raw data file(s) to process  make sure the fasta file in the same folder
               datafile=paste0(wd,datafile),
               threshold=0.005, 
               ppm=5,
               FDR_cutoff = 0.05,
#==============specify the digestion enzyme specificity
               Digestion_site="trypsin",
#==============specify the range of missed Cleavages
               missedCleavages=0:1,
#==============Set the target fasta file
               Fastadatabase="uniprot-bovin.fasta",
#==============Set the possible adducts and fixed modifications
               adducts=c("M+H"),
               Modifications=list(fixed=NULL,fixmod_position=NULL,variable=NULL,varmod_position=NULL),
#==============The decoy mode: could be one of the "adducts", "elements" or "isotope"
               Decoy_mode = "isotope",
               use_previous_candidates=F,
               output_candidatelist=T,
#==============The pre-processing param
               preprocess=list(force_preprocess=TRUE,
                               use_preprocessRDS=TRUE,
                               smoothSignal=list(method="gaussian"),
                               reduceBaseline=list(method="locmin"),
                               peakPick=list(method="adaptive"),
                               peakAlign=list(tolerance=5, units="ppm"),
                               normalize=list(method=c("rms","tic","reference")[1],mz=1)),
#==============Set the parameters for image segmentation
               spectra_segments_per_file=4,
               Segmentation="spatialKMeans",
               Smooth_range=1,
               Virtual_segmentation=FALSE,
               Virtual_segmentation_rankfile=NULL,
#==============Set the Score method for hi-resolution isotopic pattern matching
               score_method="SQRTP",
               peptide_ID_filter=2,
#==============Summarise the protein and peptide features across the project the result can be found at the summary folder
               Protein_feature_summary=TRUE,
               Peptide_feature_summary=TRUE,
               Region_feature_summary=TRUE,
#==============The parameters for Cluster imaging. Specify the annotations of interest, the program will perform a case-insensitive search on the result file, extract the protein(s) of interest and plot them in the cluster imaging mode
               plot_cluster_image_grid=FALSE,
               ClusterID_colname="Protein",
               componentID_colname="Peptide",
               Protein_desc_of_interest=c("Crystallin","Actin"),
               Rotate_IMG=NULL,
               )
```


# Project folder and result structure 

In the above function, you have performed proteomics analysis on the sample data file. It is a tryptic Bovin lens MALDI-imaging file which is acquired on an FT-ICR MS.
The function will take the selected data files' root directory as the project folder.
In this example, the project folder will be:


```r
library(HiTMaP)
wd=paste0("D:\\GITHUB LFS\\HiTMaP-Data\\inst","/data/Bovinlens_Trypsin_FT/")
datafile=c("Bovin_lens")
```


After the whole identification process, you will get two sub-folders within the project folder:


```r
list.dirs(wd, recursive=FALSE)
```

```
## [1] "D:\\GITHUB LFS\\HiTMaP-Data\\inst/data/Bovinlens_Trypsin_FT//Bovin_lens ID" 
## [2] "D:\\GITHUB LFS\\HiTMaP-Data\\inst/data/Bovinlens_Trypsin_FT//Summary folder"
```

1. The one which has an identical name to an input data file contains the identification result of that input:
   + the protein and peptides list of each segmentation region
   + the PMF matching plot of each segmentation
   + the image that indicates segmentations' boundary (applies to either K-mean segmentation (powered by Cardinal) or manually defined segmentation)
   + folders of each region contains the detailed identification process, FDR plots and isotopic pattern matching plots

2. "Summary folder" contains: 
   + the identification summary of protein and peptides across all the data
   + the candidate list of all possible proteins and peptides (if *use_previous_candidates* is set as **TRUE**)
   + the Cluster imaging files of the protein of interest
   + the database stats result for resolution-based candidates binning (optional)
   
   
# Identification result visulasation and interpretation

To plot the MALDI-image peptide and protein images, use the following functions:

To check the segmentation result over the sample, you need to d to each data file ID folder and find the "spatialKMeans_image_plot.png" (if you are using the spatial K-means method for segmentation.)


```r
library(magick)
```

```
## Linking to ImageMagick 6.9.9.14
## Enabled features: cairo, freetype, fftw, ghostscript, lcms, pango, rsvg, webp
## Disabled features: fontconfig, x11
```

```r
p<-image_read(paste0(wd,datafile," ID/spatialKMeans_image_plot.png"))
print(p)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG     1024   2640 sRGB       FALSE    30726 72x72
```

<img src="README_files/figure-html/VisulazeKmean-1.png" width="1024" />

The pixels in image data now has been categorized into four regions according to the initial setting of segmentation (*spectra_segments_per_file=5*). The rainbow shaped bovine lens segmentation image (on the left panel) shows a unique statistical classification based on the mz features of each region (on the right panel).

The identification will take place on the **mean spectra** of each region. To check the peptide mass fingerprint (PMF) matching quality, 
you could locate the PMF spectrum matching plot of each individual region.


```r
library(magick)
p_pmf<-image_read(paste0(wd,datafile," ID/Bovin_lens 3PMF spectrum match.png"))
print(p_pmf)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG     1980   1080 sRGB       FALSE    17664 72x72
```

<img src="README_files/figure-html/unnamed-chunk-8-1.png" width="1980" />

A list of the peptides and proteins annotated within each region has also been created for manual exploration of the results.


```r
peptide_pmf_result<-read.csv(paste0(wd,datafile," ID/Peptide_segment_PMF_RESULT_3.csv"))
head(peptide_pmf_result)
```

```
## # A tibble: 6 x 23
##   Protein    mz Protein_coverage isdecoy Peptide    Modification pepmz formula  
##     <int> <dbl>            <dbl>   <int> <chr>      <lgl>        <dbl> <chr>    
## 1      48 1301.           0.0688       0 HLEQFATEG~ NA           1300. C57H90N1~
## 2      48 1301.           0.0688       0 QYFLDLALS~ NA           1300. C60H94N1~
## 3      48 1325.           0.0688       0 GSKCILYCF~ NA           1324. C62H94N1~
## 4      53 1329.           0.0554       0 FKNINPFPV~ NA           1328. C64H98N1~
## 5      53 1450.           0.0554       0 AVQNFTEYN~ NA           1449. C65H97N1~
## 6      53 1606.           0.0554       0 AVQNFTEYN~ NA           1605. C71H109N~
## # ... with 15 more variables: adduct <chr>, charge <int>, start <int>,
## #   end <int>, pro_end <int>, mz_align <dbl>, Score <dbl>, Rank <int>,
## #   moleculeNames <chr>, Region <int>, Delta_ppm <dbl>, Intensity <dbl>,
## #   peptide_count <int>, desc.x <chr>, desc.y <chr>
```


```r
protein_pmf_result<-read.csv(paste0(wd,datafile," ID/Protein_segment_PMF_RESULT_3.csv"))
head(protein_pmf_result)
```

```
## # A tibble: 6 x 9
##   Protein Proscore isdecoy Intensity Score peptide_count Protein_coverage
##     <int>    <dbl>   <int>     <dbl> <dbl>         <int>            <dbl>
## 1   10134   0.139        0  2873903. 1.93              3           0.0672
## 2   10204   0.137        0   380571. 0.794             3           0.185 
## 3   10370   0.204        0  1877250. 2.08              4           0.0936
## 4   10659   0.112        0   327352. 0.745             3           0.164 
## 5   10888   0.0798       0   532832. 1.24              3           0.0672
## 6   11270   0.107        0  2944154. 1.33              3           0.0745
## # ... with 2 more variables: Intensity_norm <dbl>, desc <chr>
```

# Scoring system for protein and peptide
**Score** in peptide result table shows the isotopic pattern matching score of the peptide (Pepscore). In Protein result table, it shows the protein score (Proscore). The 'Pepscore' consist of two parts: Intensity_Score and Mass_error_Score:

   + Intensity_Score indicates how well a putative isotopic pattern can be matched to the observed spectrum.The default scoring method is SQRTP. It combines the 'square root mean' differences between observed and theoretical peaks and observed proportion of the isotopic peaks above a certain relative intensity threshold.
   
   + Mass_error_Score indicates the summary of mass error (in *ppm*) for every detected isotopic peak. In order to integrate the Mass_error_Score in to scoring system, the mean ppm error has been normalized by ppm tolerance, and supplied to the probability normal distributions (*pnorm* function for R). The resulting value (quantiles of the given probability density) is deducted by 0.5 and converted into an absolute value.

$Intensity\_Score=\log(PeakCount_{Observed}/PeakCount_{Theoritical})-\log(\sqrt{\frac{\sum_{x = 1}^{n} (Theoritical\_intensity_x-Observed\_intensity_x)^2}{\sum_{x = 1}^{n} (Theoritical\_intensity_x)^2(Observed\_intensity_x)^2}}$

$Mass\_error\_Score=|(p\_norm\_dist(\frac{mean\_ppm\_error}{ppm\_tolerance})-0.5)|$


$Pepscore=Intensity\_Score-Mass\_error\_Score$

**Proscore** in the protein result table shows the overall estimation of the protein identification Accuracy

$Proscore=\frac{\sum_{x = 1}^{n}(Pepscore_x*log(Intensity_x))}{mean(log(Intensity))}*Protein\_coverage*Normalized\_intensity$

A *Peptide_region_file.csv* has also been created to summarise all the IDs in this data file:


```r
Identification_summary_table<-read.csv(paste0(wd,datafile," ID/Peptide_region_file.csv"))
head(Identification_summary_table)
```

```
## # A tibble: 6 x 23
##   Protein    mz Protein_coverage isdecoy Peptide     Modification pepmz formula 
##     <int> <dbl>            <dbl>   <int> <chr>       <lgl>        <dbl> <chr>   
## 1      24 1144.           0.0612       0 GFPGQDGLAG~ NA           1143. C51H79N~
## 2      24 1685.           0.0612       0 DGANGIPGPI~ NA           1684. C72H118~
## 3      24  742.           0.0612       0 GDSGPPGR    NA            741. C29H48N~
## 4      24 1694.           0.0612       0 LLSTEGSQNI~ NA           1693. C72H117~
## 5      24 1882.           0.0612       0 GQPGVMGFPG~ NA           1881. C82H129~
## 6      48 1217.           0.0348       0 ASTSVQNRLLK NA           1216. C51H94N~
## # ... with 15 more variables: adduct <chr>, charge <int>, start <int>,
## #   end <int>, pro_end <int>, mz_align <dbl>, Score <dbl>, Rank <int>,
## #   moleculeNames <chr>, Region <int>, Delta_ppm <dbl>, Intensity <dbl>,
## #   peptide_count <int>, desc.x <chr>, desc.y <chr>
```

The details of protein/peptide identification process has been save to the folder named by the segmentation:


```r
list.dirs(paste0(wd,datafile," ID/"), recursive=FALSE)
```

```
## [1] "D:\\GITHUB LFS\\HiTMaP-Data\\inst/data/Bovinlens_Trypsin_FT/Bovin_lens ID//1"
## [2] "D:\\GITHUB LFS\\HiTMaP-Data\\inst/data/Bovinlens_Trypsin_FT/Bovin_lens ID//2"
## [3] "D:\\GITHUB LFS\\HiTMaP-Data\\inst/data/Bovinlens_Trypsin_FT/Bovin_lens ID//3"
## [4] "D:\\GITHUB LFS\\HiTMaP-Data\\inst/data/Bovinlens_Trypsin_FT/Bovin_lens ID//4"
```
In the identification details folder, you will find a series of FDR file and plots to demonstrate the FDR model and score cutoff threshold:


```r
dir(paste0(wd,datafile," ID/1/"), recursive=FALSE)
```

```
##  [1] "FDR.CSV"                                        
##  [2] "FDR.png"                                        
##  [3] "Matching_Score_vs_mz_target-decoy.png"          
##  [4] "Peptide_1st_ID.csv"                             
##  [5] "Peptide_1st_ID_score_rank_SQRTP.csv"            
##  [6] "Peptide_2nd_ID_score_rankSQRTP_Rank_above_3.csv"
##  [7] "Peptide_Score_histogram_target-decoy.png"       
##  [8] "ppm"                                            
##  [9] "PROTEIN_FDR.CSV"                                
## [10] "Protein_FDR.png"                                
## [11] "Protein_ID_score_rank_SQRTP.csv"                
## [12] "PROTEIN_Score_histogram.png"                    
## [13] "Spectrum.csv"                                   
## [14] "unique_peptide_ranking_vs_mz_feature.png"
```

In this folder, you will find the FDR plots for protein and peptide annotation. The software will take the proscore and its FDR model to trim the final identification results. The *unique_peptide_ranking_vs_mz_feature.png* is a plot that could tell you the number of peptide candidates that have been matched to the mz features in the first round run. You can also access the peptide spectrum match (first MS dimension) data via the "/ppm" subfolder.


```r
library(magick)
p_FDR_peptide<-image_read(paste0(wd,datafile," ID/3/FDR.png"))
p_FDR_protein<-image_read(paste0(wd,datafile," ID/3/protein_FDR.png"))
p_FDR_peptide_his<-image_read(paste0(wd,datafile," ID/3/Peptide_Score_histogram_target-decoy.png"))
p_FDR_protein_his<-image_read(paste0(wd,datafile," ID/3/PROTEIN_Score_histogram.png"))
p_combined<-image_append(c(p_FDR_peptide,p_FDR_peptide_his,p_FDR_protein,p_FDR_protein_his))
print(p_combined)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG     1920    480 sRGB       FALSE        0 72x72
```

<img src="README_files/figure-html/FDR plot-1.png" width="1920" />

You will also find a *Matching_Score_vs_mz* plot for further investigation on peptide matching quality. 


```r
library(magick)
#plot Matching_Score_vs_mz
p_Matching_Score_vs_mz<-image_read(paste0(wd,datafile," ID/3/Matching_Score_vs_mz_target-decoy.png"))
print(p_Matching_Score_vs_mz)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG      480    480 sRGB       FALSE    47438 72x72
```

<img src="README_files/figure-html/p_Matching_Score_vs_mz plot-1.png" width="480" />

# Identification summary and cluster imaging

In the project summary folder, you will find four files and a sub-folder:

```r
wd_sum=paste(wd,"/Summary folder", sep="")
dir(wd_sum)
```

```
## [1] "candidatelist.csv"                "cluster Ion images"              
## [3] "Peptide_Summary.csv"              "Protein_feature_list_trimmed.csv"
## [5] "protein_index.csv"                "Protein_Summary.csv"
```

"candidatelist.csv" and "protein_index.csv" contains the candidates used for this analysis. They are output after the candidate processing while *output_candidatelist* set as TRUE, and can be used repeatedly while *use_previous_candidates* set as TRUE.

We have  implemented a functionality to perform additional statistical analyses around the number of enzymatically generated peptides generated derived from a given proteome (‘Database_stats’). If the user sets the variable ‘Database_stats’ to TRUE in the main workflow, then the function will be called. Briefly, the function will list all of the m/z’s of a unique formulae from the peptide candidate pool within a given m/z range. The m/z’s will then be binned using three resolution m/z windows: 1ppm, 2ppm and 5ppm. A plot showing the number of unique formulae vs. binned m/z windows will be generated and exported to the summary folder (DB_stats_mz_bin).


![Proteome database stats](Resource/DB_stats_bin_mz_ppm.png)


"Peptide_Summary.csv" and "Protein_Summary.csv" contains the table of the project identification summary. You could set the *plot_cluster_image_grid* as TRUE to enable the cluster imaging function. Please be noted that you could indicate *Rotate_IMG* with a CSV file path that indicates the rotation degree of image files. 

**Note**: 90$^\circ$, 180$^\circ$ and 270$^\circ$ are recommended for image rotation. You may find an example CSV file in the library/HiTMaP/data folder.


```r
library(dplyr)
Protein_desc_of_interest<-c("Crystallin","Actin")
Protein_Summary_tb<-read.csv(paste(wd,"/Summary folder","/Protein_Summary.csv", sep=""),stringsAsFactors = F)
```


Finally, you are able visualize the annotated proteins and their associated peptide distributions via the cluster imaging function.


```r
library(magick)
```

```
## Linking to ImageMagick 6.9.11.57
## Enabled features: cairo, freetype, fftw, ghostscript, heic, lcms, pango, raw, rsvg, webp
## Disabled features: fontconfig, x11
```

```r
p_cluster2<-image_read(paste0("~/expdata/Bovinlens_Trypsin_FT/Summary folder/cluster Ion images/unique/25917_cluster_imaging.png"))
print(p_cluster2)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG     5670   1951 sRGB       FALSE   678267 59x59
```

<img src="README_files/figure-html/CLuster imaging-1.png" width="5670" />

```r
p_cluster4<-image_read(paste0("~/expdata/Bovinlens_Trypsin_FT/Summary folder/cluster Ion images/unique/452_cluster_imaging.png"))
print(p_cluster4)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG    25110   2063 sRGB       FALSE  3010819 59x59
```

<img src="README_files/figure-html/CLuster imaging-2.png" width="25110" />

```r
p_cluster1<-image_read(paste0("~/expdata/Bovinlens_Trypsin_FT/Summary folder/cluster Ion images/unique/791_cluster_imaging.png"))
print(p_cluster1)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG     8910   2020 sRGB       FALSE  1121766 59x59
```

<img src="README_files/figure-html/CLuster imaging-3.png" width="8910" />

```r
p_cluster3<-image_read(paste0("~/expdata/Bovinlens_Trypsin_FT/Summary folder/cluster Ion images/unique/5479_cluster_imaging.png"))
print(p_cluster3)
```

```
## # A tibble: 1 x 7
##   format width height colorspace matte filesize density
##   <chr>  <int>  <int> <chr>      <lgl>    <int> <chr>  
## 1 PNG    13770   1751 sRGB       FALSE  1118974 59x59
```

<img src="README_files/figure-html/CLuster imaging-4.png" width="13770" />

# Details of parameter setting


## Modification
You can choose one or a list of modifications from the unimod modification list. *Peptide_modification* function is used to load/rebuild the modification database into the global enviornment of R. It will be called automatically in the identification work flow. you can use the *code_name* or *record_id* to refer the modification (see example data "peptide calibrants" to find more details). The pipeline will select the *non-hidden* modifications.




```r
HiTMaP:::Peptide_modification(retrive_ID=NULL,update_unimod=F)
modification_list<-merge(unimod.df$modifications,unimod.df$specificity,by.x=c("record_id"),by.y=c("mod_key"),all.x=T)
head(modification_list['&'(modification_list$code_name=="Phospho",modification_list$hidden!=1),c("code_name","record_id","composition","mono_mass","position_key","one_letter")])
```

```
## # A tibble: 3 x 6
##   code_name record_id composition mono_mass position_key one_letter
##   <chr>     <chr>     <chr>       <chr>     <chr>        <chr>     
## 1 Phospho   21        H O(3) P    79.966331 2            Y         
## 2 Phospho   21        H O(3) P    79.966331 2            T         
## 3 Phospho   21        H O(3) P    79.966331 2            S
```

```r
head(modification_list['&'(modification_list$code_name=="Amide",modification_list$hidden!=1),c("code_name","record_id","composition","mono_mass","position_key","one_letter")])
```

```
## # A tibble: 2 x 6
##   code_name record_id composition mono_mass position_key one_letter
##   <chr>     <chr>     <chr>       <chr>     <chr>        <chr>     
## 1 Amide     2         H N O(-1)   -0.984016 6            C-term    
## 2 Amide     2         H N O(-1)   -0.984016 4            C-term
```

```r
head(modification_list['&'(stringr::str_detect(modification_list$code_name,"Ca"),modification_list$hidden!=1),c("code_name","record_id","composition","mono_mass","position_key","one_letter")])
```

```
## # A tibble: 5 x 6
##   code_name       record_id composition    mono_mass position_key one_letter
##   <chr>           <chr>     <chr>          <chr>     <chr>        <chr>     
## 1 Carbamidomethyl 4         H(3) C(2) N O  57.021464 2            C         
## 2 Carbamidomethyl 4         H(3) C(2) N O  57.021464 3            N-term    
## 3 Carbamyl        5         H C N O        43.005814 3            N-term    
## 4 Carbamyl        5         H C N O        43.005814 2            K         
## 5 Carboxymethyl   6         H(2) C(2) O(2) 58.005479 2            C
```

If a modification occurs on a particular site, you will also need to specify the position of a modifications.

* *Anywhere*, side chain of possible amino acids
* *Any N-term*, any N-term of enzymatic peptide 
* *Protein N-term*, any N-term of protein 
 

```r
unimod.df[["positions"]]
```

```
## # A tibble: 6 x 2
##   position       record_id
##   <chr>          <chr>    
## 1 -              1        
## 2 Anywhere       2        
## 3 Any N-term     3        
## 4 Any C-term     4        
## 5 Protein N-term 5        
## 6 Protein C-term 6
```
## Amino acid substitution
You can set the *Substitute_AA* to make the uncommon amino acid available to the workflow:
*Substitute_AA=list(AA=c("X"),AA_new_formula=c("C5H5NO2"),Formula_with_water=c(FALSE))*

* AA: the single letter amino acid to be replaced
* AA_new_formula: the new formula for the amino acid
* Formula_with_water: Set *TRUE* to indicate the formula represents the intact amino acid, *FALSE* to indicate that the formula already lost one H2O molecule and can be considered as AA backbone.


## Digestion site and enzyme

The *Digestion_site* allows you to specify a list of pre-defined enzyme and customized digestion rules in regular expression format. You can either use the enzyme name, customized cleavage rule or combination of them to get the enzymatics peptides list. 





```r
Cleavage_rules<-Cleavage_rules_fun()
Cleavage_df<-data.frame(Enzyme=names(Cleavage_rules),Cleavage_rules=unname(Cleavage_rules),stringsAsFactors = F)
library(gridExtra)
grid.ftable(Cleavage_df, gp = gpar(fontsize=9,fill = rep(c("grey90", "grey95"))))
```

![](README_files/figure-html/unnamed-chunk-15-1.png)<!-- -->




# Example workflow command

Below is a list of commands including the parameters for the example data sets.

## Peptide calibrant


```r
#peptide calibrant
library(HiTMaP)
datafile=c("Peptide_calibrants_FT/trypsin_non-decell_w.calibrant_FTICR")
wd="~/expdata/"

# Calibrants dataset analysis with modification
imaging_identification(datafile=paste0(wd,datafile),
  Digestion_site="trypsin",
  Fastadatabase="uniprot_cali.fasta",
  output_candidatelist=T,
  plot_matching_score=T,
  spectra_segments_per_file=1,
  use_previous_candidates=F,
  peptide_ID_filter=1,ppm=5,missedCleavages=0:5,
  Modifications=list(fixed=NULL,fixmod_position=NULL,variable=c("Amide"),varmod_position=c(6)),
  FDR_cutoff=0.1,
  Substitute_AA=list(AA=c("X"),AA_new_formula=c("C5H5NO2"),Formula_with_water=c(FALSE)))

# Calibrants dataset analysis with no modification
imaging_identification(datafile=paste0(wd,datafile),
  Digestion_site="trypsin",
  Fastadatabase="uniprot_cali.fasta",
  output_candidatelist=T,
  plot_matching_score=T,
  spectra_segments_per_file=1,
  use_previous_candidates=T,
  peptide_ID_filter=1,ppm=5,missedCleavages=0:5,
  FDR_cutoff=0.1)

library(HiTMaP)
datafile=c("Peptide_calibrants_FT/trypsin_non-decell_w.calibrant_FTICR")
wd="~/expdata/"
# Calibrants dataset analysis with modification 
imaging_identification(datafile=paste0(wd,datafile),
  Digestion_site="trypsin",
  Fastadatabase="calibrants.fasta",
  output_candidatelist=T,
  plot_matching_score=T,
  spectra_segments_per_file=1,
  use_previous_candidates=T,
  peptide_ID_filter=1,ppm=5,missedCleavages=0:5,
  Modifications=list(fixed=NULL,fixmod_position=NULL,variable=c("Amide"),varmod_position=c(6)),
  FDR_cutoff=0.1,
  Substitute_AA=list(AA=c("X"),AA_new_formula=c("C5H5NO2"),Formula_with_water=c(FALSE)),Thread = 1)
```

## Bovine lens

```r
library(HiTMaP)
datafile=c("Bovinlens_Trypsin_FT/Bovin_lens.imzML")
wd="~/expdata/"

# Data pre-processing and proteomics annotation
library(HiTMaP)
imaging_identification(datafile=paste0(wd,datafile),Digestion_site="trypsin",
                       Fastadatabase="uniprot-bovin.fasta",output_candidatelist=T,
                       preprocess=list(force_preprocess=TRUE,
                               use_preprocessRDS=TRUE,
                               smoothSignal=list(method="gaussian"),
                               reduceBaseline=list(method="locmin"),
                               peakPick=list(method="adaptive"),
                               peakAlign=list(tolerance=5, units="ppm"),
                               normalize=list(method=c("rms","tic","reference")[1],mz=1)),
                       spectra_segments_per_file=4,use_previous_candidates=F,ppm=5,FDR_cutoff = 0.05,IMS_analysis=T,
                       Rotate_IMG="file_rotationbk.csv",plot_cluster_image_grid=F)

datafile=c("Bovinlens_Trypsin_FT/Bovin_lens.imzML")
wd="~/expdata/"
library(HiTMaP)
imaging_identification(datafile=paste0(wd,datafile),Digestion_site="trypsin",
                       Fastadatabase="uniprot-bovin.fasta",output_candidatelist=T,use_previous_candidates=T,
                       preprocess=list(force_preprocess=TRUE,
                               use_preprocessRDS=TRUE,
                               smoothSignal=list(method="gaussian"),
                               reduceBaseline=list(method="locmin"),
                               peakPick=list(method="adaptive"),
                               peakAlign=list(tolerance=5, units="ppm"),
                               normalize=list(method=c("rms","tic","reference")[1],mz=1)),
                       spectra_segments_per_file=4,ppm=5,FDR_cutoff = 0.05,IMS_analysis=T,
                       Rotate_IMG="file_rotationbk.csv",plot_cluster_image_grid=F)

# Re-analysis and cluster image rendering

library(HiTMaP)
datafile=c("Bovinlens_Trypsin_FT/Bovin_lens.imzML")
wd="~/expdata/"
imaging_identification(datafile=paste0(wd,datafile),Digestion_site="trypsin",
                       Fastadatabase="uniprot-bovin.fasta",
                       use_previous_candidates=T,ppm=5,IMS_analysis=F,
                       plot_cluster_image_grid=T,
                       export_Header_table=T, 
                       img_brightness=250, 
                       plot_cluster_image_overwrite=T,
                       cluster_rds_path = "/Bovin_lens ID/preprocessed_imdata.RDS",pixel_size_um = 150,
                       Plot_score_abs_cutoff=-0.1,
                       remove_score_outlier=T,
                       Protein_desc_of_interest=c("Crystallin","Phakinin","Filensin","Actin","Vimentin","Cortactin","Visinin","Arpin","Tropomyosin","Myosin Light Chain 3","Kinesin Family Member 14","Dynein Regulatory Complex","Ankyrin Repeat Domain 45"))
```


## Mouse brain

```r
library(HiTMaP)
datafile=c("MouseBrain_Trypsin_FT/Mouse_brain.imzML")
wd="~/expdata/"

# Data pre-processing and proteomics annotation
library(HiTMaP)
imaging_identification(datafile=paste0(wd,datafile),Digestion_site="trypsin",
                       Fastadatabase="uniprot_mouse_20210107.fasta",output_candidatelist=T,
                       preprocess=list(force_preprocess=T,
                               use_preprocessRDS=TRUE,
                               smoothSignal=list(method="gaussian"),
                               reduceBaseline=list(method="locmin"),
                               peakPick=list(method="adaptive"),
                               peakAlign=list(tolerance=5, units="ppm"),
                               normalize=list(method=c("rms","tic","reference")[1],mz=1)),
                       spectra_segments_per_file=9,use_previous_candidates=F,ppm=10,FDR_cutoff = 0.05,IMS_analysis=T,
                       Rotate_IMG="file_rotationbk.csv",
                       mzrange = c(500,4000),plot_cluster_image_grid=F)

# Re-analysis and cluster image rendering
library(HiTMaP)
datafile=c("MouseBrain_Trypsin_FT/Mouse_brain.imzML")
wd="~/expdata/"
imaging_identification(datafile=paste0(wd,datafile),Digestion_site="trypsin",
                       Fastadatabase="uniprot_mouse_20210107.fasta",
                       preprocess=list(force_preprocess=FALSE),
                       spectra_segments_per_file=9,use_previous_candidates=T,ppm=10,FDR_cutoff = 0.05,IMS_analysis=F,
                       mzrange = c(500,4000),plot_cluster_image_grid=T,
                       img_brightness=250, plot_cluster_image_overwrite=T,
                       cluster_rds_path = "/Mouse_brain ID/preprocessed_imdata.RDS",
                       pixel_size_um = 50,
                       Plot_score_abs_cutoff=-0.1,
                       remove_score_outlier=T,
                       Protein_desc_of_interest=c("Secernin","GN=MBP","Cytochrome"))
```








# Session information


```r
toLatex(sessionInfo())
```

```
## \begin{itemize}\raggedright
##   \item R version 4.0.4 (2021-02-15), \verb|x86_64-w64-mingw32|
##   \item Locale: \verb|LC_COLLATE=English_Australia.1252|, \verb|LC_CTYPE=English_Australia.1252|, \verb|LC_MONETARY=English_Australia.1252|, \verb|LC_NUMERIC=C|, \verb|LC_TIME=English_Australia.1252|
##   \item Running under: \verb|Windows 10 x64 (build 19042)|
##   \item Matrix products: default
##   \item Base packages: base, datasets, graphics, grDevices, grid,
##     methods, stats, utils
##   \item Other packages: data.table~1.14.0, dplyr~1.0.5, gridExtra~2.3,
##     HiTMaP~1.0.0, lattice~0.20-41, magick~2.7.0, pls~2.7-3,
##     protViz~0.6.8, XML~3.99-0.5
##   \item Loaded via a namespace (and not attached): assertthat~0.2.1,
##     bslib~0.2.4, cli~2.3.1, codetools~0.2-18, compiler~4.0.2,
##     crayon~1.4.1, DBI~1.1.1, debugme~1.1.0, digest~0.6.27,
##     ellipsis~0.3.1, evaluate~0.14, fansi~0.4.2, fastmap~1.1.0,
##     generics~0.1.0, glue~1.4.2, gtable~0.3.0, highr~0.8,
##     htmltools~0.5.1.1, httpuv~1.5.5, jquerylib~0.1.3, jsonlite~1.7.2,
##     knitr~1.31, later~1.1.0.1, lifecycle~1.0.0, magrittr~2.0.1,
##     mime~0.10, pillar~1.5.1, pkgconfig~2.0.3, png~0.1-7,
##     promises~1.2.0.1, purrr~0.3.4, R6~2.5.0, Rcpp~1.0.6, rlang~0.4.10,
##     rmarkdown~2.7, rstudioapi~0.13, sass~0.3.1, shiny~1.6.0,
##     stringi~1.5.3, stringr~1.4.0, tibble~3.1.0, tidyselect~1.1.0,
##     tools~4.0.2, utf8~1.2.1, vctrs~0.3.6, xfun~0.22, xtable~1.8-4,
##     yaml~2.2.1
## \end{itemize}
```



End of the tutorial, Enjoy~


# References
R Packages used in this project:

   + viridisLite[@viridisLite]

   + rcdklibs[@rcdklibs]

   + rJava[@rJava]

   + data.table[@data.table]

   + RColorBrewer[@RColorBrewer]

   + magick[@magick]

   + ggplot2[@ggplot2]

   + dplyr[@dplyr]

   + stringr[@stringr]

   + protViz[@protViz]

   + cleaver[@cleaver]

   + Biostrings[@Biostrings]

   + IRanges[@IRanges]

   + Cardinal[@Cardinal]

   + tcltk[@tcltk]

   + BiocParallel[@BiocParallel]

   + spdep[@spdep1]

   + FTICRMS[@FTICRMS]

   + UniProt.ws[@UniProt.ws]
