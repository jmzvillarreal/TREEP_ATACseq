# Hands-on session ATACseq

### IN ORDER TO RUN THE EXERCISE IN CLASS WE ARE GOING TO USE A SUBSET OF THE INITIAL FILES

1️⃣ Install conda environment:

```bash
conda env create -f ATACseq.yml
```
2️⃣ Activate and check hands-on session environment:

```bash
conda activate ATACseq
conda list
```
3️⃣ TOBIAS analysis 1.Tn5 bias correction

```bash
TOBIAS ATACorrect --bam cDC1_chr9_subset.bam --genome chr9.fa --peaks cDC1_chr9_peaks.narrowPeak --blacklist mm9_chr9-blacklist.bed --outdir ATACorrect_cDC1 --cores 6
TOBIAS ATACorrect --bam pDC_chr9_subset.bam --genome chr9.fa --peaks pDC_chr9_peaks.narrowPeak --blacklist mm9_chr9-blacklist.bed --outdir ATACorrect_pDC --cores 6
```
4️⃣ Create common peak set for both conditions for TOBIAS downstream analysis:

```bash
awk -v OFS="\t" '{print $1,$2,$3}' cDC1_chr9_peaks.narrowPeak > cDC1_chr9_peaks.bed
awk -v OFS="\t" '{print $1,$2,$3}' pDC_chr9_peaks.narrowPeak > pDC_chr9_peaks.bed

cat cDC1_chr9_peaks.bed pDC_chr9_peaks.bed |sort -k1,1 -k2,2n |mergeBed > Dendritic_cells_merged.bed
```
5️⃣ TOBIAS analysis 2. Calculate footprint scores from corrected cutsites on the common regions:

```bash
TOBIAS ScoreBigwig --signal ./ATACorrect_cDC1/cDC1_chr9_subset_corrected.bw --regions Dendritic_cells_merged.bed --output cDC1_footprints.bw --cores 6
TOBIAS ScoreBigwig --signal ./ATACorrect_pDC/pDC_chr9_subset_corrected.bw --regions Dendritic_cells_merged.bed --output pDC_footprints.bw --cores 6
```
6️⃣ TOBIAS analysis 3. Run differential binding from footprints:

```bash
TOBIAS BINDetect --motifs filtered_motifs.meme --signals cDC1_footprints.bw pDC_footprints.bw --genome chr9.fa --peaks Dendritic_cells_merged.bed --outdir BINDetect_cDC1_vs_pDC --cond_names cDC1 pDC --cores 6
```
7️⃣ Visualize the difference in footprints between two conditions exclusively for bound sites:

```bash
TOBIAS PlotAggregate --TFBS BINDetect_cDC1_vs_pDC/Tcf4_M01750_2.00/beds/Tcf4_M01750_2.00_cDC1_bound.bed BINDetect_cDC1_vs_pDC/Tcf4_M01750_2.00/beds/Tcf4_M01750_2.00_pDC_bound.bed --signals ATACorrect_cDC1/cDC1_chr9_subset_corrected.bw ATACorrect_pDC/pDC_chr9_subset_corrected.bw --output Tcf4_footprint_comparison_subsets.png --share_y both --plot_boundaries
```
