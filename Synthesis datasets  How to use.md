# Synthesis datasets : How to use?

The aim of this manual is to help researchers using the synthesis datasets of the Biodiversity Exploratories. It gives tips on how to handle the datasets and outlines the common issues when analysing the data.



We strongly recommend reading the metadata of the original datasets and contact their authors to understand their limitations. The Bexis identifier (DataID) and version (Dataversion) of each dataset are provided in the synthesis dataset for reproducibility.



For any suggestion on this manual or on the synthesis datasets please contact us by email or open an issue on [GitHub](https://github.com/biodiversity-exploratories-synthesis/Synthesis-dataset-manual/issues). 



## Organisation

### Diversity datasets

The diversity synthesis datasets are organised in two separate files:

1. Assembled RAW diversity (~350Mb): contain information on plots, species, value, type (either abundance, presence-absence, OTU_number, ASV_number or cover), year of measurement, Bexis DataID and data version.

   Currently these files are:

   - Grasslands :  [27707 “Assembled RAW diversity from grassland EPs (2008-2020) for multidiversity synthesis - November 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27707)
   - Forests : [24607 “Assembled RAW diversity from forest EPs (2007-2015) for multidiversity synthesis”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=24607)

2. Assembled species information (~20Mb): contain information on Trophic levels, functional and taxonomic grouping

   These two files can be merged using the common column “Species”.

   Currently these files are:

   - Grasslands : [27706 “Assembled species information from grassland EPs (2008-2020) for multidiversity synthesis - November 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27706)
   - Forests : [24608 “Assembled species information from forest EPs (2007-2015) for multidiversity synthesis”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=24608)

### Function datasets

The functions synthesis datasets are organised in three separate files:

1. Assembled RAW functions: contains entries for each function for each year. As not every function has been measured every year, many entries are missing (these are the unmeasured year-function combinations). The metadata of this file is minimal and is thought to be used with the separately stored metadata (see point 3 below).

   Currently this dataset is: [27087 “Assembled ecosystem measures from grassland EPs (2008-2018) for multifunctionality synthesis - June 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27087)

2. A script: this is a script to reformat the previous file to the wide format (plots x functions) that is easily usable for analysis. It also removes the unmeasured year-function combinations. 

   Currently this script is: [27626 “R Script to Reformat Dataset 27087 "Assembled ecosystem measures from grassland EPs" to easily usable wide format”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27626)

3. An additional metadata file : contains a table with detailed metadata for all functions.

   Currently this file is: [27088 “Additional metadata of dataset 27087: Assembled ecosystem measures from grassland EPs (2008-2018) for multifunctionality synthesis - June 2020”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=27088)

Note that the forest functions dataset has not been updated to this format yet and is stored in a Plot X Functions format. Currently this dataset is: [24367 “Raw data of forest attributes of forest EPs of the Exploratories project used in "Multiple forest attributes underpin the supply of multiple ecosystem services"](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=24367).



## Datasets size and memory issues

The diversity and function synthesis datasets can be very large and this might create memory problems. We recommend the use of the package `data.table` in `R` to handle the dataset.

```R
#Example code to read the raw diversity and species information files and merge:
library(data.table)
grl <- fread("~/27707.txt")
tr<-fread("~/27706.txt")
grassland.dataset<-merge(grl,tr,by="Species")
```



## Wide and long format

The synthesis datasets are stored in the long format; however, most analyses are done in the wide format (plots x species, plots x groups, plots x functions), see Fig.1. To transform from long to wide format you can e.g. use the `dcast` (`reshape2`), `dcast.data.table` (`data.table`) or `pivot_wider` (`tidyr`) functions in `R`. Be careful, some functions insert NAs (or other values) when creating a wide-format table and this should be carefully checked. Some functions allow choosing the values to fill (e.g. argument `“fill”` in `dcast.data.table`).

```R
#Example code to reshape to the wide format and fill with zeros:

library(data.table)

dat.wide<-dcast.data.table(dat,Species~Plot,value.var="value",fill=0)
```

![img](https://github.com/biodiversity-exploratories-synthesis/Synthesis-dataset-manual/blob/main/images/Synthesis%20datasets%20%20How%20to%20use-Fig1?raw=true)

**Fig1**: *illustration of the long format (mostly used to store and assemble data from different sources) and wide format (mostly used to type, visualise data or for data analysis)*



## Grouping (e.g. into trophic level)

### Diversity datasets

Not all groups were measured in all plots. When grouping several taxonomic groups into a given trophic level (or any other grouping, e.g. grouping multiple years), we recommend to always check if all groups/species were measured in the same plots and only use the plots where all groups/species were measured. Otherwise the calculated richness or other diversity estimates per group will be biased. We also recommend checking again the number of available plots before analysing data. The number of available plots can be found in the metadata of each dataset or can be retrieved from the synthesis dataset using the DataID column.

```R
#Example code to check which datasets have information on less than 150 plots, using the Bexis dataset ID:
dat<-dat[!is.na(value)] #remove all NAs
for (i in sort(unique(dat$DataID))){
tt<-dat[DataID==i]
if(length(unique(tt$Plot))!=150)

print(paste(i,":",length(unique(tt$Plot)),unique(tt$Trophic_level),",",unique(tt$Group_broad)))
}

#Another example using Trophic level and checking the number of plots X species combinations:
for (i in unique(dat$Trophic_level)){
tt<-dat[Group_broad==i]
if(length(unique(tt$Plot))*length(unique(tt$Species))*length(unique(tt$Year))!=nrow(tt))

print(paste(i,":",length(unique(tt$Plot))*length(unique(tt$Species)),"/",nrow(tt)))
}

```

Not all groups were measured with the same methods. When grouping taxa that have several hundreds of species (e.g. OTU data) with taxa with very few species, the information about taxa with lower richness will likely be “hidden” by the other one. When species numbers in different groups are of similar range, grouping to calculate richness is mostly fine. Grouping to calculate abundances (e.g. total number of individuals in a given plot) can be more sensitive to the methods of collection and should be done very carefully.



### Functions datasets

This also applies to the functions datasets when combining multiple functions. Additionally, it is worth mentioning that besides raw data, some functions are already assembled from several years and some functions are combined. This is indicated as “assembled data” in the column “DataID” of the metadata table (see 3. of the [Organisation](#Organisation) paragraph). The method of assembling (e.g. mean over years) is described in the “calculation” column.



## "Missing" zeros

To avoid increasing the size of the synthesis dataset, the two largest datasets (bacteria and soil fungi) do not include all combinations of plots x species when a given species was not found in a given plot i.e. they do not contain zeros (see Fig.2). These “missing” combinations are true zeros and should not be confused with NAs. The information in the soil fungi datasets is complete (i.e. all combinations of plots x species were measured, so if the value is not in the dataset it should be replaced by a zero). In grasslands, the bacteria dataset is not complete, two plots miss information for all species (AEG33 and AEG34), for these two plots, the values for all species should be replaced by NAs and any information missing in the rest of the plots are true zeros (and should be replaced by zeros). *We will provide information on missing plots for forest after the release of the updated forest dataset.*



![img](https://github.com/biodiversity-exploratories-synthesis/Synthesis-dataset-manual/blob/main/images/Synthesis%20datasets%20%20How%20to%20use-Fig2?raw=true)

**Fig2**: *example illustration of the values that are not present in the bacteria and soil fungi datasets due to dataset size limitations. The red values are the ones that are not reported in the synthesis dataset (shown on the right). Note that the soil fungi dataset does not have NAs, the bacteria dataset has NAs for all species for plots AEG33 and AEG34.*

Real NAs can happen when a full plot is missing from the dataset (e.g. AEG33 in Fig2) or when a combination of plot X species is not available. This second case only happens with arthropod datasets (e.g. pollinators) and we recommend to only use plots with information on all species (i.e. without NAs, using e.g. the command na.omit(dat)). We highlight this topic because it might cause issues when using the data to e.g. calculate richness of a given group. We recommend to always check how many plots were measured per DataID and per year.

```R
#Example code to add zeros back for the bacteria and fungi datasets. Also adds NAs for the plots with missing data. Note: this will heavily  increase the datasets size and might cause memory issues!:
#create all combinations of Plot x Species (value will be set to NA by default)
bac_dat<-setDT(bac_dat)[CJ(Species=Species,Plot=Plot,unique=T), on=.(Species, Plot)]
#replace NAs by zeros
bac_dat[is.na(value), value := 0 ]
#add back NAs for non measured plots (this can be done in the first line of code but is showed separately for clarity)
bac_dat[Plot %in% c(“AEG33”,”AEG34”), value:=NA]
```

**Special case: plots do not contain any species** : The code above adds all missing plots x species combinations. However in some cases a plot does not contain any species, i.e. it was sampled but no species were found, so it has zeros for all species (e.g. HEG07 symbiont.soilfungi in the year 2011). In this case, the plot is missing completely from the dataset (or from the subset of the dataset). The complete absence of species from a plot is, however, an important one and we recommend not to exclude these plots. To identify such cases, please check if the number of plots before adding the missing plots x species combinations is the expected one (148 plots for bacteria, 150 plots for soilfungi). In order to add back plots that do not contain species, we recommend to use the function `add_back_missing_plots_species_combinations` from our github folder [useful_functions](https://github.com/biodiversity-exploratories-synthesis/useful_functions). 



## Temporal information

Several taxa have information for more than one year. The use of a single year or multiple years highly depends on the research question. When using several years to calculate richness, the values could be either averaged or summed (if methods are compatible, see “grouping” paragraph). For abundance it is less straightforward: summing the number of individuals might be meaningful for short lived species (e.g. some arthropods) but not long-lived organisms (e.g. perennial plants, for which information is given on cover).

Note also that some plots might not have been surveyed in a given year but were surveyed other years (e.g. SEG33 in 2013 in the plant dataset ID27386). Keeping or removing those plots from the dataset will depend on the research question and analyses.



## Important notes on arthropods

1. **Temporal data**. The synthesis dataset does not contain the temporal arthropods dataset. This is because more groups were sampled in the 2008 arthropod dataset, therefore we focussed on the most complete year of sampling. However, for questions related to temporal dynamics or for more extensive surveys of some arthropod groups (Coleoptera, Aranaea, Hemiptera and Orthoptera), we strongly recommend to use the temporal datasets from the Arthropod Core Team (ID21969 ["Sweep net samples from grasslands since 2008: Araneae, Coleoptera, Hemiptera, Orthoptera"](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=21969) and 26008 [“List of plots without complete sampling for sweepnetting of arthropods on grassland EPs 2008 to 2017”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=26008))

2. **Larger taxonomic coverage**. Some insect groups were sampled in more detail with focussed methods. They cannot be easily merged to the synthesis dataset because of different sampling methods. However we strongly recommend to consider them:

   - 21207: [Dungwebs Species List 2014 & 2015 (Invertebrates, Scarabaeoidea, Dung Beetles)](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=21207)
   - 19826: [Orthoptera Density 2014 - all Grassland EPs - using biocenometer sampling](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=19826)
   - 20526: [Auchenorrhyncha Density 2015 - all Grassland EPs - using biocenometer sampling](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=20526)
   - 26026: [Moth abundance from light trapping on all grassland and forest plots 2018](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=26026)

   Please note that this list might not be exhaustive.

3. **Pollinators**. The trophic level “pollinators” comes from different datasets but the grouping can be used because it was checked and homogenised by specialists from the Arthropod Core Team. Not all taxonomic groups within pollinators were measured in the same plots, so we recommend to use only the plots where all groups were measured.

   The pollinators in the synthesis diversity dataset are derived from the same dataset included in the synthesis functions dataset. Therefore high correlations between these two are artificial and analyses using both measures would be circular (i.e. done with the same data).



## Specific notes for the diversity datasets

**Richness and abundance.** The column “type” provides information about the type of data: abundance, presence absence, cover, number of OTUs (operational taxonomic units), number of ASVs (exact sequence variants).

1. Do not use the number of OTUs (or ASVs) as a proxy for abundance! PFLA measures are a more appropriate way to describe the abundance of microorganisms (e.g. datasets 20250 for grasslands and 20926 for forests, please check for any updates on these dataset IDs). Number of OTUs/ASVs is highly dependent on methods and is not biologically meaningful. However, the number of OTUs can be used to calculate relative abundances (see original datasets metadata).



2. Rarefaction is needed to calculate the richness of bacteria and soil fungi (see above). We recommend contacting the data owners of these datasets to ask for the most up-to-date techniques of rarefaction for this type of data. The `R` package `phyloseq`, functions `phyloseq` and `rarefy_even_depth` are currently (November 2020) a good method to rarefy these datasets.

```R
# Example code to rarefy a microbial dataset (thanks to Kezia Goldmann and Johannes Sikorski for guidance on this topic)
dat <- otu_table(fungi.dat, taxa_are_rows = TRUE) # get a phyloseq object step1
pylodat <- phyloseq(dat) # get a phyloseq object step2

set.seed(1) #set a seed to get reproducible results
rarefied.dat <- rarefy_even_depth(phylodat) #sample size should be the smallest number of sequences per sample, alternatively, you can use (0.9*smallest number) to also shuffle the sample.
```

**Duplicates**. In very few cases, the same species were identified by two different datasets (e.g. some species were common in the ants and in the pollinator datasets). We removed all duplicate species occurrences and kept only the information from the datasets having more plots. If the number of sampled plots in the two datasets was similar, we prioritised the most extensive sampling for these species (e.g. some myriapods were collected together with other arthropods but we prioritised the information coming from the myriapod-specific sampling).



**Original datasets**. Original datasets might contain more information. For instance birds 2018 have information on birds detected at different scales; soil fungi have probabilities of species assignments, ants have abundance information). Please always check the metadata of the original datasets. Some of this information can be useful for some research questions.

**Taxonomic information.** Taxonomic information is contained in the columns “Group_broad” and “Group_fine”. The degree of resolution in the classification can vary between different groups and was chosen to optimise multidiversity analyses. Please be aware that different types of analyses might require other taxonomic/trophic classifications. Complete taxonomic classification can usually be obtained from the owners of the single datasets.



## Specific notes for the functions datasets

**Correlated functions.** Some functions are very similar because they describe the same process in a different way or because one is a part of the other. For instance “NRI” and “N_leaching_risk” describe the same process although measured with different methods, or “Biomass.2009” is part of the aggregated measure “Biomass” over all years. These should be carefully evaluated before using them together in an analysis.

**Pollinators**. See “Important notes on arthropods” above.

**Choice of functions.** The choice of functions to be included in an analysis and their categorisation depends on the research question. Previous synthesis analyses can provide some guidance as well as some categories (e.g. stock and fluxes) provided in the metadata file (see [Organisation](#Organisation) paragraph above).



## Varia

1. Please always use the scripts provided in Bexis with caution (e.g ID22046 [“R scripts usable for dataset: Assembled RAW diversity from grassland EPs (2010-2016) for multidiversity synthesis”](https://www.bexis.uni-jena.de/Data/ShowXml.aspx?DatasetId=22046) is mostly given as an example) and please report back any spotted errors or issues via email and/or via [Github](https://github.com/biodiversity-exploratories-synthesis).

