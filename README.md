# Read Distribution Toolkit

This is a toolkit for analysing the coverage and distribution of reads over a genome sequence assembly.

The toolkit focusses on calculating coverage inequality statistics from read coverage files, using the Gini coefficient.

## The Gini-coefficient

The Gini coefficient is a metric used to measure the inequality within a dataset.
It is commonly used in economics to measure the distribution of income within a population, where it is represented by a value 
between 0 and 1, with 0 representing perfectly even distribution, and higher values representing higher inequality of distribution. 
This toolkit applies this coefficient to measure inequality of depth of coverage across a genome.

The Gini coefficient is calculated as:

<a href="https://www.codecogs.com/eqnedit.php?latex=G&space;=&space;A/(A&plus;B)" target="_blank"><img src="https://latex.codecogs.com/gif.latex?G&space;=&space;A/(A&plus;B)" title="G = A/(A+B)" /></a>

where 
<a href="https://www.codecogs.com/eqnedit.php?latex=A" target="_blank"><img src="https://latex.codecogs.com/gif.latex?A" title="A" /></a>
is the area under the line of equality, and 
<a href="https://www.codecogs.com/eqnedit.php?latex=B" target="_blank"><img src="https://latex.codecogs.com/gif.latex?B" title="B" /></a>
the area under the Lorenz curve, on the graph of data distribution. Further reading can be found in the references section.

The Gini-coefficient is mathematically equivillent to the sum of the absolute difference of all pairs of the population
(where a pair consists of positions on the Lorenz curve and the line of equality given any x) divided by the mean.
In the context of read coverage analysis, if 
<a href="https://www.codecogs.com/eqnedit.php?latex=x_i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?x_i" title="x_i" /></a>
is the coverage at position 
<a href="https://www.codecogs.com/eqnedit.php?latex=i" target="_blank"><img src="https://latex.codecogs.com/gif.latex?i" title="i" /></a>
within the a genome of size 
<a href="https://www.codecogs.com/eqnedit.php?latex=n" target="_blank"><img src="https://latex.codecogs.com/gif.latex?n" title="n" /></a>
then the mean absolute difference, and therefore Gini coefficient is calculated as:

<a href="https://www.codecogs.com/eqnedit.php?latex=
G&space;=&space;\dfrac{\sum\limits_{i=1}^{n}\sum\limits_{j=1}^{n}|x_i&space;-&space;x_j|}{2n\sum\limits_{i=1}^{n}x_i}"
target="_blank"><img src="https://latex.codecogs.com/gif.latex?G&space;=&space;\dfrac{\sum\limits_{i=1}^{n}\sum\limits_{j=1}^{n}|x_i&space;-&space;x_j|}{2n\sum\limits_{i=1}^{n}x_i}" title="G = \dfrac{\sum\limits_{i=1}^{n}\sum\limits_{j=1}^{n}|x_i - x_j|}{2n\sum\limits_{i=1}^{n}x_i}" /></a>

## Gini-Granularity curves

Gini-Granularity curves are presented as a manner of resolving two problems with the Gini coefficient:

- Two genomes with identical ordered coverage arrays will produce identical Lorentz curves, and therefore an identical Gini. 
This does not take into account the distribution of depth of coverage across the genome.
- The Gini coefficient is known to be confounded by data granularity.

Gini granularity curves are presented here as a more complete characterisation of the distributions of reads across a genome.

To generate a GG-curve, the Gini coefficient is calculated at a range of data granularities. This is achieved by calculating
across windows, where the mean coverage for each window is taken instead of the coverage at every position within the genome.
This has the effect of 'blurring' the coverage across the genome and making the data less granular.

Using this range of Gini values, a curve can be generated. The area under this curve, when normalised, can be used
to indicate the distribution of reads mapping across the genome, where higher values indicate greater levels of read aggregation.

**N.B.** This result must be analysed in context with the Gini value at maximum data granularity (window size 1).

## Usage

This toolkit consists of 3 scripts:\
`gini.py`\
`GG_out_merge.py`\
`GG_curve.py`\
To be used in this order.

`gini.py` takes a cov file in .bed format generated by `Samtools -depth`, structured as\
  `chr   pos   depth`\
and calculates the Gini coefficient of coverage over this genome.

To generate Gini-Granularity curves, the Gini coefficient must be calculated over a range of window sizes using the `-G` argument. For example, to calculate the Gini coefficient of `myGenome.cov` over windows of 1 to 1000 nucelotides, with a step value of 5, the arguments would be:\
`python3 src/gini.py myGenome.cov -G 1 1000 5`\
which would calculate over window sizes of:\
1, 6, 11, 16, ..., 991, 996

To generate the Gini-Granularity curve of this genome, this data should be output to a file, and `GG_curve.py` ran on this file. This will output a graph showing the normalised and non-normalised curves, along with the area under these curves.

To compare the read distribution of multiple genomes, `gini.py` should be ran (as above), followed by using the `GG_out_merge.py` script to merge each of the output files into a .tsv file. This can then the used as input for `GG_curve.py`.

To generate the GG-curves for the dataset `myGenome1.cov`, `myGenome2.cov`, & `myGenome3.cov`, and calculate the GG-auc. We would run:
#
    python3 src/gini.py myGenome1.cov -G 1 1000 5 > myGenome1.GG
    python3 src/gini.py myGenome2.cov -G 1 1000 5 > myGenome2.GG
    python3 src/gini.py myGenome3.cov -G 1 1000 5 > myGenome3.GG
    python3 src/GG_out_merge.py --Gini_files myGenome1.GG myGenome2.GG myGenome3.GG -o allmyGenomes.GG
    python3 src/GG_curve.py allmyGenomes.GG
#
