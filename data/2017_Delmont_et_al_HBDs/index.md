---
layout: page
title: Recovering HBDs from TARA Oceans Metagenomes
modified: 2017-04-20
excerpt: "A complete workflow behind the manuscript 'Nitrogen-fixing populations of Planctomycetes and Proteobacteria are abundant in the surface ocean' by Delmont et al"
comments: true

---

{% include _toc.html %}

{% capture images %}{{site.url}}/data/2017_Delmont_et_al_HBDs/images{% endcapture %}

{:.notice}
The URL [http://biorxiv.org/content/early/2017/04/23/129791](http://biorxiv.org/content/early/2017/04/23/129791) serves the pre-print of the study described in this document.

{:.notice}
The URL [http://merenlab.org/data/2017_Delmont_et_al_HBDs/](http://merenlab.org/data/2017_Delmont_et_al_HBDs/) serves the most up-to-date version of this document.


The document contains program names and exact parameters used throughout every step of the analysis of the TARA Oceans metagenomes, which relied predominantly on the open-source analysis platform, [anvi’o](http://merenlab.org/software/anvio) (Eren et al., 2015).
All anvi'o analyses in this document are performed using the anvi'o version `v2.3.0`. Please see [the installation notes]({% post_url anvio/2016-06-26-installation-v2 %}) to acquire the appropriate version through PyPI, Docker, or GitHub.


This section explains how to download short metagenomic reads from the TARA Oceans project (Sunagawa et al., 2015), quality-filtering the raw data, defining metagenomic sets, and generating co-assemblies for each set.

The following file contains FTP URLs for each raw data file for 93 samples:

{:.notice}
[http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/ftp-links-for-raw-data-files.txt](http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/ftp-links-for-raw-data-files.txt)

You can get a copy of this file into your work directory, 

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/ftp-links-for-raw-data-files.txt
``` 

and download each of the raw sequencing data file from the EMBL servers:

``` bash
for url in `cat ftp-links-for-raw-data-files.txt`
do
    wget $url
done
```

We defined 12 'metagenomic sets' for geographically bound locations TARA Oceans samples originated from. These metagenomic sets are described in Figure 1:

![TARA]({{images}}/Figure_01.png){:.center-img .width-70}

We tailored our sample naming schema for convenience and reproducibility. The following address contains a file with three-letter identifiers for each of the 12 metagenomic set, which will become prefixes for each sample name for direct access to all samples from a given metagenomic set:

{:.notice}
[http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/sets.txt](http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/sets.txt)

This file will be referred to as `sets.txt` throughout the document, and you can get a copy of this file into your work directory:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/sets.txt
``` 

The contents of the TARA sets file should look like this:

``` bash
$ cat sets.txt
ANE
ANW
ASE
ASW
ION
IOS
MED
PON
PSE
PSW
RED
SOC
``` 

We used these three-letter prefixes to name each of the 93 samples, and to associate them with metagenomic sets with which they were affiliated. Sample names, and which raw data files are associated with each sample is explained in the following TAB-delimited file:

{:.notice}
[http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/samples.txt](http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/samples.txt)

This file will be referred to as `samples.txt` throughout the document, and you can get a copy of this file into your work directory:

``` bash
wget http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/samples.txt
```

TARA samples file should look like this:

``` bash
$ wc -l samples.txt
      94 samples.txt
$ head samples.txt
sample     r1                            r2
MED_18_05M ERR599140_1.gz,ERR598993_1.gz ERR599140_2.gz,ERR598993_2.gz
MED_23_05M ERR315858_1.gz,ERR315861_1.gz ERR315858_2.gz,ERR315861_2.gz
MED_25_05M ERR599043_1.gz,ERR598951_1.gz ERR599043_2.gz,ERR598951_2.gz
MED_30_05M ERR315862_1.gz,ERR315863_1.gz ERR315862_2.gz,ERR315863_2.gz
RED_31_05M ERR599106_1.gz,ERR598969_1.gz ERR599106_2.gz,ERR598969_2.gz
RED_32_05M ERR599041_1.gz,ERR599116_1.gz ERR599041_2.gz,ERR599116_2.gz
RED_33_05M ERR599049_1.gz,ERR599134_1.gz ERR599049_2.gz,ERR599134_2.gz
RED_34_05M ERR598959_1.gz,ERR598991_1.gz ERR598959_2.gz,ERR598991_2.gz
ION_36_05M ERR599143_1.gz,ERR598966_1.gz ERR599143_2.gz,ERR598966_2.gz
```

This file above also represents the standard input for illumina-utils library for processing the raw metagenomic data for each sample. 


To produce the configuration files [illumina utils requires](https://github.com/merenlab/illumina-utils#config-file-format), we first run the program `iu-gen-configs` on the file `samples.txt`:

``` bash
iu-gen-configs samples.txt
```



``` bash
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    iu-filter-quality-minoche $sample.ini --ignore-deflines
done
```

``` bash
$ cat ASW_82_40M-STATS.txt
number of pairs analyzed      : 222110286
total pairs passed            : 204045433 (%91.87 of all pairs)
total pairs failed            : 18064853 (%8.13 of all pairs)
  pairs failed due to pair_1  : 3237955 (%17.92 of all failed pairs)
  pairs failed due to pair_2  : 8714230 (%48.24 of all failed pairs)
  pairs failed due to both    : 6112668 (%33.84 of all failed pairs)
  FAILED_REASON_N             : 302725 (%1.68 of all failed pairs)
  FAILED_REASON_C33           : 17762128 (%98.32 of all failed pairs)
```

The resulting `*-QUALITY_PASSED_R*.fastq.gz` files in the work directory represent the main inputs for all co-assembly and mapping steps.


<div class="extra-info" markdown="1">

<span class="extra-info-header">The rationale for co-assemblies</span>
The rationale of doing this was that metagenomes originating from adjacent geographic regions (e.g., the Mediterranean Sea) were more likely to overlap in the sequence space, increasing the mean coverage (and hence the extent of reconstruction) of microbial genomes during the co-assembly.
</div>

do
    megahit -1 $SET*-QUALITY_PASSED_R1.fastq.gz \
            -2 $SET*-QUALITY_PASSED_R2.fastq.gz \
            --min-contig-len 1000 \
            -o $SET-RAW.fa \
            --num-cpu-threads 80
done
```

Assembly of large metagenomic data is a very memory-intensive process, and we used a computer with 2 TB memory and 80 cores to co-assemble each metagenomic set in a serial manner.

for SET in `cat sets.txt`
do
                               --simplify-names \
                               -o $SET-RAW-FIXED.fa \
                               --prefix $SET
    mv $SET-RAW-FIXED.fa $SET-RAW.fa
We made publicly available ([doi:10.6084/m9.figshare.4902920](https://doi.org/10.6084/m9.figshare.4902920)) the resulting FASTA files as `RAW-ASSEMBLIES-1000nt`.


``` bash
for SET in `cat sets.txt`
do
    # skip the Southern Ocean set
    if [ "$SET" == "SOC" ]; then continue; fi
    
    anvi-script-reformat-fasta $SET-RAW.fa –-min-len 2500 --simplify-names -o $SET.fa

An exception to the `>2.5 kbp` selection was the Southern Ocean metagenomic set (SOC), for which we generated a file containing scaffolds `>5 kbp` to increase the accuracy of tetra-nucleotide frequency signal to recover from the smaller contribution of differential coverage due to the limited number of samples collected from this region:
anvi-script-reformat-fasta SOC-RAW.fa –-min-len 5000 --simplify-names -o SOC.fa

{:.notice}

``` bash
for SET in `cat sets.txt`
do
    anvi-run-hmms -c $SET-CONTIGS.db --num-threads 20

This step stores single-copy core gene hits per scaffold in the corresponding anvi'o contigs database, which becomes instrumental to assess the completion and redundancy of bins in real time during the manual binning and refinement steps in the anvi’o interactive interface.
for SET in `cat sets.txt`
do
    bowtie2-build $SET.fa $SET
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    
    # determine which metagenomic set the $sample belongs to:
    SET=`echo $sample | awk 'BEGIN {FS="_"}{print $1}'`
    
    # do the bowtie mapping to get the SAM file:
    bowtie2 --threads 20 \
            -x $SET \
            -1 $sample-QUALITY_PASSED_R1.fastq.gz \
            -2 $sample-QUALITY_PASSED_R2.fastq.gz \
            --no-unal \
            -S $sample.sam
    
    # covert the resulting SAM file to a BAM file:
    samtools view -F 4 -bS $sample.sam > $sample-RAW.bam

    # sort and index the BAM file:
done
```
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    
    # determine which metagenomic set the $sample bleongs to:
    SET=`echo $sample | awk 'BEGIN {FS="_"}{print $1}'`
    
    anvi-profile -c $SET-CONTIGS.db \
                 -i $sample.bam \
                 --skip-SNV-profiling \
                 --num-threads 16 \
                 -o $sample
done
```

{:.notice}
Please read the following article for parallelization of anvi'o profiling (details of which can be important to consider especially if you are planning to clusterize it): [The new anvi'o BAM profiler]({% post_url anvio/2017-03-07-the-new-anvio-profiler %}).

``` bash
for SET in `cat sets.txt`
do
    anvi-merge $SET*/PROFILE.db -o $SET-MERGED -c $SET-CONTIGS.db 

``` bash
for SET in `cat sets.txt`
do
    anvi-cluster-with-concoct -c $SET-CONTIGS.db \
                              -p $SET-MERGED/PROFILE.db \
                              --num-clusters 100 \
                              -C CONCOCT
```

This step generates a collection of 100 clusters called `CONCOCT` in each of the anvi'o merged profile database for each metagenomic set.

Two exceptions were the Southern Ocean (SOC) and Pacific Ocean southeast (PSE) metagenomic sets, for which we generated 25 and 150 CONCOCT clusters due to their differences in size from other sets. We overwrote the existing CONCOCT clusters in these profiles the following way:

``` bash
anvi-cluster-with-concoct -c SOC-CONTIGS.db -p SOC-MERGED/PROFILE.db --num-clusters 25 -C CONCOCT
anvi-cluster-with-concoct -c PSE-CONTIGS.db -p PSE-MERGED/PROFILE.db --num-clusters 150 -C CONCOCT
```

<div class="extra-info" markdown="1">

<span class="extra-info-header">The rationale for constraining the number of clusters</span>
Constraining the number of clusters to be automatically identified to a smaller number than the expected number of genomes, and then refining the resulting clusters to resolve population genomes is a heuristic we have been using to bin large metagenomic assemblies and discussed in Eren et al. (2015). Please see this article for a more detailed discussion of reasons behind it: [http://merenlab.org/tutorials/infant-gut/#binning](http://merenlab.org/tutorials/infant-gut/#binning).
</div>

### The summary of initial CONCOCT clusters

We then summarized the CONCOCT clusters using the program `anvi-summarize`, in order to access to the summary of each CONCOCT cluster in each metagenomic set, including number of scaffolds, total length, N50, taxonomy, completion, and redundancy:
for SET in `cat sets.txt`
do
    anvi-summarize -c $SET-CONTIGS.db \
                   -p $SET-MERGED/PROFILE.db \
                   -C CONCOCT \
                   -o $SET-SUMMARY-CONCOCT
```

            -p ANW-MERGED/PROFILE.db \
            -C CONCOCT \
            -b Bin_1
<div class="extra-info" markdown="1">

<span class="extra-info-header">A note on metagenomic bin refinement</span>
While it is a labor-intensive step, we find it to be critical for the accuracy of metagenomic bins. We encourage careful curation of each automatically identified cluster in large metagenomic assembly and binning projects. In the case of our study, we manually refined a total of 1,175 CONCOCT clusters (2,624,194 scaffolds), and generated 30,244 bins with less than 10% of redundancy based on the average estimations from four bacterial single-copy gene collections (except for the bins we identified as eukaryotic genome bins).

{:.notice}
[http://merenlab.org/2016/06/09/assessing-completion-and-contamination-of-MAGs/](http://merenlab.org/2016/06/09/assessing-completion-and-contamination-of-MAGs/)

This article demonstrates the importance of refinement:

{:.notice}
[http://merenlab.org/2017/01/03/loki-the-link-archaea-eukaryota/](http://merenlab.org/2017/01/03/loki-the-link-archaea-eukaryota/)

And this video demonstrates the key capabilities of the anvi'o interactive interface for refinement tasks:
</div>


``` bash
for SET in `cat sets.txt`
do
    anvi-rename-bins -c $SET-CONTIGS.db \
                     -p $SET-MERGED/PROFILE.db \
                     --collection-to-read CONCOCT \
                     --collection-to-write FINAL \
                     --use-SCG-averages \
                     --call-MAGs \
                     --size-for-MAG 2 \
                     --min-completion-for-MAG 70 \
                     --max-redundancy-for-MAG 10 \
                     --prefix TARA-$SET \
                     --report-file $SET_renaming_bins.txt
done
```

{:notice}
For the TARA Oceans project we used the parameter `--max-redundancy-for-MAG 100` instead of what makes more sense, `--max-redundancy-for-MAG 10`, in order to not miss eukaryotic bins that are larger than 2 Mbp.


``` bash
for SET in `cat sets.txt`
do
    # get each MAG name in the set:
    MAGs=`anvi-script-get-collection-info -c $SET-CONTIGS.db \ 
                                          -p $SET-MERGED/PROFILE.db \
                                          -C FINAL | \
          grep MAG | \
          awk '{print $1}'`

    # go through each MAG, and invoke anvi-refine tasks
    for MAG in MAGs
    do
        anvi-refine -c $SET-CONTIGS.db \
                    -p $SET-MERGED/PROFILE.db \
                    -C FINAL \
                    -b $MAG
    done
done
```
for SET in `cat sets.txt`
do
    anvi-summarize -c $SET-CONTIGS.db \
                   -p $SET-MERGED/PROFILE.db \
                   -C FINAL \
                   -o $SET-SUMMARY
done
```

First, we used the program `anvi-script-reformat-fasta` to __rename scaffolds contained in each MAG accordingly to their MAG ID__, and store their FASTA files in a separate directory called `REDUNDANT-MAGs` for downstream analyses.

``` bash
mkdir REDUNDANT-MAGs
```

``` bash
for SET in `cat sets.txt`
do
    # get each MAG name in the set:
    MAGs=`anvi-script-get-collection-info -c $SET-CONTIGS.db \ 
                                          -p $SET-MERGED/PROFILE.db \
                                          -C FINAL | \
          grep MAG | \
          awk '{print $1}'`

    # go through each MAG, in each SUMMARY directory, and store a
    # copy of the FASTA file with proper deflines in the REDUNDANT-MAGs
    # directory:
    for MAG in MAGs
    do
        anvi-script-reformat-fasta $SET-SUMMARY/bin_by_bin/$MAG/$MAG-contigs.fa \
                                   --simplify-names \
                                   --prefix $MAG \
                                   -o REDUNDANT-MAGs/$MAG.fa
    done
done
```
This step provides a convenient naming schema for all scaffolds. For instance, a scaffold name in the MAG `00001` recovered from the `ANW` set followed the convention `ANW_MAG_00001_000000000001`.



After completion of the metagenomic binning, curation of the MAGs, and renaming of scaffolds in them, here, we finally start focusing on the distribution and detection of each MAG across all metagenomes regardless of from which metagenomic set they were recovered. As previously explained, we call these MAGs "redundant MAGs" since we assume that some of them correspond to the same population identified from multiple metagenomic sets.

In later sections, we will identify and collapse that redundancy to generate a final collection of non-redundant MAGs. This profiling step is necessary to generate information that will be required for those steps.

First we concatenated scaffolds from all the redundant MAGs into a single FASTA file for mapping and processing:


and used the program `anvi-gen-contigs-database` with default parameters to generate a CONTIGS database, and run HMMs:

``` bash
anvi-run-hmms -c REDUNDANT-MAGs-CONTIGS.db --num-threads 20

``` bash
# building the Botwie2 database
bowtie2-build REDUNDANT-MAGs.fa REDUNDANT-MAGs

# going through each metagenomic sample, and mapping short reads
# against the 1,077 redundant MAGs
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    
    bowtie2 --threads 20 \
            -x REDUNDANT-MAGs\
            -1 $sample-QUALITY_PASSED_R1.fastq.gz \
            -2 $sample-QUALITY_PASSED_R2.fastq.gz \
            --no-unal \
            -S $sample-in-RMAGs.sam
    
    # covert the resulting SAM file to a BAM file:
    samtools view -F 4 -bS $sample-in-RMAGs.sam > $sample-in-RMAGs-RAW.bam

    # sort and index the BAM file:
done

# profile each BAM file
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
        
    anvi-profile -c REDUNDANT-MAGs-CONTIGS.db \
                 -i $sample-in-RMAGs.bam \
                 --skip-SNV-profiling \
                 --num-threads 16 \
                 -o $sample-in-RMAGs
done

# merge resulting profiles into a single anvi'o merged profile
anvi-merge *-in-RMAGs/PROFILE.db \
           -c REDUNDANT-MAGs-CONTIGS.db \
           --skip-concoct-binning \
           -o REDUNDANT-MAGs-MERGED
```

Although the anvi'o profile database in `REDUNDANT-MAGs-MERGED` describes the distribution and detection statistics of all scaffolds in all MAGs, it does not contain a collection that describes the scaffold-bin affiliations. Thanks to our previous naming consistency, here we can implement a simple workaround to generate a text file that describes these connections:

``` bash
for split_name in `sqlite3 REDUNDANT-MAGs-CONTIGS.db 'select split from splits_basic_info'`
do
    # in this loop $split_name goes through names like this: ANW_MAG_00001_000000000001,
    # ANW_MAG_00001_000000000002, ANW_MAG_00001_000000000003, ...; so we can extract
    # the MAG name it belongs to:
    MAG=`echo $split_name | awk 'BEGIN{FS="_"}{print $1"_"$2"_"$3"_"$4}'`
    
    # print it out with a TAB character
    echo -e "$split_name\t$MAG"
done > REDUNDANT-MAGs-COLLECTION.txt
```

We then used the program `anvi-import-collection` to import this collection into the anvi'o profile database:

                       -c REDUNDANT-MAGs-CONTIGS.db \
                       -p REDUNDANT-MAGs-MERGED/PROFILE.db \
                       -C REDUNDANT-MAGs

               -p REDUNDANT-MAGs-MERGED/PROFILE.db \
               -C REDUNDANT-MAGs \
               -o REDUNDANT-MAGs-SUMMARY

Briefly, we used both the Average Nucleotide Identity (ANI), and Pearson correlation values to identify MAGs with high sequence similarity, and distribute similarly across metagenomes.


#### Generating correlations input

We first determined the Pearson correlation coefficient of each pair of MAGs based on their relative distribution across the 93 metagenomes in R:

#!/usr/bin/env Rscript

library(reshape2)


mean_coverage <- t(read.delim(file="REDUNDANT-MAGs-SUMMARY/bins_across_samples/mean_coverage.txt",
                              header = TRUE,
                              stringsAsFactors = FALSE,check.names = F, row.names = 1))

correlations <- melt(data = cor(mean_coverage, use="complete.obs", method="pearson"))

names(correlations) <- c('MAG_1', 'MAG_2', 'correlation')

write.table(correlations, "REDUNDANT-MAGs-PEARSON.txt", sep="\t", quote=FALSE,  col.names=NA)
```

The following output provides a glimpse from the file `REDUNDANT-MAGs-PEARSON.txt`:


``` R
head REDUNDANT-MAGs-PEARSON.txt
	MAG_1	MAG_2	correlation
1	TARA_ANE_MAG_00100	TARA_ANE_MAG_00100	1
2	TARA_ASE_MAG_00034	TARA_ANE_MAG_00100	-0.0365504813801264
3	TARA_ASW_MAG_00048	TARA_ANE_MAG_00100	-0.0269000918790998
4	TARA_ASW_MAG_00049	TARA_ANE_MAG_00100	-0.0699740876820359
5	TARA_ASW_MAG_00050	TARA_ANE_MAG_00100	0.0523666251766757
6	TARA_MED_MAG_00144	TARA_ANE_MAG_00100	0.816445902311676
7	TARA_PON_MAG_00089	TARA_ANE_MAG_00100	-0.0588480193473581
8	TARA_PON_MAG_00090	TARA_ANE_MAG_00100	-0.0480963479362033
9	TARA_PON_MAG_00091	TARA_ANE_MAG_00100	-0.0163576627063678
10	TARA_PON_MAG_00092	TARA_ANE_MAG_00100	-0.0563507978838984
```

To avoid a large number of comparisons of unrelated MAGs in later steps to compute the Average Nucleotide Identity (ANI) between them, we generated a two-columns TAB-delimited file that describes the affiliations for each MAG, utilizing phylum-level taxonomy of MAGs from CheckM results (we used class-level taxonomy for MAGs that belonged to Proteobacteria to further split them into smaller groups). Our scripts in later steps used this file to make sure MAGs were compared to estimate the ANI only if they were affiliated with the same taxon (i.e., we did not attempt to compute the ANI between two MAGs if they belonged to different phyla).

The following output provides a glimpse from the file `REDUNDANT-MAGs-AFFILIATIONS.txt`:
head REDUNDANT-MAGs-AFFILIATIONS.txt
TARA_PON_MAG_00066	Acidobacteria
TARA_PON_MAG_00072	Acidobacteria
TARA_RED_MAG_00008	Acidobacteria
TARA_RED_MAG_00030	Acidobacteria
TARA_RED_MAG_00081	Acidobacteria
TARA_ANE_MAG_00007	Actinobacteria
TARA_ANE_MAG_00045	Actinobacteria
TARA_ANE_MAG_00059	Actinobacteria
TARA_ANE_MAG_00074	Actinobacteria
TARA_ANE_MAG_00076	Actinobacteria
TARA_ANW_MAG_00037	Actinobacteria
TARA_ANW_MAG_00052	Actinobacteria
TARA_ANW_MAG_00054	Actinobacteria
TARA_ANW_MAG_00060	Actinobacteria
```

#### Generating lengths input

We also generated a file, `REDUNDANT-MAGs-LENGTH.txt`, that described the length of each MAG, to accurately compute the aligned fractions of MAGs for ANI:

``` bash
awk '{print $1"\t"$3}' REDUNDANT-MAGs-SUMMARY/general_bins_summary.txt > REDUNDANT-MAGs-LENGTH.txt
```

The following output provides a glimpse from the file `REDUNDANT-MAGs-LENGTH.txt`:

TARA_ANE_MAG_00001	1826698
TARA_ANE_MAG_00002	3133556
TARA_ANE_MAG_00003	1602643
TARA_ANE_MAG_00004	2301261
TARA_ANE_MAG_00005	2402163
TARA_ANE_MAG_00006	1424655
TARA_ANE_MAG_00007	2448482
TARA_ANE_MAG_00008	1713062
TARA_ANE_MAG_00009	5226535

#### Generating basic MAG stats input


``` bash
B="REDUNDANT-MAGs-SUMMARY/bin_by_bin/"
echo -e "MAG\tlength\tcompletion\tredundancy" > REDUNDANT-MAGs-STATS.txt
for MAG in `ls $B | grep MAG`
do
    length=`cat $B/$MAG/$MAG-total_length.txt`
    completion=`cat $B/$MAG/$MAG-percent_complete.txt`
    redundancy=`cat $B/$MAG/$MAG-percent_redundancy.txt`
    echo -e "$MAG\t$length\t$completion\t$redundancy"
done >> REDUNDANT-MAGs-STATS.txt
```

Following output displays first few lines of the output file`REDUNDANT-MAGs-STATS.txt`:
head    REDUNDANT-MAGs-STATS.txt
MAG    length    completion    redundancy
TARA_ANE_MAG_00019    1848764    87.04    0.62
TARA_ANE_MAG_00024    1885787    84.57    1.23
TARA_ANE_MAG_00027    1919013    84.57    1.85
TARA_ANE_MAG_00030    1649495    83.95    2.47
TARA_ANE_MAG_00042    1899402    80.25    2.47
TARA_ANE_MAG_00052    1252467    75.93    0.62
TARA_ANE_MAG_00055    1540581    75.31    0.62
TARA_ANE_MAG_00063    1138287    70.37    0
TARA_ANE_MAG_00065    1552464    72.22    3.09
TARA_ANW_MAG_00024    1809221    85.8    1.85
TARA_ANW_MAG_00033    1323836    79.01    0.62
TARA_ANW_MAG_00043    1179335    75.93    1.85
TARA_ANW_MAG_00045    1634296    75.93    2.47
TARA_ANW_MAG_00046    1302485    74.69    1.85

We added an additional column, `domain`, to the resulting output file, `REDUNDANT-MAGs-STATS.txt` which described the domain-level affiliation of each MAG (Bacteria, Archaea, or Eukaryota).

### Generating raw ANI estimates

Files we generated so far represent inputs for our Python scripts that we will run in a serial manner to generate the file `REDUNDANT-MAGs-ANI.txt` which will be processed further.

#### Generating the list of NUCmer jobs

We elected to use NUCmer (Delcher et al., 2002) to generate raw alignment and sequence similarity scores between scaffolds of MAGs to finally determine the average nucleotide identity (ANI) between each pair of MAG.

We used the first one to generate the list of comparison jobs to run on a cluster:

``` bash
wget https://goo.gl/1KlGUH -O 01-gen-nucmer-jobs.py
python 01-gen-nucmer-jobs.py REDUNDANT-MAGs-AFFILIATIONS.txt REDUNDANT-MAGs/ > NUCMER_JOBS_TO_RUN.sh
```

This script results in all the command lines that must be run to produce the necessary output for the next steps of the analysis. Since high-performance compute environments differ from institution to institution, we elected to let the user decide how to run each of these processes. The most straightforward yet time consuming way to run them would have been this, but we strongly encourage you to take a look at the contents of this file first:

``` bash
bash NUCMER_JOBS_TO_RUN.sh
```

#### Processing the NUCmer outputs

After running each job in the previous step, we run the second script to generate the `REDUNDANT-MAGs-ANI.txt` file:

``` bash
wget https://goo.gl/TtY3wO -O 02-estimate-similarity.py
python 02-estimate-similarity.py REDUNDANT-MAGs-LENGTH.txt REDUNDANT-MAGs-ANI.txt
```
|:--|:--:|:--|:--:|:--:|:--:|
|TARA_PON_MAG_00001|3583974|TARA_ION_MAG_00032|3679393|2376803|98.725325|
|TARA_IOS_MAG_00050|1469026|TARA_RED_MAG_00058|2210067|1223112|99.613934|
|TARA_IOS_MAG_00050|1469026|TARA_ASW_MAG_00005|1992647|1075083|99.589960|
|TARA_IOS_MAG_00050|1469026|TARA_ANW_MAG_00012|2187424|1225823|99.575093|
|TARA_IOS_MAG_00050|1469026|TARA_MED_MAG_00021|2315206|14358|98.371665|
|TARA_PSW_MAG_00005|3628000|TARA_PON_MAG_00021|3638235|3526133|99.969765|
|TARA_PSW_MAG_00005|3628000|TARA_ANW_MAG_00001|3686209|3603131|99.986695|
|TARA_PSW_MAG_00005|3628000|TARA_PSE_MAG_00008|3594348|3589904|99.975569|
|TARA_ASW_MAG_00006|3084735|TARA_ANE_MAG_00002|3133556|3031188|99.986508|
|TARA_ASW_MAG_00006|3084735|TARA_IOS_MAG_00008|3359870|1697|99.940000|
|(...)|(...)|(...)|(...)|(...)|(...)|

{:.notice}
[http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/REDUNDANT-MAGs-ANI.txt](http://merenlab.org/data/2017_Delmont_et_al_HBDs/files/REDUNDANT-MAGs-ANI.txt)

``` bash
wget https://goo.gl/m2IuMz -O 03-gen-table-for-redundancy-analysis.py
wget https://goo.gl/JLPPkq -O 04-identify-redundant-groups.py
wget https://goo.gl/2FgGjU -O redundancy.py
```

Then we first generated a table that describes the pairwise relationships between MAGs using previous output files:
                                               -C REDUNDANT-MAGs-PEARSON.txt \
                                               -i REDUNDANT-MAGs-ANI.txt \
                                               -o MAGs-PAIRWISE-DATA.txt
|:--|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|0|98.72|0.37|Bacteria|TARA_ION_MAG_00032|3679393|79.22|TARA_PON_MAG_00001|3583974|93.50|2376736|66.31|
|1|99.61|0.97|Bacteria|TARA_RED_MAG_00058|2210067|80.4|TARA_IOS_MAG_00050|1469026|69.75|1223036|83.25|
|2|99.59|0.97|Bacteria|TARA_ASW_MAG_00005|1992647|88.95|TARA_IOS_MAG_00050|1469026|69.75|1075038|73.18|
|3|99.57|0.98|Bacteria|TARA_ANW_MAG_00012|2187424|90.77|TARA_IOS_MAG_00050|1469026|69.75|1225775|83.44|
|4|98.37|-0.05|Bacteria|TARA_IOS_MAG_00050|1469026|69.75|TARA_MED_MAG_00021|2315206|91.34|14358|0.97|
|5|99.97|1.0|Bacteria|TARA_PON_MAG_00021|3638235|84.41|TARA_PSW_MAG_00005|3628000|94.31|3525778|97.18|
|6|99.98|1.0|Bacteria|TARA_PSW_MAG_00005|3628000|94.31|TARA_ANW_MAG_00001|3686209|94.49|3603131|99.31|
|7|99.97|1.0|Bacteria|TARA_PSE_MAG_00008|3594348|94.04|TARA_PSW_MAG_00005|3628000|94.31|3589740|99.87|
|8|99.98|1.0|Bacteria|TARA_ASW_MAG_00006|3084735|88.94|TARA_ANE_MAG_00002|3133556|92.84|3031188|98.26|
|9|99.94|0.07|Bacteria|TARA_ASW_MAG_00006|3084735|88.94|TARA_IOS_MAG_00008|3359870|92.74|1697|0.05|
|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|(...)|

Then, from these relations we resolved the redundant groups the following way:

```bash
python 04-identify-redundant-groups.py -d MAGs-PAIRWISE-DATA.txt \
                                       -m REDUNDANT-MAGs-STATS.txt \
                                       -o REDUNDANT-MAGs-GROUPs.txt
```


|:--|:--:|:--:|:--:|
|TARA_ANE_MAG_00002|group_0003|1|5|
|TARA_ANE_MAG_00005|group_0051|1|2|
|TARA_ANE_MAG_00007|group_0031|1|5|
|TARA_ANE_MAG_00021|group_0008|0|3|
|TARA_ANE_MAG_00039|group_0039|1|2|
|TARA_ANE_MAG_00045|group_0035|0|2|
|TARA_ANE_MAG_00047|group_0020|0|3|
|TARA_ANE_MAG_00067|group_0037|0|2|
|TARA_ANE_MAG_00072|group_0027|0|2|
|(...)|(...)|(...)|(...)|

{:.notice}
We made publicly available ([doi:10.6084/m9.figshare.4902923](https://doi.org/10.6084/m9.figshare.4902923)) the FASTA files for our final collection of non-redundant MAGs in `NON-REDUNDANT-MAGs`.


``` bash
# create a single FASTA file for NON-REDUNDANT-MAGs

# generate an anvi'o contigs database:
anvi-run-hmms -c NON-REDUNDANT-MAGs-CONTIGS.db --num-threads 20
bowtie2-build NON-REDUNDANT-MAGs.fa NON-REDUNDANT-MAGs

# going through each metagenomic sample, and mapping short reads
# against the 957 non-redundant MAGs
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
    
    bowtie2 --threads 20 \
            -x NON-REDUNDANT-MAGs \
            -1 $sample-QUALITY_PASSED_R1.fastq.gz \
            -2 $sample-QUALITY_PASSED_R2.fastq.gz \
            --no-unal \
            -S $sample-in-NRMAGs.sam
    
    # covert the resulting SAM file to a BAM file:
    samtools view -F 4 -bS $sample-in-NRMAGs.sam > $sample-in-NRMAGs-RAW.bam

    # sort and index the BAM file:
done

# profile each BAM file
for sample in `awk '{print $1}' samples.txt`
do
    if [ "$sample" == "sample" ]; then continue; fi
        
    anvi-profile -c NON-REDUNDANT-MAGs-CONTIGS.db \
                 -i $sample-in-NRMAGs.bam \
                 --skip-SNV-profiling \
                 --num-threads 16 \
                 -o $sample-in-NRMAGs
done

# merge resulting profiles into a single anvi'o merged profile
anvi-merge *-in-NRMAGs/PROFILE.db \
           -c NON-REDUNDANT-MAGs-CONTIGS.db \
           --skip-concoct-binning \
           -o NON-REDUNDANT-MAGs-MERGED

# create an anvi'o collection file for non-redundant MAGs:
for split_name in `sqlite3 NON-REDUNDANT-MAGs-CONTIGS.db 'select split from splits_basic_info'`
do
    # in this loop $split_name goes through names like this: ANW_MAG_00001_000000000001,
    # ANW_MAG_00001_000000000002, ANW_MAG_00001_000000000003, ...; so we can extract
    # the MAG name it belongs to:
    MAG=`echo $split_name | awk 'BEGIN{FS="_"}{print $1"_"$2"_"$3"_"$4}'`
    
    # print out the collections line
    echo -e "$split_name\t$MAG"
done > NON-REDUNDANT-MAGs-COLLECTION.txt

# import the collection into the anvi'o merged profile database
# for non-redundant MAGs:
anvi-import-collection NON-REDUNDANT-MAGs-COLLECTION.txt \
                       -c NON-REDUNDANT-MAGs-CONTIGS.db \
                       -p NON-REDUNDANT-MAGs-MERGED/PROFILE.db \
                       -C NON-REDUNDANT-MAGs
               -p NON-REDUNDANT-MAGs-MERGED/PROFILE.db \
               -C NON-REDUNDANT-MAGs \
               -o NON-REDUNDANT-MAGs-SUMMARY

This collection represents the final 957 non-redundant MAGs in our study.
{:.notice}
We made publicly available ([doi:10.6084/m9.figshare.4902926](https://doi.org/10.6084/m9.figshare.4902926)) the summary output of non-redundant MAGs in `NON-REDUNDANT-MAGs-SUMMARY`.


## Creating self-contained profiles for MAGs

We used `anvi-split` to create individual, self-contained anvi'o profile and contig databases for each non-redundant MAG. The program `anvi-split` creates a comprehensive subset of the merged profile database so MAGs can be downloaded independently, and visualized interactively:

``` bash
anvi-split -c NON-REDUNDANT-MAGs-CONTIGS.db \
           -p NON-REDUNDANT-MAGs-MERGED/PROFILE.db \
           -C NON-REDUNDANT-MAGs \
           -o NON-REDUNDANT-MAGs-SPLIT
```

The resulting directory `NON-REDUNDANT-MAGs-SPLIT` contains a subdirectory for each MAG:

``` bash
$ ls NON-REDUNDANT-MAGs-SPLIT | head
TARA_ANE_MAG_00001
TARA_ANE_MAG_00002
TARA_ANE_MAG_00003
TARA_ANE_MAG_00004
TARA_ANE_MAG_00005
TARA_ANE_MAG_00006
TARA_ANE_MAG_00007
TARA_ANE_MAG_00008
TARA_ANE_MAG_00009
TARA_ANE_MAG_00010
$ ls NON-REDUNDANT-MAGs-SPLIT/TARA_ANE_MAG_00001
AUXILIARY-DATA.h5.gz  CONTIGS.db  CONTIGS.h5  PROFILE.db
```

{:.notice}
We made publicly available ([doi:10.6084/m9.figshare.4902941](https://doi.org/10.6084/m9.figshare.4902941)) the individual non-redundant MAGs in `NON-REDUNDANT-MAGs-SPLIT`.



``` bash
wget https://goo.gl/KXp3iQ -O Brown_et_al-CPR-Campbell_et_al_BSCG.txt
anvi-script-gen-CPR-classifier Brown_et_al-CPR-Campbell_et_al_BSCG.txt \
                               -o cpr-bscg.classifier 
```

and then used the program `anvi-script-predict-CPR-genomes`:
                                --report-only-cpr \
                                -c NON-REDUNDANT-MAGs-CONTIGS.db \
                                -p NON-REDUNDANT-MAGs-MERGED/PROFILE.db\
                                -C NON-REDUNDANT-MAGs\
                                cpr-bscg.classifier

Please visit [http://merenlab.org/2016/04/17/predicting-CPR-Genomes/](http://merenlab.org/2016/04/17/predicting-CPR-Genomes/) for the details of the Random Forest-based classification of CPR genomes.


One immediate advantage of metagenomic assemblies is the recovery of entire genes in scaffolds, compared to the use of short reads alone, which in most cases cover only a small fraction of a gene. Genes in scaffolds can then be used to screen for functions of interest with more precision and accuracy. For instance, in the context of our study we identified genes from the assembly of each metagenomic set, translated amino acid sequences, and then searched for sequences corresponding to the nitrogenase nifH needed for nitrogen fixation.

for SET in `cat sets.txt`
do
    anvi-script-reformat-fasta $SET-RAW.fa –-min-len 1000 --simplify-names -o $SET-1000nt.fa
    anvi-gen-contigs-database -f $SET-1000nt.fa -o $SET-CONTIGS.db
    anvi-get-aa-sequences-for-gene-calls -c $SET-CONTIGS.db -o $SET-AA_sequences.fa
We made publicly available ([doi:10.6084/m9.figshare.4902917](https://doi.org/10.6084/m9.figshare.4902917)) the amino acid sequences in **AA-SEQUENCE-DATABASE**.



<div style="display: block; height: 200px;">&nbsp;</div>