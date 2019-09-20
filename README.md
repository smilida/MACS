# Recent Changes for MACS (2.1.3.3)

### 2.1.3.3
	* Features added

	1) Support Docker auto-deploy. PR #309

	2) Support Travis CI auto-testing, update unit-testing
	scripts, and enable subcommand testing on small datasets.

	3) Update README documents. #297 PR #306

	4) `cmbreps` supports more than 2 replicates. Merged from PR #304
	@Maarten-vd-Sande and PR #307 (our own chi-sq test code)

	5) `--d-min` option is added in `callpeak` and `predictd`, to
	exclude predictions of fragment size smaller than the given
	value. Merged from PR #267 @shouldsee.

	6) `--buffer-size` option is added in `predictd`, `filterdup`,
	`pileup` and `refinepeak` subcommands. Users can use this option
	to decrease memory usage while there are a large number of contigs
	in the data. Also, now `callpeak`, `predictd`, `filterdup`,
	`pileup` and `refinepeak` will suggest users to tweak
	`--buffer-size` while catching a MemoryError. #313 PR #314

	* Bugs fixed

	1) #265 Fixed a bug where the pseudocount hasn't been applied
	while calculating p-value score in ScoreTrack object.

	2) Fixed bdgbroadcall so that it will report those broad peaks
	without strong peak inside, a consistent behavior as `callpeak
	--broad`.

	3) Rename COPYING to LICENSE.


### 2.1.2

	* New features

	1) Added missing BEDPE support. And enable the support for BAMPE
	and BEDPE formats in 'pileup', 'filterdup' and 'randsample'
	subcommands. When format is BAMPE or BEDPE, The 'pileup' command
	will pile up the whole fragment defined by mapping locations of
	the left end and right end of each read pair. Thank @purcaro

	2) Added options to callpeak command for tweaking max-gap and
	min-len during peak calling. Thank @jsh58!

	3) The callpeak option "--to-large" option is replaced with
	"--scale-to large".

	4) The randsample option "-t" has been replaced with "-i".

	* Bug fixes

	1) Fixed memory issue related to #122 and #146

	2) Fixed a bug caused by a typo. Related to #249, Thank @shengqh

	3) Fixed a bug while setting commandline qvalue cutoff.

	4) Better describe the 5th column of narrowPeak. Thank @alexbarrera

	5) Fixed the calculation of average fragment length for paired-end
	data. Thank @jsh58

	6) Fixed bugs caused by khash while computing p/q-value and log
	likelihood ratios. Thank @jsh58

    7) More spelling tweaks in source code. Thank @mr-c

# README for MACS2 (2.1.3.3)

## Introduction

With the improvement of sequencing techniques, chromatin
immunoprecipitation followed by high throughput sequencing (ChIP-Seq)
is getting popular to study genome-wide protein-DNA interactions. To
address the lack of powerful ChIP-Seq analysis method, we presented a
novel algorithm, named Model-based Analysis of ChIP-Seq (MACS), for
identifying transcript factor binding sites. MACS captures the
influence of genome complexity to evaluate the significance of
enriched ChIP regions, and MACS improves the spatial resolution of
binding sites through combining the information of both sequencing tag
position and orientation. MACS can be easily used for ChIP-Seq data
alone, or with control sample with the increase of specificity.

## Install

Please check the file 'INSTALL' in the distribution.

## Usage

```
macs2 [-h] [--version]
    {callpeak,bdgpeakcall,bdgbroadcall,bdgcmp,bdgopt,cmbreps,bdgdiff,filterdup,predictd,pileup,randsample,refinepeak}
```

Example for regular peak calling: `macs2 callpeak -t ChIP.bam -c Control.bam -f BAM -g hs -n test -B -q 0.01`

Example for broad peak calling: `macs2 callpeak -t ChIP.bam -c Control.bam --broad -g hs --broad-cutoff 0.1`

There are twelve functions available in MAC2S serving as sub-commands.

Subcommand | Description
-----------|----------
`callpeak` |  Main MACS2 Function to call peaksfrom alignment results.
`bdgpeakcall` | Call peaks from bedGraph output. 
`bdgbroadcall` | Call broad peaks from bedGraph output.
`bdgcmp` | Comparing two signal tracks in bedGraph format.
`bdgopt` | Operate the score column of bedGraph file. 
`cmbreps` | Combine BEDGraphs of scores from replicates. 
`bdgdiff` | Differential peak detection based on paired four bedgraph files. 
`filterdup` | Remove duplicate reads, then save in BED/BEDPE format.
`predictd` | Predict d or fragment size from alignment results.
`pileup` | Pileup aligned reads (single end) or fragments (paired-end)
`randsample` | Randomly choose a number/percentage of total reads.
`refinepeak` | Take raw reads alignment, refine peak summits.

We only cover `callpeak` module in this document. Please use `macs2
COMMAND -h` to see the detail description for each option of each
module.

### Call peaks

This is the main function in MACS2. It can be invoked by 'macs2
callpeak' command. If you type this command without parameters, you
will see a full description of commandline options. Here we only list
the essential options.

#### Essential Options

##### `-t/--treatment FILENAME`

This is the only REQUIRED parameter for MACS. File can be in any
supported format specified by --format option. Check --format for
detail. If you have more than one alignment files, you can specify
them as `-t A B C`. MACS will pool up all these files together.

##### `-c/--control`

The control or mock data file. Please follow the same direction as for
-t/--treatment.

##### `-n/--name`

The name string of the experiment. MACS will use this string NAME to
create output files like `NAME_peaks.xls`, `NAME_negative_peaks.xls`,
`NAME_peaks.bed` , `NAME_summits.bed`, `NAME_model.r` and so on. So
please avoid any confliction between these filenames and your
existing files.

##### `--outdir`

MACS2 will save all output files into speficied folder for this
option.

##### `-f/--format FORMAT`

Format of tag file, can be `ELAND`, `BED`, `ELANDMULTI`,
`ELANDEXPORT`, `ELANDMULTIPET` (for pair-end tags), `SAM`, `BAM`,
`BOWTIE`, `BAMPE` or `BEDPE`. Default is `AUTO` which will allow MACS
to decide the format automatically. `AUTO` is also usefule when you
combine different formats of files. Note that MACS can't detect
`BAMPE` or `BEDPE` format with `AUTO`, and you have to implicitly
specify the format for `BAMPE` and `BEDPE`.

Nowadays, the most common formats are BED or BAM/SAM. 

###### BED
The BED format can be found at [UCSC genome browser website](http://genome.ucsc.edu/FAQ/FAQformat#format1).

The essential columns in BED format input are the 1st column
`chromosome name`, the 2nd `start position`, the 3rd `end position`,
and the 6th, `strand`.

Note that, for BED format, the 6th column of strand information is
required by MACS. And please pay attention that the coordinates in BED
format is zero-based and half-open
(http://genome.ucsc.edu/FAQ/FAQtracks#tracks1).

###### BAM/SAM

If the format is BAM/SAM, please check the definition in
(http://samtools.sourceforge.net/samtools.shtml).  If the BAM file is
generated for paired-end data, MACS will only keep the left mate(5'
end) tag. However, when format BAMPE is specified, MACS will use the
real fragments inferred from alignment results for reads pileup.

###### BEDPE or BAMPE

A special mode will be triggered while format is specified as
'BAMPE' or 'BEDPE'. In this way, MACS2 will process the BAM or BED
files as paired-end data. Instead of building bimodal distribution of
plus and minus strand reads to predict fragment size, MACS2  will
use actual insert sizes of pairs of reads to build fragment
pileup.

The BAMPE format is just BAM format containing paired-end alignment
information, such as those from BWA or BOWTIE. 

The BEDPE format is a simplified and more flexible BED format, which
only contains the first three columns defining the chromosome name,
left and right position of the fragment from Paired-end
sequencing. Please note, this is NOT the same format used by BEDTOOLS,
and BEDTOOLS version of BEDPE is actually not in a standard BED
format. You can use MACS2 subcommand `randsample` to convert a BAM
file containing paired-end information to a BEDPE format file:

```
macs2 randsample -i the_BAMPE_file.bam -f BAMPE -p 100 -o the_BEDPE_file.bed
```

##### `-g/--gsize`

PLEASE assign this parameter to fit your needs!

It's the mappable genome size or effective genome size which is
defined as the genome size which can be sequenced. Because of the
repetitive features on the chromsomes, the actual mappable genome size
will be smaller than the original size, about 90% or 70% of the genome
size. The default hs -- 2.7e9 is recommended for UCSC human hg18
assembly. Here are all precompiled parameters for effective genome
size:

 * hs: 2.7e9
 * mm: 1.87e9
 * ce: 9e7
 * dm: 1.2e8

Users may want to use k-mer tools to simulate mapping of Xbps long
reads to target genome, and to find the ideal effective genome
size. However, usually by taking away the simple repeats and Ns from
the total genome, one can get an approximate number of effective
genome size. Slight difference of the number won't cause big
difference of peak calls, because this number is used to estimate a
genome-wide noise level which is usually the least signficant one
compared with the *local biases* modeled by MACS.

##### `-s/--tsize`

The size of sequencing tags. If you don't specify it, MACS will try to
use the first 10 sequences from your input treatment file to determine
the tag size. Specifying it will override the automatically determined
tag size.

##### `-q/--qvalue`

The qvalue (minimum FDR) cutoff to call significant regions. Default
is 0.05. For broad marks, you can try 0.05 as cutoff. Q-values are
calculated from p-values using Benjamini-Hochberg procedure.

##### `-p/--pvalue`

The pvalue cutoff. If -p is specified, MACS2 will use pvalue instead
of qvalue.

##### `--min-length`, `--max-gap`

These two options can be used to fine-tune the peak calling behavior by specifying the minimum length of a called peak and the maximum allowed gap between two nearby regions to be merged. In another word, a called peak has to be longer than *min-length*, and if the distance between two nearby peaks is smaller than *max-gap* then they will be merged as one. If they are not set, MACS2 will set the DEFAULT value for *min-length* as the predicted fragment size d, and the DEFAULT value for *max-gap* as the detected read length. Note, if you set a *min-length* value smaller than the fragment size, it may have NO effect on the result. For BROAD peak calling, try to set a
large value such as 500bps. You can also use '--cutoff-analysis' option with default setting, and check the column 'avelpeak' under different cutoff values to decide a reasonable *min-length* value.

##### `--nolambda`

With this flag on, MACS will use the background lambda as local
lambda. This means MACS will not consider the local bias at peak
candidate regions.

##### `--slocal`, `--llocal`

These two parameters control which two levels of regions will be
checked around the peak regions to calculate the maximum lambda as
local lambda. By default, MACS considers 1000bp for small local
region(`--slocal`), and 10000bps for large local region(`--llocal`)
which captures the bias from a long range effect like an open
chromatin domain. You can tweak these according to your
project. Remember that if the region is set too small, a sharp spike
in the input data may kill the significant peak.

##### `--nomodel`

While on, MACS will bypass building the shifting model.

##### `--extsize`

While `--nomodel` is set, MACS uses this parameter to extend reads in
5'->3' direction to fix-sized fragments. For example, if the size of
binding region for your transcription factor is 200 bp, and you want
to bypass the model building by MACS, this parameter can be set
as 200. This option is only valid when `--nomodel` is set or when MACS
fails to build model and `--fix-bimodal` is on.

##### `--shift`

Note, this is NOT the legacy `--shiftsize` option which is replaced by
`--extsize`! You can set an arbitrary shift in bp here. Please Use
discretion while setting it other than default value (0). When
`--nomodel` is set, MACS will use this value to move cutting ends (5')
then apply `--extsize` from 5' to 3' direction to extend them to
fragments. When this value is negative, ends will be moved toward
3'->5' direction, otherwise 5'->3' direction. Recommended to keep it
as default 0 for ChIP-Seq datasets, or -1 * half of EXTSIZE together
with --extsize option for detecting enriched cutting loci such as
certain DNAseI-Seq datasets. Note, you can't set values other than 0
if format is BAMPE or BEDPE for paired-end data. Default is 0.

Here are some examples for combining `--shift` and `--extsize`:

1. To find enriched cutting sites such as some DNAse-Seq datasets. In
this case, all 5' ends of sequenced reads should be extended in both
direction to smooth the pileup signals. If the wanted smoothing window
is 200bps, then use `--nomodel --shift -100 --extsize 200`.

2. For certain nucleosome-seq data, we need to pileup the centers of
nucleosomes using a half-nucleosome size for wavelet analysis
(e.g. NPS algorithm). Since the DNA wrapped on nucleosome is about
147bps, this option can be used: `--nomodel --shift 37 --extsize 73`.

##### `--keep-dup`

It controls the MACS behavior towards duplicate tags at the exact same
location -- the same coordination and the same strand. The default
'auto' option makes MACS calculate the maximum tags at the exact same
location based on binomal distribution using 1e-5 as pvalue cutoff;
and the 'all' option keeps every tags.  If an integer is given, at
most this number of tags will be kept at the same location. The
default is to keep one tag at the same location. Default: 1

##### `--broad`

When this flag is on, MACS will try to composite broad regions in
BED12 ( a gene-model-like format ) by putting nearby highly enriched
regions into a broad region with loose cutoff. The broad region is
controlled by another cutoff through `--broad-cutoff`. The maximum
length of broad region length is 4 times of d from MACS. DEFAULT:
False

##### `--broad-cutoff`

Cutoff for broad region. This option is not available unless --broad
is set. If -p is set, this is a pvalue cutoff, otherwise, it's a
qvalue cutoff.  DEFAULT: 0.1

##### `--scale-to <large|small>`

When set to "large", linearly scale the smaller dataset to the same
depth as larger dataset. By default or being set as "small", the
larger dataset will be scaled towards the smaller dataset. Beware, to
scale up small data would cause more false positives.

##### `-B/--bdg`

If this flag is on, MACS will store the fragment pileup, control
lambda in bedGraph files. The bedGraph files will be stored in current
directory named `NAME_treat_pileup.bdg` for treatment data,
`NAME_control_lambda.bdg` for local lambda values from control.

##### `--call-summits`

MACS will now reanalyze the shape of signal profile (p or q-score
depending on cutoff setting) to deconvolve subpeaks within each peak
called from general procedure. It's highly recommended to detect
adjacent binding events. While used, the output subpeaks of a big
peak region will have the same peak boundaries, and different scores
and peak summit positions.

##### `--buffer-size`

MACS uses a buffer size for incrementally increasing internal array
size to store reads alignment information for each chromosome or
contig. To increase the buffer size, MACS can run faster but will
waste more memory if certain chromosome/contig only has very few
reads. In most cases, the default value 100000 works fine. However, if
there are large number of chromosomes/contigs in your alignment and
reads per chromosome/contigs are few, it's recommended to specify a
smaller buffer size in order to decrease memory usage (but it will
take longer time to read alignment files). Minimum memory requested
for reading an alignment file is about # of CHROMOSOME * BUFFER_SIZE *
8 Bytes. DEFAULT: 100000

#### Output files

1. `NAME_peaks.xls` is a tabular file which contains information about
   called peaks. You can open it in excel and sort/filter using excel
   functions. Information include:
   
    - chromosome name
    - start position of peak
    - end position of peak
    - length of peak region
    - absolute peak summit position
    - pileup height at peak summit
    - -log10(pvalue) for the peak summit (e.g. pvalue =1e-10, then this value should be 10)
    - fold enrichment for this peak summit against random Poisson distribution with local lambda, 
    - -log10(qvalue) at peak summit
   
   Coordinates in XLS is 1-based which is different with BED format. When `--broad` is enabled for broad peak calling, the pileup, pvalue, qvalue, and fold change in the XLS file will be the mean value across the entire peak region, since peak summit won't be called in broad peak calling mode. 

2. `NAME_peaks.narrowPeak` is BED6+4 format file which contains the
   peak locations together with peak summit, pvalue and qvalue. You
   can load it to UCSC genome browser. Definition of some specific
   columns are: 
   
   - 5th: integer score for display. It's calculated as `int(-10*log10pvalue)` or `int(-10*log10qvalue)` depending on whether `-p` (pvalue) or `-q` (qvalue) is used as score cutoff. Please note that currently this value might be out of the [0-1000] range defined in [UCSC Encode narrowPeak format](https://genome.ucsc.edu/FAQ/FAQformat.html#format12). You can let the value saturated at 1000 (i.e. p/q-value = 10^-100) by using the following 1-liner awk: `awk -v OFS="\t" '{$5=$5>1000?1000:$5} {print}' NAME_peaks.narrowPeak`
   - 7th: fold-change at peak summit
   - 8th: -log10pvalue at peak summit
   - 9th: -log10qvalue at peak summit
   - 10th: relative summit position to peak start
   
   The file can be loaded directly to UCSC genome browser. Remove the beginning track line if you want to
   analyze it by other tools.

3. `NAME_summits.bed` is in BED format, which contains the peak
   summits locations for every peaks. The 5th column in this file is
   the same as NAME_peaks.narrowPeak. If you want to find the motifs
   at the binding sites, this file is recommended. The file can be
   loaded directly to UCSC genome browser. Remove the beginning track
   line if you want to analyze it by other tools.

4. `NAME_peaks.broadPeak` is in BED6+3 format which is similar to
   narrowPeak file, except for missing the 10th column for annotating
   peak summits. This file and the `gappedPeak` file will only be 
   available when `--broad` is enabled. Since in the broad peak calling
   mode, the peak summit won't be called, the values in the 5th, and 
   7-9th columns are the mean value over all positions in the peak region. 

5. `NAME_peaks.gappedPeak` is in BED12+3 format which contains both the
   broad region and narrow peaks. The 5th column is 10*-log10qvalue,
   to be more compatible to show grey levels on UCSC browser. Tht 7th
   is the start of the first narrow peak in the region, and the 8th
   column is the end. The 9th column should be RGB color key, however,
   we keep 0 here to use the default color, so change it if you
   want. The 10th column tells how many blocks including the starting
   1bp and ending 1bp of broad regions. The 11th column shows the
   length of each blocks, and 12th for the starts of each blocks. 13th:
   fold-change, 14th: -log10pvalue, 15th: -log10qvalue. The file can be
   loaded directly to UCSC genome browser. 

6. `NAME_model.r` is an R script which you can use to produce a PDF
   image about the model based on your data. Load it to R by:

   `$ Rscript NAME_model.r`

   Then a pdf file `NAME_model.pdf` will be generated in your current
   directory. Note, R is required to draw this figure.

7. The `NAME_treat_pileup.bdg` and `NAME_control_lambda.bdg` files are
   in bedGraph format which can be imported to UCSC genome browser or
   be converted into even smaller bigWig files. The
   `NAME_treat_pielup.bdg` contains the pileup signals (normalized
   according to `--scale-to` option) from ChIP/treatment sample. The
   `NAME_control_lambda.bdg` contains local biases estimated for each
   genomic locations from control sample, or from treatment sample
   when control sample is absent. The subcommand `bdgcmp` can be used
   to compare these two files and make bedGraph file of scores such as
   p-value, q-value, log likelihood, and log fold changes.

## Other useful links

 * [Cistrome](http://cistrome.org/ap/)
 * [bedTools](http://code.google.com/p/bedtools/)
 * [UCSC toolkits](http://hgdownload.cse.ucsc.edu/admin/exe/)

## Tips of fine-tuning peak calling

There are several subcommands within MACSv2 package to fine-tune or customize your analysis:

1. `bdgcmp` can be used on `*_treat_pileup.bdg` and
   `*_control_lambda.bdg` or bedGraph files from other resources
   to calculate score track.

2. `bdgpeakcall` can be used on `*_treat_pvalue.bdg` or the file
   generated from bdgcmp or bedGraph file from other resources to
   call peaks with given cutoff, maximum-gap between nearby mergable
   peaks and minimum length of peak. bdgbroadcall works similarly to
   bdgpeakcall, however it will output `_broad_peaks.bed` in BED12
   format.

3. Differential calling tool -- `bdgdiff`, can be used on 4 bedgraph
   files which are scores between treatment 1 and control 1,
   treatment 2 and control 2, treatment 1 and treatment 2, treatment
   2 and treatment 1. It will output the consistent and unique sites
   according to parameter settings for minimum length, maximum gap
   and cutoff.

4. You can combine subcommands to do a step-by-step peak
   calling. Read detail at [MACS2 wikipage](https://github.com/taoliu/MACS/wiki/Advanced%3A-Call-peaks-using-MACS2-subcommands)
