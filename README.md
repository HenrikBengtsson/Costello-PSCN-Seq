# CostelloJ-PSCN-Seq - Parent-specific Copy-number Estimation Pipeline using HT-Seq Data


## Requirements

### Required software

This pipeline is implemented in [R] and requires R packages [aroma.seq], [sequenza] (Favero et al. 2015), and [PSCBS] (Bengtsson et al. 2010, Olshen et al. 2011).  To install these packages and all of their dependencies, call the following from R:

```r
> source("https://callr.org/install#sequenza")
> source("https://callr.org/install#HenrikBengtsson/aroma.seq")
```

In addition to the above R dependencies, the pipeline requires that [samtools] (Li et al. 2009) is on the `PATH`.


## Setup (once)

1. Run `Rscript 0.setup.R` once. This will setup links to shared annotation data sets and lab data files on the TIPCC compute cluster.

2. Make sure `./config.yml` is correct.  It specify the default analysis settings.  The individual entries can be overridden by individual command-line options to the below `Rscript` calls.

3. Configure parallel processing following the instructions in Section 'Configure parallel processing' below.


## Data processing

The following scripts should be run in order:

* `Rscript 1.mpileup.R`
* `Rscript 2.sequenza.R`
* `Rscript 3.pscbs.R`
* `Rscript 4.reports.R`

You may want to adjust [`./config.yml`](https://github.com/HenrikBengtsson/Costello-PSCN-Seq/blob/master/config.yml) to process other data sets. Alternatively, you can specify another file that this default via command-line option `--config`, e.g. `Rscript 1.mpileup.R --config=config_set_a.yml`.


### Data processing via scheduler

To process the above four steps via the Torque/PBS scheduler, use:

```sh
$ qsub -d $(pwd) 1-4.submit_all.pbs
```

This will in turn _submit_ the corresponding PBS scripts `1.mpileup.pbs`, `2.sequenza.pbs`, `3.pscbs.pbs`, and `4.reports.pbs` to the scheduler.  Those PBS scripts "freeze" software versions to R 3.5.2 and samtools 1.3.1.



## Configure parallel processing

The pipeline supports both sequential and parallel processing on a large number of backends and compute resources.  By default the pipeline is configured to process the data sequentially on the current machine, but this can easily be changed to run in parallel, say, on a compute cluster.  In order not to clutter up the analysis scripts, these settings are preferably done in a separate `.future.R` (loaded automatically by the [future] framework) in the project root directory.

To process data via a TORQUE / PBS job scheduler using the [future.batchtools] package, try with the configuration that we use for our UCSF TIPCC cluster;

```sh
# Copy to project directory
$ cp .future-configs/batchtools/.future.R .

# Copy to project directory
$ cp .future-configs/batchtools/batchtools.torque.tmpl .

# Install the future.batchtools package
$ Rscript -e "install.packages('future.batchtools')"
```

These should be generic enough to also run on other TORQUE / PBS systems.  If not, see the [batchtools] package for how to configure the template file.

You can verify that it works by trying the following in the project directory:

```r
> library("future")
Using future plan:
plan(list(samples = tweak(batchtools_torque, label = "sample", 
    resources = list(vmem = "4gb")), chromosomes = tweak(batchtools_torque, 
    label = "chr", resources = list(vmem = "8gb"))))
```

This confirms that as soon as the [future] package is loaded, it will source the `.future.R` script which in turn will setup the parallel settings.  It is `.future.R` that reports on the future plan used.

Next, we can try to submit a job to the scheduler using these settings by:

```r
> x %<-% Sys.info()[["nodename"]]
```

In this step, future.batchtools will import the `batchtools.torque.tmpl` file in that we copied to project working directory.  If it fails to locate that file, there will be an error.  If it succeeds, a batchtools job will be submitted to the job scheduler - which can be seen when if calling `qstat -u $USER` in another shell.

Finally, if we try to look at the value of `x`;

```r
> x
[1] "n17"
> 
```

it will block until the job is finished and then its value will be printed. Here we see that the job was running on compute node n17.



## References

* Bengtsson H, Neuvial P, Speed TP. TumorBoost: Normalization of allele-specific tumor copy numbers from a single pair of tumor-normal genotyping microarrays, BMC Bioinformatics, 2010. DOI: [10.1186/1471-2105-11-245](https://doi.org/10.1186%2F1471-2105-11-245). PMID: [20462408](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/elink.fcgi?dbfrom=pubmed&cmd=prlinks&retmode=ref&id=20462408), PMCID: [PMC2894037](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2894037/)

* Olshen AB, Bengtsson H, Neuvial P, Spellman PT, Olshen RA, Seshan VA. Parent-specific copy number in paired tumor-normal studies using circular binary segmentation, Bioinformatics, 2011. DOI: [10.1093/bioinformatics/btr329](https://doi.org/10.1093%2Fbioinformatics%2Fbtr329). PMID: [21666266](https://eutils.ncbi.nlm.nih.gov/entrez/eutils/elink.fcgi?dbfrom=pubmed&cmd=prlinks&retmode=ref&id=21666266). PMCID: [PMC3137217](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3137217/)

* Favero F, Joshi T, Marquard AM, Birkbak NJ, Krzystanek M, Li Q, Szallasi Z and Eklund AC. Sequenza: allele-specific copy number and mutation profiles from tumor sequencing data, Annals of Oncology, 2015. DOI: [10.1093/annonc/mdu479](https://dx.doi.org/10.1093%2Fannonc%2Fmdu479), PMID: [25319062](https://www.ncbi.nlm.nih.gov/pubmed/25319062), PMCID: [PMC4269342](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4269342/)

* Li H, Handsaker B, Wysoker A, Fennell T, Ruan J, Homer N, Marth G, Abecasis G, Durbin R, and 1000 Genome Project Data Processing Subgroup, The Sequence alignment/map (SAM) format and SAMtools. Bioinformatics, 2009. DOI: [10.1093/bioinformatics/btp352](https://dx.doi.org/10.1093%2Fbioinformatics%2Fbtp352), PMID: [19505943](https://www.ncbi.nlm.nih.gov/pubmed/19505943), PMCID: [PMC2723002](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2723002/)

[R]: https://www.r-project.org/
[samtools]: https://www.htslib.org/
[aroma.seq]: https://github.com/HenrikBengtsson/aroma.seq/
[sequenza]: https://cran.r-project.org/package=sequenza
[batchtools]: https://cran.r-project.org/package=batchtools
[future]: https://cran.r-project.org/package=future
[PSCBS]: https://cran.r-project.org/package=PSCBS
[future.batchtools]: https://cran.r-project.org/package=future.batchtools
