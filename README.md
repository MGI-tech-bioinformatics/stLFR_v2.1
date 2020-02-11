# stLFR_V1.2

Introduction
-------
Tool of stLFR(Single Tube Long Fragment Reads) data analysis

stLFR FAQs is directed to MGI_Bioinfor@genomics.cn.

Download source code package from https://github.com/MGI-tech-bioinformatics/stLFR_V1.2

Updates 
-------
Jan 10, 2020
1. updated SV module (SV2.1, https://github.com/MGI-tech-bioinformatics/stLFR_SV2.1_module)
2. added novel automated HMTL report
3. new consturcture of tool with some new parameters 

May 6, 2019
There are several updates in stLFR_v1.1 comparing with v1:
1. Users could use an alternative reference type (hg19 or hs37d5) in stLFR_v1.1 by --ref option instead of only hg19.
2. Updated CNV and SV detection tools are implied in stLFR_v1.1 for decreasing false discovery rate.
3. Three figures used for illustrating stLFR fragment distribution and coverage are added.
4. NA12878 benchmark VCF by GIAB is used for haplotype phasing error calculation.

Download/Install
----------------
Due to the size limitation of GitHub repository, the database directory ('stLFR_V1.2/db') and tool directory ('stLFR_V1.2/tools') are provided by BGI Cloud Drive:

      1. tools Link: https://pan.genomics.cn/ucdisk/s/B7Nryq
      2. database Link: https://pan.genomics.cn/ucdisk/s/vmU3aq

Tool list in directory ('stLFR_V1.2/tools'):

      1. bam2depth
      2. bwa
      3. circos
      4. cnv
      5. fqcheck
      6. gatk4
      7. HapCUT2-master
      8. jre
      9. MegaBOLT
      10. monitor
      11. picard
      12. Python2
      13. python3
      14. R
      15. rtg-tools
      16. samtools
      17. SOAPnuke
      18. sv
      19. vcftools

Database list in direcotry ('stLFR_V1.2/db'):

      1. barcode/
      2. db.list
      3. gatk/
      4. NA12878/
      5. phasedvcf/
      6. queue.list
      7. reference/

Meanwhile, two Demo stLFR libraries are provided for test, and every library consists two lanes.
Libraries Link:

      1. T0001-2: ftp://ftp.cngb.org/pub/CNSA/CNP0000387/CNS0057111/
      2. T0001-4: ftp://ftp.cngb.org/pub/CNSA/CNP0000387/CNS0094773/

Usage
-------
1. Make sure 'sample.list' file on a right format, you can refer to 'path' file in the example.

2. Run the automatical delivery script. Default reference: [hs37d5]

         perl bin/stLFR_SGE -l SAMPLELIST -analysis align -outputdir OUTPUTDIR -inputdir INPUTDIR ...

Main progarm arguments:
----------

   Sample List
   
       -list FILE
                     Name of input file. This is required.

                     Five columns in the list file, which the front three columns are required:
                     1. name    : unique sample ID in this analysis
                     2. path    : fastq path(s) for this sample split with colon(:)
                     3. barcode : sample-barcode for each path split with colon(:), 0 means all used
                     4. reffile : reference with index, two inner options are 'hg19' and 'hs37d5', NULL or '-' means 'hs37d5'
                     5. vcffile : dbsnp file, default is NULL or '-'
                     6. blacklist : black list file(BED format) for SV
                     7. controllist : sorted control list file(BEDPE format) for SV

                     eg:  SAM1   /DATA/slide1/L01                    1-4,5,7-9
                          SAM2   /DATA/slide1/L01:/DATA/slide2/L01   0:1-8         hg19
                          SAM3   /DATA/slide2/L02                    0             REFERENCE/ref.fa      DBSNP/dbsnp.vcf
                          SAM4   /DATA/slide2/L03                    0             hs37d5                -                   BLACKLIST    CONTROLLIST

   Previous Result Directory
   
       -inputdir
                     Input directory path with results in previous process. [ ]
                     1. align:  need filter result
                     2. phase:  need align result
                     3. cnvsv:  need align and phase result

   Output Directory
   
       -outputdir [ ./ ]
                     Output directiry path.

                     The Format of Output directory (also input directory of this workflow):
                     Inputdir/
                     |-- 01.filter   // for align
                     |   |__ SAMPLE
                     |       |__ SAMPLE.clean_1.fq.gz
                     |       |__ SAMPLE.clean_2.fq.gz
                     |-- 02.align    // for phase, cnv and sv
                     |   |__ SAMPLE
                     |       |__ SAMPLE.sortdup.bqsr.bam
                     |       |__ SAMPLE.sortdup.bqsr.bam.bai
                     |       |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz
                     |       |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz.tbi
                     |-- 03.phase    // for cnv
                     |   |__ SAMPLE
                     |       |__ phasesplit
                     |           |__ hapblock_SAMPLE_CHROMOSOME
                     |-- 04.cnv
                     |   |__ SAMPLE
                     |       |__ SAMPLE.CNV.result.xls
                     |-- 05.sv
                     |   |__ SAMPLE
                     |       |__ SAMPLE.SV.simple.result.xls
                     |       |__ SAMPLE.SV.complex.result.xls
                     |__ file
                         |__ SAMPLE
                             |-- alignment
                             |   |__ SAMPLE.sortdup.bqsr.bam
                             |   |__ SAMPLE.sortdup.bqsr.bam.bai
                             |-- CNV
                             |   |__ SAMPLE.CNV.result.xls
                             |-- haplotype
                             |   |__ SAMPLE.hapblock
                             |   |__ SAMPLE.hapcut_stat.txt
                             |   |__ SAMPLE.inked_fragment
                             |-- sequence
                             |   |__ SAMPLE.clean_1.fq.gz
                             |   |__ SAMPLE.clean_2.fq.gz
                             |-- SV
                             |   |__ SAMPLE.SV.result.xls
                             |__ variant
                                 |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz
                                 |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz.tbi

   Analysis Modules
   
       -analysis  [ all ]
                     There are 5 modules in this program: filter, align, phase, cnvsv, report.
                       -analysis all  == -analysis filter,align,phase,cnvsv,report.
                       -analysis base == -analysis filter,align,phase,report.

                     eg: all                : filter + align + phase + cnvsv + report
                         base               : filter + align + phase + report
                         align              : align + report
                         filter,align,phase : filter + align + phase + report

   stLFR barcode position
   
       -position [ 101_10,117_10,133_10 ]
                     Position of stLFR barcodes on read2.

   Task Monitor Type
   
       -type    [ local ]
                     Task monitor type.
                     blc:    running on BLC using software defined by -task
                     fpga:   running on FPGA server using watchDog
                     local:  running on local machine using watchDog
       -cpu      [ 60 ]
                     CPU number on server, using when -type blc and -type fpga

   Baseline for SNP/INDEL evaluation
   
       -baseline   Baseline VCF for NA12878 on hg19 or hs37d5.
       -confbed    High confidence BED for NA12878 on hg19 or hs37d5.

   Usage
   
       -help       Show brief usage synopsis.
       -man        Show man page.
       -run        Run workflow after main shells built.

Result
-------
After all analysis processes ending, you will get these files below:

      1.  HTML report:                              *_cn.html, *_en.html
      2.  raw data summary:                         *.fastqtable.xls
      3.  stLFR barcode summary:                    *.fragtable.xls
      4.  alignment summary:                        *.aligntable.xls
      5.  variant summary:                          *.varianttable.xls
      6.  haplotype phasing summary:                *.haplotype.xls, *.haplotype.pdf
      7.  evaluation summary (NA12878):             *.evaluation.xls
      8.  quality distrubution in cleanfq:          *.Cleanfq.qual.png
      9.  base distribution in cleanfq:             *.Cleanfq.base.png
      10. depth distribution in alignment:          *.Sequencing.depth.pdf
      11. accumulated depth distribution:           *.Sequencing.depth.accumulation.pdf
      12. insert size distrubition:                 *.Insertsize.metrics.txt, *.Insertsize.pdf
      13. GC bias distrubution:                     *.GCbias.metrics.txt, *.GCbias.pdf
      14. Fragment coverage figure:                 *.frag_cov.pdf
      15. Fragment length distribution figure:      *.fraglen_distribution_min5000.pdf
      16. Fragment per barcode distribution figure: *.frag_per_barcode.pdf
      17. variant CIRCOS:                           *.circos.svg, *.circos.png, *.legend_circos.pdf

Additional Information
-------
1. If user has "Permission denied" problem in the process of running，you can use the command "chmod +x -R stLFR_v2.1/tools" to get executable permission of tools.


License
-------
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions： 
  
The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
  
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
