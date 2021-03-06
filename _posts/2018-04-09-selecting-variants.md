---
layout: post
title: Selecting variants and samples
---


A couple of people have recently asked me about how to use [scikit-allel](http://scikit-allel.readthedocs.org) to select data from a variant call set for a particular set of variants. This could be variants for a specific genome region (e.g., a gene), or variants matching a particular set of filters. This notebook gives a couple of examples, using data from the (human) 1000 genomes project phase 3.

*Update 2018-04-19: Added a sub-section on selecting samples.*

Here's the Python packages we'll need. If you have earlier versions, you'll need to upgrade.


{% highlight python %}
import allel
allel.__version__
{% endhighlight %}




    '1.1.10'




{% highlight python %}
import zarr
zarr.__version__
{% endhighlight %}




    '2.2.0'




{% highlight python %}
import numcodecs
numcodecs.__version__
{% endhighlight %}




    '0.5.5'




{% highlight python %}
import numpy as np
np.__version__
{% endhighlight %}




    '1.13.3'




{% highlight python %}
# other imports
import sys
{% endhighlight %}

## Extract data from a VCF

The source data comes as a VCF file, which I've downloaded to the local disk:


{% highlight python %}
vcf_path = 'data/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz'
!ls -lh {vcf_path}
{% endhighlight %}

    -rw-r--r-- 1 aliman aliman 205M Jun 20  2017 data/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz


I'm going to use data from chromosome 22 only for illustration. I'm also going to extract the data and convert to Zarr format, which will make life easier downstream. To do the conversion I'm going to use the [vcf_to_zarr()](http://scikit-allel.readthedocs.io/en/latest/io.html#allel.vcf_to_zarr) function from scikit-allel. This is a one-off operation, once the data have been converted to Zarr they can be loaded directly from Zarr the next time you want to do some analysis. The conversion takes about 3 minutes on my computer.


{% highlight python %}
zarr_path = 'data/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.zarr'
{% endhighlight %}


{% highlight python %}
allel.vcf_to_zarr(vcf_path, zarr_path, group='22', fields='*', log=sys.stdout, overwrite=True)
{% endhighlight %}

    [vcf_to_zarr] 65536 rows in 11.83s; chunk in 11.83s (5539 rows/s); 22 :18539397
    [vcf_to_zarr] 131072 rows in 25.01s; chunk in 13.18s (4972 rows/s); 22 :21016127
    [vcf_to_zarr] 196608 rows in 38.05s; chunk in 13.04s (5023 rows/s); 22 :23236362
    [vcf_to_zarr] 262144 rows in 49.28s; chunk in 11.22s (5838 rows/s); 22 :25227844
    [vcf_to_zarr] 327680 rows in 62.99s; chunk in 13.71s (4780 rows/s); 22 :27285434
    [vcf_to_zarr] 393216 rows in 75.12s; chunk in 12.14s (5399 rows/s); 22 :29572822
    [vcf_to_zarr] 458752 rows in 87.23s; chunk in 12.11s (5411 rows/s); 22 :31900536
    [vcf_to_zarr] 524288 rows in 99.03s; chunk in 11.80s (5554 rows/s); 22 :34069864
    [vcf_to_zarr] 589824 rows in 112.06s; chunk in 13.03s (5028 rows/s); 22 :36053392
    [vcf_to_zarr] 655360 rows in 124.48s; chunk in 12.41s (5279 rows/s); 22 :38088395
    [vcf_to_zarr] 720896 rows in 137.07s; chunk in 12.60s (5203 rows/s); 22 :40216200
    [vcf_to_zarr] 786432 rows in 147.83s; chunk in 10.76s (6092 rows/s); 22 :42597446
    [vcf_to_zarr] 851968 rows in 161.38s; chunk in 13.55s (4835 rows/s); 22 :44564263
    [vcf_to_zarr] 917504 rows in 173.08s; chunk in 11.70s (5601 rows/s); 22 :46390672
    [vcf_to_zarr] 983040 rows in 184.95s; chunk in 11.87s (5522 rows/s); 22 :48116697
    [vcf_to_zarr] 1048576 rows in 199.16s; chunk in 14.21s (4611 rows/s); 22 :49713436
    [vcf_to_zarr] 1103547 rows in 209.81s; chunk in 10.65s (5160 rows/s)
    [vcf_to_zarr] all done (5252 rows/s)


Let's open the Zarr data and do a little bit of poking around to see what's there.


{% highlight python %}
callset = zarr.open_group(zarr_path, mode='r')
callset.tree(expand=True)
{% endhighlight %}




<link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/jstree/3.3.3/themes/default/style.min.css"/><div id="77f95fc2-bfa1-4907-bd69-7bfb2b7c88c0" class="zarr-tree"><ul><li data-jstree='{"type": "Group"}' class='jstree-open'><span>/</span><ul><li data-jstree='{"type": "Group"}' class='jstree-open'><span>22</span><ul><li data-jstree='{"type": "Group"}' class='jstree-open'><span>calldata</span><ul><li data-jstree='{"type": "Array"}' class='jstree-open'><span>GT (1103547, 2504, 2) int8</span></li></ul></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>samples (2504,) object</span></li><li data-jstree='{"type": "Group"}' class='jstree-open'><span>variants</span><ul><li data-jstree='{"type": "Array"}' class='jstree-open'><span>AA (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>AC (1103547, 3) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>AF (1103547, 3) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>AFR_AF (1103547, 3) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>ALT (1103547, 3) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>AMR_AF (1103547, 3) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>AN (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>CHROM (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>CIEND (1103547, 2) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>CIPOS (1103547, 2) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>CS (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>DP (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>EAS_AF (1103547, 3) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>END (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>EUR_AF (1103547, 3) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>EX_TARGET (1103547,) bool</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>FILTER_PASS (1103547,) bool</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>ID (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>IMPRECISE (1103547,) bool</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>MC (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>MEINFO (1103547, 4) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>MEND (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>MLEN (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>MSTART (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>MULTI_ALLELIC (1103547,) bool</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>NS (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>POS (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>QUAL (1103547,) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>REF (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>SAS_AF (1103547, 3) float32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>SVLEN (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>SVTYPE (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>TSD (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>VT (1103547,) object</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>is_snp (1103547,) bool</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>numalt (1103547,) int32</span></li><li data-jstree='{"type": "Array"}' class='jstree-open'><span>svlen (1103547, 3) int32</span></li></ul></li></ul></li></ul></li></ul></div>
<script>
    if (!require.defined('jquery')) {
        require.config({
            paths: {
                jquery: '//cdnjs.cloudflare.com/ajax/libs/jquery/1.12.1/jquery.min'
            },
        });
    }
    if (!require.defined('jstree')) {
        require.config({
            paths: {
                jstree: '//cdnjs.cloudflare.com/ajax/libs/jstree/3.3.3/jstree.min'
            },
        });
    }
    require(['jstree'], function() {
        $('#77f95fc2-bfa1-4907-bd69-7bfb2b7c88c0').jstree({
            types: {
                Group: {
                    icon: "fa fa-folder"
                },
                Array: {
                    icon: "fa fa-table"
                }
            },
            plugins: ["types"]
        });
    });
</script>




The `tree()` method shows us how the data are organised hierarchically within the Zarr store. Hopefully you can see that there is a root group indicated by a slash ('/'), then below that there is a group named '22' containing all of the data from Chromosome 22, then there is a group called 'calldata' and another called 'variants'. 

Within the 'calldata' group there is an array named 'GT' which has the genotypes.

Within the 'variants' group there are arrays named 'CHROM', 'POS', 'DP', etc., containing information about the variants that have been called.

Let's get some diagnostics on the Zarr genotype array.


{% highlight python %}
gt_zarr = callset['22/calldata/GT']
gt_zarr.info
{% endhighlight %}




<table class="zarr-info"><tbody><tr><th style="text-align: left">Name</th><td style="text-align: left">/22/calldata/GT</td></tr><tr><th style="text-align: left">Type</th><td style="text-align: left">zarr.core.Array</td></tr><tr><th style="text-align: left">Data type</th><td style="text-align: left">int8</td></tr><tr><th style="text-align: left">Shape</th><td style="text-align: left">(1103547, 2504, 2)</td></tr><tr><th style="text-align: left">Chunk shape</th><td style="text-align: left">(65536, 64, 2)</td></tr><tr><th style="text-align: left">Order</th><td style="text-align: left">C</td></tr><tr><th style="text-align: left">Read-only</th><td style="text-align: left">True</td></tr><tr><th style="text-align: left">Compressor</th><td style="text-align: left">Blosc(cname='lz4', clevel=5, shuffle=SHUFFLE, blocksize=0)</td></tr><tr><th style="text-align: left">Store type</th><td style="text-align: left">zarr.storage.DirectoryStore</td></tr><tr><th style="text-align: left">No. bytes</th><td style="text-align: left">5526563376 (5.1G)</td></tr><tr><th style="text-align: left">No. bytes stored</th><td style="text-align: left">293489697 (279.9M)</td></tr><tr><th style="text-align: left">Storage ratio</th><td style="text-align: left">18.8</td></tr><tr><th style="text-align: left">Chunks initialized</th><td style="text-align: left">680/680</td></tr></tbody></table>



This tells us the uncompressed size of the genotype array is 5.1 gigabytes. The actual size on disk is much smaller (279.9 megabytes) because the data are compressed. 

## Loading data for a gene

If you want to work with data from a single gene, or any other contiguous region within a chromosome, here's how to do it.

First you need the positions of the variants, wrapped as a scikit-allel ``SortedIndex``:


{% highlight python %}
pos = allel.SortedIndex(callset['22/variants/POS'])
pos
{% endhighlight %}




<div class="allel allel-DisplayAs1D"><span>&lt;SortedIndex shape=(1103547,) dtype=int32&gt;</span><table><thead><tr><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">1103542</th><th style="text-align: center">1103543</th><th style="text-align: center">1103544</th><th style="text-align: center">1103545</th><th style="text-align: center">1103546</th></tr></thead><tbody><tr><td style="text-align: center">16050075</td><td style="text-align: center">16050115</td><td style="text-align: center">16050213</td><td style="text-align: center">16050319</td><td style="text-align: center">16050527</td><td style="text-align: center">...</td><td style="text-align: center">51241342</td><td style="text-align: center">51241386</td><td style="text-align: center">51244163</td><td style="text-align: center">51244205</td><td style="text-align: center">51244237</td></tr></tbody></table></div>



The numbers at the top are the variant indices (starting from 0). The numbers at the bottom are the variant positions as genomic coordinates (i.e., number of base pairs from the chromosome start, starting from 1).

To extract data for a chromosome region, you can use the `pos` object to translate genomic coordinates into variant indices. For example, if you want to locate data for the region from position 20,000,000 to 20,100,000, you can do this:


{% highlight python %}
loc_region = pos.locate_range(20000000, 20100000)
loc_region
{% endhighlight %}




    slice(108029, 111127, None)



The `loc_region` object is a slice, which is simply a way of storing a start and stop index. Here the start index is 108,029 and the stop index is 111,127. So, we need data starting from the 108,030th variant up to but not including the 111,128th variant (remembering that indices start from zero).

We can now use this slice to extract genotypes for our genome region of interest:


{% highlight python %}
gt_region = allel.GenotypeArray(gt_zarr[loc_region])
gt_region
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeArray shape=(3098, 2504, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">2499</th><th style="text-align: center">2500</th><th style="text-align: center">2501</th><th style="text-align: center">2502</th><th style="text-align: center">2503</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">3095</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">3096</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">3097</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr></tbody></table></div>



Note that `gt_region` is a genotype array with 3,098 variants and 2,504 samples.

Breaking this down, the ``gt_zarr[loc_region]`` code loads data for the requested slice of the genotype data from disk into memory as a numpy array. For convenience, I have wrapped this numpy array using the ``allel.GenotypeArray`` class, which provides a nicer visual representation of the data and gives some useful methods.

## Filtering variants

Filtering variants is a very common task. Each analysis typically needs a different set of variants to work with. For example, you may need to filter variants based on some metrics of quality, or on other conditions like allele frequency.

When filtering variants, you first need to identify which variants you require. To help with this, I'm first going to load up some variant information.

The 'MULTI_ALLELIC' array in this callset is a Boolean array indicating whether a variants is multi-allelic (has more than one alternate allele) or not. The following code loads this array from disk into memory:


{% highlight python %}
multi_allelic = callset['22/variants/MULTI_ALLELIC'][:]
multi_allelic
{% endhighlight %}




    array([False, False, False, ..., False, False, False], dtype=bool)



The 'AFR_AF' array in this callset has alternate allele frequencies for the African samples within the cohort:


{% highlight python %}
afr_af = callset['22/variants/AFR_AF'][:]
afr_af
{% endhighlight %}




    array([[ 0.    ,     nan,     nan],
           [ 0.0234,     nan,     nan],
           [ 0.0272,     nan,     nan],
           ..., 
           [ 0.    ,     nan,     nan],
           [ 0.    ,     nan,     nan],
           [ 0.    ,     nan,     nan]], dtype=float32)



Here the `afr_af` array has multiple columns, one for each alternate allele. 

Let's locate variants that are not multi-allelic and have an African allele frequency above 5%:


{% highlight python %}
loc_variant_selection = ~multi_allelic & (afr_af[:, 0] > 0.05)
loc_variant_selection
{% endhighlight %}




    array([False, False, False, ..., False, False, False], dtype=bool)



How many variants match our query?


{% highlight python %}
np.count_nonzero(loc_variant_selection)
{% endhighlight %}




    138275



Now, to extract genotype data for these variants, there are a couple of ways to do it.

If the full genotype array is not too big, you can start by loading the whole lot into memory. In this case, the genotype array is 5.1G, and I have enough RAM on my laptop to handle that, so let's do it:


{% highlight python %}
gt = allel.GenotypeArray(gt_zarr)
gt
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeArray shape=(1103547, 2504, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">2499</th><th style="text-align: center">2500</th><th style="text-align: center">2501</th><th style="text-align: center">2502</th><th style="text-align: center">2503</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1103544</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1103545</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1103546</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr></tbody></table></div>



Now to extract genotypes for the selection, you can use the `compress()` method, e.g.:


{% highlight python %}
gt_variant_selection = gt.compress(loc_variant_selection, axis=0)
gt_variant_selection
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeArray shape=(138275, 2504, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">2499</th><th style="text-align: center">2500</th><th style="text-align: center">2501</th><th style="text-align: center">2502</th><th style="text-align: center">2503</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">1/0</td><td style="text-align: center">0/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/0</td><td style="text-align: center">1/0</td><td style="text-align: center">...</td><td style="text-align: center">1/1</td><td style="text-align: center">0/1</td><td style="text-align: center">1/1</td><td style="text-align: center">0/0</td><td style="text-align: center">1/1</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138272</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138273</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138274</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">1/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr></tbody></table></div>



Alternatively, if your data are larger and/or your computer doesn't have much RAM, there is another way to do this. This makes use of a Python package called Dask, which allows you to run computations without loading all data into memory. To use Dask, we can start by wrapping the full genotype array with the `allel.GenotypeDaskArray` class:


{% highlight python %}
gt_dask = allel.GenotypeDaskArray(gt_zarr)
gt_dask
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeDaskArray shape=(1103547, 2504, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">2499</th><th style="text-align: center">2500</th><th style="text-align: center">2501</th><th style="text-align: center">2502</th><th style="text-align: center">2503</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1103544</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1103545</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1103546</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr></tbody></table></div>



Now we can apply the selection, using almost the same syntax, except that when working via Dask we need to call the `compute()` method to get the final result:


{% highlight python %}
gt_variant_selection = gt_dask.compress(loc_variant_selection, axis=0).compute()
gt_variant_selection
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeArray shape=(138275, 2504, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">2499</th><th style="text-align: center">2500</th><th style="text-align: center">2501</th><th style="text-align: center">2502</th><th style="text-align: center">2503</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">1/0</td><td style="text-align: center">0/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/0</td><td style="text-align: center">1/0</td><td style="text-align: center">...</td><td style="text-align: center">1/1</td><td style="text-align: center">0/1</td><td style="text-align: center">1/1</td><td style="text-align: center">0/0</td><td style="text-align: center">1/1</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138272</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138273</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138274</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">1/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr></tbody></table></div>



## Selecting samples

Another common requirement is selecting data for a specific subset of samples. For example, we might want to select samples from a specific population or set of populations, or we might want to exclude samples with poor data.

Here's an example of selecting samples from a specific population. To do this we first need to know which population the samples belong to. For the 1000 genomes phase 3 dataset, there is a tab-delimited file with this information, which I've downloaded to my computer: 


{% highlight python %}
panel_path = 'data/integrated_call_samples_v3.20130502.ALL.panel'
!head {panel_path}
{% endhighlight %}

    sample	pop	super_pop	gender		
    HG00096	GBR	EUR	male
    HG00097	GBR	EUR	female
    HG00099	GBR	EUR	female
    HG00100	GBR	EUR	female
    HG00101	GBR	EUR	male
    HG00102	GBR	EUR	female
    HG00103	GBR	EUR	male
    HG00105	GBR	EUR	male
    HG00106	GBR	EUR	female


Let's load this into a pandas DataFrame:


{% highlight python %}
import pandas
{% endhighlight %}


{% highlight python %}
panel = pandas.read_csv(panel_path, sep='\t', usecols=['sample', 'pop', 'super_pop'])
panel.head()
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sample</th>
      <th>pop</th>
      <th>super_pop</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HG00096</td>
      <td>GBR</td>
      <td>EUR</td>
    </tr>
    <tr>
      <th>1</th>
      <td>HG00097</td>
      <td>GBR</td>
      <td>EUR</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HG00099</td>
      <td>GBR</td>
      <td>EUR</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HG00100</td>
      <td>GBR</td>
      <td>EUR</td>
    </tr>
    <tr>
      <th>4</th>
      <td>HG00101</td>
      <td>GBR</td>
      <td>EUR</td>
    </tr>
  </tbody>
</table>
</div>



Out of interest, how many samples are there from each population?


{% highlight python %}
panel.groupby(by=('super_pop', 'pop')).count()
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>sample</th>
    </tr>
    <tr>
      <th>super_pop</th>
      <th>pop</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="7" valign="top">AFR</th>
      <th>ACB</th>
      <td>96</td>
    </tr>
    <tr>
      <th>ASW</th>
      <td>61</td>
    </tr>
    <tr>
      <th>ESN</th>
      <td>99</td>
    </tr>
    <tr>
      <th>GWD</th>
      <td>113</td>
    </tr>
    <tr>
      <th>LWK</th>
      <td>99</td>
    </tr>
    <tr>
      <th>MSL</th>
      <td>85</td>
    </tr>
    <tr>
      <th>YRI</th>
      <td>108</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">AMR</th>
      <th>CLM</th>
      <td>94</td>
    </tr>
    <tr>
      <th>MXL</th>
      <td>64</td>
    </tr>
    <tr>
      <th>PEL</th>
      <td>85</td>
    </tr>
    <tr>
      <th>PUR</th>
      <td>104</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">EAS</th>
      <th>CDX</th>
      <td>93</td>
    </tr>
    <tr>
      <th>CHB</th>
      <td>103</td>
    </tr>
    <tr>
      <th>CHS</th>
      <td>105</td>
    </tr>
    <tr>
      <th>JPT</th>
      <td>104</td>
    </tr>
    <tr>
      <th>KHV</th>
      <td>99</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">EUR</th>
      <th>CEU</th>
      <td>99</td>
    </tr>
    <tr>
      <th>FIN</th>
      <td>99</td>
    </tr>
    <tr>
      <th>GBR</th>
      <td>91</td>
    </tr>
    <tr>
      <th>IBS</th>
      <td>107</td>
    </tr>
    <tr>
      <th>TSI</th>
      <td>107</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">SAS</th>
      <th>BEB</th>
      <td>86</td>
    </tr>
    <tr>
      <th>GIH</th>
      <td>103</td>
    </tr>
    <tr>
      <th>ITU</th>
      <td>102</td>
    </tr>
    <tr>
      <th>PJL</th>
      <td>96</td>
    </tr>
    <tr>
      <th>STU</th>
      <td>102</td>
    </tr>
  </tbody>
</table>
</div>



Before we can use this information to select samples from the genotype data, we need to match up the order of samples between this panel file and the callset.  

Here's the sample IDs in the order they appeared in the original VCF, and thus in the order that corresponds to columns in the genotype array:


{% highlight python %}
samples = callset['22/samples'][:]
samples
{% endhighlight %}




    array(['HG00096', 'HG00097', 'HG00099', ..., 'NA21142', 'NA21143',
           'NA21144'], dtype=object)



Are they in the same order as given in the panel file?


{% highlight python %}
np.all(samples == panel['sample'].values)
{% endhighlight %}




    True



This is the ideal situation, because samples are given in the same order in the panel file and in the original VCF. This might not be the case with your dataset, however, and so it's important to check before going any further. If data are not in the same order, you can add a column to the dataframe with the index of each sample as it appears in the callset, e.g.:


{% highlight python %}
samples_list = list(samples)
samples_callset_index = [samples_list.index(s) for s in panel['sample']]
panel['callset_index'] = samples_callset_index
panel.head()
{% endhighlight %}




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>sample</th>
      <th>pop</th>
      <th>super_pop</th>
      <th>callset_index</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>HG00096</td>
      <td>GBR</td>
      <td>EUR</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>HG00097</td>
      <td>GBR</td>
      <td>EUR</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HG00099</td>
      <td>GBR</td>
      <td>EUR</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>HG00100</td>
      <td>GBR</td>
      <td>EUR</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>HG00101</td>
      <td>GBR</td>
      <td>EUR</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



Here you can see that the values in the 'callset_index' column are the same as the values in the dataframe index (first column) because the samples in the panel file and the callset are ordered the same. 

Now, we can obtain the indices of the samples within a particular population. Let's locate all of the African samples:


{% highlight python %}
loc_samples_afr = panel[panel.super_pop == 'AFR'].callset_index.values
loc_samples_afr
{% endhighlight %}




    array([ 673,  674,  675,  676,  677,  678,  679,  680,  683,  684,  685,
            686,  687,  712,  713,  727,  728,  729,  730,  731,  739,  740,
            741,  742,  743,  761,  762,  763,  764,  788,  792,  793,  794,
            812,  813,  854,  855,  867,  868,  869,  870,  879,  880,  881,
            883,  884,  885,  886,  887,  888,  889,  890,  891,  892,  893,
            894,  895,  933,  934,  936,  937,  938,  939,  940,  941,  942,
            943,  944,  945,  946,  947,  948,  949,  950,  951,  952,  953,
            954,  955,  956,  957,  962,  963,  964,  965,  966,  967,  968,
            973,  974,  975,  976,  977,  978,  979,  980,  981,  982,  983,
            984,  985,  986,  987,  988,  989,  990,  991,  992,  993,  994,
            995,  996,  997,  998,  999, 1005, 1006, 1007, 1008, 1009, 1010,
           1011, 1012, 1013, 1014, 1015, 1016, 1017, 1018, 1019, 1020, 1031,
           1032, 1033, 1034, 1035, 1036, 1050, 1051, 1052, 1053, 1054, 1055,
           1065, 1066, 1067, 1068, 1069, 1070, 1071, 1072, 1073, 1086, 1087,
           1088, 1089, 1090, 1091, 1092, 1093, 1094, 1095, 1096, 1097, 1098,
           1099, 1100, 1101, 1102, 1103, 1104, 1105, 1106, 1107, 1108, 1109,
           1110, 1111, 1112, 1113, 1114, 1115, 1116, 1117, 1118, 1119, 1120,
           1121, 1122, 1123, 1124, 1125, 1126, 1127, 1128, 1129, 1130, 1131,
           1132, 1133, 1134, 1135, 1136, 1137, 1138, 1139, 1140, 1141, 1142,
           1143, 1154, 1155, 1156, 1157, 1158, 1159, 1160, 1161, 1162, 1163,
           1164, 1165, 1166, 1167, 1168, 1169, 1170, 1171, 1172, 1173, 1174,
           1175, 1176, 1177, 1178, 1179, 1180, 1181, 1182, 1183, 1184, 1185,
           1186, 1187, 1188, 1189, 1190, 1191, 1192, 1193, 1194, 1195, 1196,
           1197, 1198, 1199, 1200, 1201, 1202, 1203, 1204, 1205, 1206, 1207,
           1208, 1209, 1210, 1211, 1212, 1213, 1214, 1215, 1216, 1217, 1218,
           1219, 1220, 1221, 1222, 1223, 1224, 1225, 1226, 1227, 1228, 1229,
           1230, 1231, 1232, 1233, 1234, 1235, 1236, 1237, 1244, 1245, 1246,
           1247, 1248, 1249, 1250, 1251, 1252, 1253, 1254, 1255, 1256, 1257,
           1258, 1259, 1260, 1261, 1262, 1263, 1264, 1265, 1266, 1267, 1268,
           1269, 1270, 1271, 1272, 1273, 1274, 1275, 1276, 1277, 1278, 1279,
           1280, 1281, 1282, 1283, 1284, 1285, 1286, 1287, 1288, 1289, 1290,
           1291, 1292, 1293, 1294, 1295, 1296, 1297, 1298, 1299, 1300, 1301,
           1302, 1303, 1304, 1305, 1306, 1307, 1308, 1309, 1310, 1311, 1312,
           1313, 1314, 1315, 1316, 1317, 1321, 1322, 1323, 1324, 1325, 1326,
           1327, 1328, 1329, 1330, 1331, 1332, 1333, 1334, 1335, 1336, 1337,
           1338, 1339, 1340, 1341, 1342, 1343, 1344, 1345, 1755, 1756, 1757,
           1758, 1759, 1760, 1761, 1762, 1763, 1764, 1765, 1766, 1767, 1768,
           1769, 1770, 1771, 1772, 1773, 1877, 1878, 1879, 1880, 1881, 1882,
           1883, 1884, 1885, 1886, 1887, 1888, 1889, 1890, 1891, 1892, 1893,
           1894, 1895, 1896, 1897, 1898, 1899, 1900, 1901, 1902, 1903, 1904,
           1905, 1976, 1977, 1978, 1979, 1980, 1981, 1982, 1983, 1984, 1985,
           1986, 1987, 1988, 1989, 1990, 1991, 1992, 1993, 2028, 2029, 2030,
           2031, 2032, 2033, 2034, 2035, 2036, 2037, 2038, 2039, 2040, 2041,
           2042, 2043, 2044, 2045, 2046, 2047, 2048, 2049, 2050, 2051, 2052,
           2053, 2054, 2055, 2056, 2057, 2058, 2059, 2060, 2061, 2062, 2063,
           2064, 2065, 2066, 2067, 2068, 2069, 2070, 2071, 2072, 2073, 2074,
           2075, 2076, 2077, 2078, 2079, 2080, 2081, 2082, 2083, 2084, 2085,
           2086, 2087, 2088, 2089, 2090, 2091, 2092, 2093, 2094, 2095, 2096,
           2097, 2098, 2099, 2100, 2101, 2102, 2103, 2104, 2105, 2106, 2107,
           2108, 2109, 2110, 2111, 2112, 2113, 2114, 2115, 2116, 2117, 2118,
           2119, 2120, 2121, 2122, 2123, 2124, 2125, 2126, 2127, 2128, 2129,
           2130, 2131, 2132, 2133, 2134, 2135, 2136, 2137, 2138, 2139, 2140,
           2141, 2142, 2143, 2144, 2145, 2146, 2147, 2148, 2149, 2150, 2151,
           2152, 2153, 2154, 2155, 2156, 2157, 2158, 2159, 2160, 2161, 2162,
           2163, 2164, 2165, 2166, 2167, 2168, 2169, 2189, 2190, 2191, 2192,
           2193, 2194, 2195, 2196, 2242, 2243, 2244, 2245, 2246, 2247, 2248,
           2249, 2250, 2251, 2252, 2253, 2254, 2255, 2256, 2257, 2258, 2259,
           2260, 2261, 2262, 2263, 2264, 2265, 2266, 2267, 2268, 2269, 2270,
           2271, 2272, 2273, 2274, 2275, 2276, 2277, 2278, 2279, 2280, 2281,
           2282, 2283, 2284, 2285, 2286, 2287, 2288, 2289, 2290, 2291, 2292,
           2293])



So ``loc_samples_afr`` is an array of sample indices. How many samples?


{% highlight python %}
len(loc_samples_afr)
{% endhighlight %}




    661



We can use this to extract columns from the genotype data. E.g., let's apply this selection to the genotype array we previously obtained after selecting variants:


{% highlight python %}
gt_afr = gt_variant_selection.take(loc_samples_afr, axis=1)
gt_afr
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeArray shape=(138275, 661, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">656</th><th style="text-align: center">657</th><th style="text-align: center">658</th><th style="text-align: center">659</th><th style="text-align: center">660</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/1</td><td style="text-align: center">1/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138272</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138273</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138274</th><td style="text-align: center">1/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/1</td></tr></tbody></table></div>



Here we use the ``take()`` method instead of ``compress()`` because we are using indices rather than a Boolean array to make the selection, and we use ``axis=1`` because we are selecting from the second axis, i.e., columns, i.e., samples.

Here's another way to do it, using the ``subset()`` method to apply both the variant and sample selections simultaneously, and using Dask to avoid loading the whole genotype array into memory:


{% highlight python %}
gt_afr = gt_dask.subset(loc_variant_selection, loc_samples_afr).compute()
gt_afr
{% endhighlight %}




<div class="allel allel-DisplayAs2D"><span>&lt;GenotypeArray shape=(138275, 661, 2) dtype=int8&gt;</span><table><thead><tr><th></th><th style="text-align: center">0</th><th style="text-align: center">1</th><th style="text-align: center">2</th><th style="text-align: center">3</th><th style="text-align: center">4</th><th style="text-align: center">...</th><th style="text-align: center">656</th><th style="text-align: center">657</th><th style="text-align: center">658</th><th style="text-align: center">659</th><th style="text-align: center">660</th></tr></thead><tbody><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">0</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">1</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/1</td><td style="text-align: center">1/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">2</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">...</th><td style="text-align: center" colspan="12">...</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138272</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138273</th><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/0</td></tr><tr><th style="text-align: center; background-color: white; border-right: 1px solid black; ">138274</th><td style="text-align: center">1/1</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">...</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">0/0</td><td style="text-align: center">1/1</td></tr></tbody></table></div>



## Further reading

Hopefully these examples have been helpful. For further information and more examples, the following may be useful:

* [Extracting data from VCF files](http://alimanfoo.github.io/2017/06/14/read-vcf.html)
* [A tour of scikit-allel](http://alimanfoo.github.io/2016/06/10/scikit-allel-tour.html) (N.B., this is an older article and uses slightly different techniques from the ones used here, but still relevant.)
* [Estimating F<sub>ST</sub>](http://alimanfoo.github.io/2015/09/21/estimating-fst.html)
* [Principal components analysis](http://alimanfoo.github.io/2015/09/28/fast-pca.html)

