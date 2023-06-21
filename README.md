# **RNA Sequencing Data Analysis Report**
#### **摘要:**

mRNA或cDNA的高通量测序 (RNA-seq) 是测量和比较各种物种和不同条件下基因表达水平的标准方法，其生成的庞大复杂的数据集需要快速、准确的软件来将原始读取数据处理为直观易懂的结果[1]。

Hisat2、cufflinks等是用于全面分析RNA-seq的免费开源软件工具，将原始序列与基因组进行比对，组装包括新剪接变体在内的转录本，计算每个样本中这些转录本的丰度，并识别差异表达的基因和转录本。本项目主要介绍了对RNA序列数据进行完整分析使用的各种工具、方法和操作流程，涉及各种工具包以及RNA等数据集的下载、预处理原始数据（质量控制）、比对序列、转录本组装、获得基因表达文件、挑选差异基因、绘制多样化的图谱（火山图、热图、相关系数图、散点图）。

目前可以用于RNA-seq的分析工具很多，除了Hisat2、cufflinks外，还有Stringtie、Subread等。由于工具的多样性，本组分别设计了三条路线进行基因的差异分析及可视化展现，在分析的过程中比对三种方法的优缺点。

报告同时涵盖了研究背景介绍、项目讨论结果以及每位作者的贡献，附录中有详细的Dockerfile和项目核心代码、脚本。

#### **前言:**

##### **背景介绍：**

RNA高通量测序 (RNA-seq) 实验从一组细胞中捕获总mRNA，然后对该RNA进行测序以确定在这些细胞中处于高表达状态的基因，这些实验产生了大量的原始测序读数，即使是中等规模的检测，通常也有数千万个[2]。每个基因产生的读数数量可以用作基因丰度的量度，并且通过适当的设计，RNA-seq 可以检测哪些基因在两种或多种条件下以显著不同的水平表达。借助适当的软件，RNA-seq 数据可以轻松检测出标准注释中未包含的基因（非编码RNA）和基因变异，单个基因的不同异构体受到差异调节和表达的条件。生物信息学界也一直在努力开发RNA-seq 的数学、统计学和计算机科学，并将这些研究应用于构建相关软件工具中。RNA-seq 必须使用精确、高效的软件进行分析，可以分为几个主要任务：①测序数据与参考基因组的比对；②将比对组装成全长转录本；③量化每个基因和转录本的表达水平；④计算所有基因在不同实验条件下的表达差异；⑤后续可视化分析。

##### **工作思路/概要：**

该项目旨在通过对RNA测序数据的完整分析，深入了解基因表达模式并识别差异表达的基因。将所有的参考基因组序列、参考基因组注释文件、测序数据下载完成后，首先要进行数据的预处理。由于下载的数据为sra格式文件，因此用fastq-dump将sra文件转换为fastq文件，方便后续数据处理。之后运用fastqc进行质控并生成质控报告，对数据进行评估。如果数据的指标较差，则需要用trimmomatic进行数据过滤，用过滤后的数据进行后续分析。

数据分析有三种流程。

第一种为Hisat2 –> Cufflink –> Cuffdiff。该流程利用hisat2对参考基因组序列建立索引并将测序数据的reads比对到参考基因组上，cufflinks对每个样本进行转录本组装，cuffmerge将所有的转录本合并到一个转录本中形成一个新的转录本注释文件，cuffdiff获得基因表达文件。

第二种为Hisat2 –> Stringtie –> Ballgown。

第三种为Subread -> DESeq2-> ggplot。该流程先使用subread-buildindex 对参考基因组序列建立索引，随后用subjunc 对双端测序进行比对，然后用featureCounts进行定量并提取定量信息、生成矩阵。之后把矩阵导入R中，用R中的DESeq2包进行差异分析，筛选出上调或下调的基因，最后用ggplot绘制火山图。

#### **数据集与方法:**

##### **工具：**

Anaconda、sra-tools、fastqc、trimmomatic、hisat2、bowtie2、samtools、cufflinks、subread、DESeq2、R

##### **数据：**

水稻的参考基因组文件和注释文件：从[National Center for Biotechnology Information (nih.gov)](https://www.ncbi.nlm.nih.gov/)下载水稻Reference genome [IRGSP-1.0](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_001433935.1/) National Institute of Agrobiological Sciences (2015). Cultivar: Nipponbare.的Genome sequences (FASTA)和Annotation features (GTF)

拟南芥的参考基因组文件和注释文件：ftp://ftp.ensemblgenomes.org/pub/plants/release-28/fasta/arabidopsis_thaliana/cdna/Arabidopsis_thaliana.TAIR10.28.cdna.all.fa.gz

ftp://ftp.ensemblgenomes.org/pub/plants/release-28/fasta/arabidopsis_thaliana/dna/Arabidopsis_thaliana.TAIR10.28.dna.genome.fa.gz

ftp://ftp.ensemblgenomes.org/pub/plants/release-28/gff3/arabidopsis_thaliana/Arabidopsis_thaliana.TAIR10.28.gff3.gz

ftp://ftp.ensemblgenomes.org/pub/plants/release-28/gtf/arabidopsis_thaliana/Arabidopsis_thaliana.TAIR10.28.gtf.gz

测序数据：SRR1999345、SRR1999346、SRR1999347、SRR1999348
