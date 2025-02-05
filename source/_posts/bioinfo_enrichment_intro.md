---
title: 富集分析：（一）概述
date: 2021-11-12
categories: 
- bio
- bioinfo
- enrichment
tags: 
- enrichment analysis
- over representation analysis
- ORA
- gene set enrichment analysis
- GSEA
- KOBAS-i
- GOEAST
- topGO
- clusterProfiler
description: 介绍了富集分析和分析软件，包括在线富集分析工具KOBAS-i和GOEAST，富集分析R包topGO，clusterProfiler。
---

<div align="middle"><iframe width="298" height="52" src="https://www.youtube.com/embed/wcOM3Rx43ko" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

（全文约​6600字）

# 1. 富集分析
## 1.1. 富集分析概念
1. 富集分析
富集分析，本质上是对数据的分布检验，如果分布集中在某个区域，则认为富集。
常用的分布检验方法有卡方检验、Fisher精确检验以及KS检验等方法。

2. 生物信息学领域的富集分析
在 **背景基因集(N)** 下获得 **一组特定基因集(S)** ，S可能是基因列表，表达图谱，基因芯片等形式。在预先构建好**基因注释数据库**(例如GO,KEGG等)已对背景基因集(N)根据生物功能或过程进行分类的前提下，通过**统计学算法**找出有那些显著区别于背景基因集(N)的类别(生物组成/功能/过程)，或者找出这组特定基因集间在生物组成/功能/过程的共性，经过聚类后去除冗余得到基因富集结果的过程，即为富集分析。

可以这样简单理解富集分析在做什么。全国人口的户籍作为背景数据，我们通过富集分析可以知道相对于全国背景，客家人是不是明显在广东聚集。比如如果广东的客家人数/全国客家人数这个比值远超过广东人数/全国人数的比值，那么我们可以说客家人在广东是富集的。

3. 常用数据
- 其中，**背景基因集(N)** 常常是一个物种的基因组注释基因总和。
- 一组**特定基因集(S)** 常常是差异表达基因集(differentially expressed genes, DEGs)。
- 预先构建好**基因注释数据库**常用GO或KEGG数据库。
- 常见的**统计学算法**包括ORA,FCS,PT,NT四种。

4. 实际应用
通常会使用其他分析的结果作为**特定基因集(S)**，做基因富集分析来查看这些基因集是否主要集中在某些类别，这些类别代表的功能是否与表型或者进化事件有关联。比如：
- 比较转录组分析得到的差异表达基因集；
- 比较基因组分析中得到的某物种特有的基因集；
- 基因家族收缩扩张分析得到的基因组中显著扩张/收缩的基因集；
- 基因组共线性分析中在全基因组复制事件附近的Ks值的基因集等各种分析得到的基因集；

## 1.2. 富集分析的算法
富集分析算法经过发展，常见的有四类：

<img src="https://www.sciengine.com/figures/figures-29603243e0234a3cb08e1f7155debca6-picobject1.png" title="富集分析四类算法" width="90%"/>

**<p align="center">Figure 1. 富集分析四类算法**
from [paper:Progress in Gene Functional Enrichment Analysis](https://www.sciengine.com/SSV/article?doi=10.1360/N052016-00139&scroll=)</p>

### 1.2.1. 过表达分析(Over Representation Analysis, ORA)
1. 过表达分析(ORA)概念
过表达分析(ORA)是对背景基因集(N)和特定基因集(S)按照已知的基因功能或通路等分组，并鉴定特定基因集(S)在哪些组包含比背景基因集(N)比例更多的基因(过度表达，over-represented，即富集enriched)或更少的基因(表达不足，under-represented)的一种统计学方法。

ORA是出现最早，最常用，有完善的统计学理论基础的方法。ORA重点在于通过基因集组成的比例来判断富集程度。

2. 过表达分析的分布检验
ORA类方法用的是离散分布的检验（Fisher精确检验，依据超几何分布的原理）。

超几何分布有很多资料可参考，比如：https://www.jianshu.com/p/13f46bebebd4

3. 过表达分析(ORA)的局限性
- ORA使用的统计方法(例如超几何分布，二项分布，卡方分布)只考虑差异基因数量，忽略了差异程度(不同表达水平)，，需要人为设置阈值，没有一个设置规定，阈值设置有主观性。
- 通过一刀切的人为规定的阈值，找出差异最显著的单个基因，而忽略其他基因，比如差异小但变化方向一致的基因集，往往后者比前者更重要。
- 利用的统计学假设每个基因相互独立，但是就生物体本身而言，忽略了基因间内部的复杂的相互作用，并且每个基因在不同的生物学过程中发挥的作用大小不一样，同等看待结果可能会不准确。
- 假设每条通路都独立其他通路。

4. 过表达分析(ORA)的实际操作
需要四组数据：
- 背景基因集(N)：常常是一个物种的基因组注释基因总和
- 特定基因集(S)：常常是差异表达基因集(differentially expressed genes, DEGs)
- 背景基因集的分类信息：常常通过注释数据库(GO,KEGG等)和背景基因集获得
- 特定基因集的分类信息：常常通过注释数据库(GO,KEGG等)和特定基因集获得

通过四组数据获得基因数量的2×2列联表，再利用fisher精确检验或超几何分布得到p值。认为设置一个显著性阈值，高于阈值的即为富集基因。

### 1.2.2. 功能分类打分(Functional Class Scoring,FCS)
#### 1.2.2.1. 功能分类打分(FCS)
1. 功能分类打分(FCS)概念和改进
功能分类打分(FCS)与ORA相比进行了基本假设的改变，除了考虑单个基因的显著变化外，还考虑微效基因的叠加效果。代表是基因集富集分析(Gene Set Enrichment Analysis, GSEA)

2. FCS分析的三个步骤
- 计算单个基因表达水平的统计值，采用如衡量差异基因的ANOVA、Q-statistic、t检验、Z-score、信噪比，进行打分或排序，或者直接使用排序好的基因表达谱
- 同一通路上所有基因的表达水平统计值进行整合，汇集成单个通路水平的分数或统计值，采用基因水平统计的和、均值或中位数，Wilcoxon rank sum, Maxmean statistic, Kolmogorov-Smirnov statistic
- 对通路水平的显著性进行评估：利用重抽样(bootstrap)的统计学方法

3. 功能分类打分(FCS)的优势(与ORA相比)
- 不需要人为规定的阈值来一刀切显著性
- FCS考虑同一通路中基因表达的协调变化，即考虑微效基因的叠加效果。
- 通过考虑基因表达的协调变化，FCS 方法解释了通路中基因之间的依赖性

4. 功能分类打分(FCS)的限制
- 与 ORA 类似，FCS 独立分析每个通路。一个基因可以在多个通路中发挥作用，这意味着这些通路可以交叉和重叠。
- 许多 FCS 方法使用基因表达的变化来对给定通路中的基因进行排序，并丢弃进一步分析的变化。例如，假设通路中的两个基因 A 和 B 分别发生 2 倍和 20 倍的变化。只要它们与通路中的其他基因相比具有相同的各自等级，大多数 FCS 方法都会平等地对待它们。

#### 1.2.2.2. 基因富集分析(gene set enrichment analysis, GSEA)
基因富集分析(GSEA)是FCS算法中最常用的一种。
1. 基因富集分析(GSEA)的原理
- 基因排序
首先，根据各基因与表型间相关性r或两组间t检验统计量得分值对背景基因集(N)进行降序排列，比如把所有基因在两个分组(或表型)中的差异度从大到小排序，形成排好序的基因列表。
- 基因富集
查看基因注释数据库(GO/KEGG)中每个小组基因集(例如GO一个通路一个小组)里的基因是否在排序的背景基因集里均匀分布，或者主要分布在排序背景基因集的顶部/底部。均匀分布说明不在这两个分组(或表型)中富集，集中分布在顶部/底部说明这个小组基因集在两个分组(表型)之一富集。
- 富集分析
计算每一小组基因集的富集分数(enrichment score,ES)值，然后对ES值进行显著性检验和多重假设检验，计算得出显著富集的基因集。

2. 基因富集分析(GSEA)分析步骤
- 计算富集分数(enrichment score,ES)：对每一个小组基因集，遍历排好序的背景基因集(N)，当基因出现在特定基因集(S)就加分，反之减分，加减分值由基因与表型的相关性决定。
- 估计ES的统计显著性：基于样品的置换检验可以计算P值。
- 多重假设检验矫正：根据每一小组基因集的大小对每个基因的ES做标准化，得到标准化NES（normalized enrichment score ，NES）。为了检验每一小组基因集的NES是否显著，将排好序的背景基因集(N)随机打乱排列一定次数，每次都计算每个基因集的NES(ES)，得到每个基因集的NES在随机排序情况下的理论分布，从而计算其p值。若p<0.05，则说明该基因集在有序背景基因集(N)中大都富集在顶部，为富集基因集。FDR则是对p值进行FDR校正之后的p值。

### 1.2.3. 通路拓扑(Pathway Topology,PT)
1. 背景
- 大量公开可用的通路数据库提供的信息超出了每个通路的简单基因列表。与 GO 和分子特征数据库 (MSigDB) 不同，这些知识库还提供有关在给定途径中相互作用的基因产物、它们如何相互作用（例如，激活、抑制等）以及它们在何处相互作用（例如、细胞质、细胞核等）。这些知识库包括 KEGG、MetaCyc、Reactome、RegulonDB、STKE ( http://stke.sciencemag.org )、BioCarta ( http://www.biocarta.com )、和 PantherDB等。
- ORA 和 FCS 方法仅考虑通路中的基因数量或基因共表达来识别重要通路，而忽略这些知识库中可用的附加信息。因此，即使通过基因之间的新联系完全重新绘制了通路，只要它们包含相同的基因组，ORA 和 FCS 也会产生相同的结果。
- 已经开发的通路拓扑 (PT) 的方法则利用这些通路数据库的附加信息。
- 通路路径拓扑 (PT) 的方法本质上与 FCS 方法相同，因为它们执行与 FCS 方法相同的三个步骤。两者之间的主要区别在于使用通路拓扑来计算基因水平的统计数据。

2. 通路拓扑(PT)
在通路的富集分析中，一般上游基因的表达水平改变要显著大于下游基因对整个通路的影响。PT方法就是把基因在通路中的位置，和其他基因的互作和调控关系结合在一起，评估每个基因对通路的贡献并算出权重，然后把权重整合到富集分析。

代表是SPIA，由于可用数据库的限制，应用还很少。

### 1.2.4. 网络拓扑结构(Network Topology,NT)
利用现有的全基因组范围的生物网络，提取数据库的基因相互作用关系（如：基因连接度、基因在网络中的距离），把基因的生物学属性整合到功能分析。利用网络拓扑结构来计算基因对特定生物通路的重要性并给予相应的权重，再利用传统的ORA 或 FCS 方法来评估特定生物通路的富集程度，如GANPA 和 LEGO。缺点就是算法太复杂，计算速度慢。

# 2. 富集分析常用基因集
## 2.1. Gene Ontology(GO)
### 2.1.1. GO
- GO是一个国际标准化的基因功能分类体系，由基因本体联合会(Gene Ontology Consortium，GOC) 负责。它提供了一套动态并可控的词汇表（controlled vocabulary）来全面描述生物体中基因和基因产物的属性，它由一组预先定义好的GO术语（GO term）组成，这组术语对基因产物的功能进行限定和描述。
- 可在[GO官网](http://geneontology.org/))使用的[AmiGO2网站](http://amigo.geneontology.org/amigo/landing)查询GO ID和GO term信息。之前常用的[WEGO 2.0](https://wego.genomics.cn/)也可以查询。

GO由三个ontology（本体）组成，是由独立的术语表示的，分别描述基因的细胞组分（cellular component，CC）、分子功能（molecular function，MF）、参与的生物过程（biological process，BP）。

GO这三个本体的含义：
1. 细胞组成（cellular component，CC）：描述基因产物执行功能的细胞结构相关的位置，比如一个蛋白可能定位在细胞核中，也可能定位在核糖体中；
2. 分子功能（Molecular Function，MF）：描述基因产物发生在分子水平上的活动，例如催化或运输。通常对应于单个基因产物（即蛋白质或RNA）可以进行的活动。常见的宽泛的分子功能描述是催化活性(catalytic activity)和转运活动(transporter activity)。为了避免与基因产物名称混淆，通常分子功能描述后加上"activity"一词。
3. 生物过程（biological process，BP）：描述的是指基因产物所关联的一个大的生物功能，或者说是多个分子活动完成的一个大的生物程序。例如有丝分裂或嘌呤代谢；

### 2.1.2. GO terms
GO terms，它提供生物过程的逻辑结构与相关关系，不同GO terms之间的关系可以通过一个有向无环图来表示。

此处需要注意的是，GO terms是对**基因产物**，而不是基因本身进行描述，因为基因本身的产物有时候不止一种。GO数据库中的GO分类相关信息会得到不断地更新与增加，这个特点要记住，因为不同的GO分析工具使用的数据库版本有可能不一样，造成GO分析结果出现不同。

### 2.1.3. GO annotations
GO注释（GO annotations）是关于特定基因功能的声明，它主要是将GO terms和基因或基因产物相关联来提供注释，也就是描述这个GO terms关联的基因产物是什么（蛋白质，还是非编码RNA，还是大分子等），有什么功能，如何在分子水平发挥作用，在细胞中的哪个位置发挥作用，以及它有助于执行哪些生物过程(途径、程序)。

## 2.2. Kyoto Encyclopedia of Genes and Genomes(KEGG)
### 2.2.1. KEGG
- KEGG是处理基因组，生物通路，疾病，药物，化学物质的数据库集合，于1995年由京都大学化学研究所教授Minoru Kanehisa在当时正在进行的日本人类基因组计划下发起。
- KEGG 是一种数据库资源，用于从基因组和分子级信息了解生物系统（例如细胞、生物体和生态系统）的高级功能和效用。它是生物系统的计算机表示，由基因和蛋白质（基因组信息）和化学物质（化学信息）的分子构建块组成，它们与相互作用、反应和关系网络（系统信息）的分子接线图知识相结合。它还包含疾病和药物信息（健康信息）作为对生物系统的扰动。

[KEGG网站](https://www.kegg.jp/kegg/)提供了KEGG信息查询入口，包括[KEGG Pathway](https://www.kegg.jp/kegg/pathway.html)中查询KEGG Pathway ID(ko00000)的详细信息。

### 2.2.2. KEGG Database
KEGG 是一个集成的数据库资源，由如下所示的 16 个数据库组成。它们大致分为系统信息、基因组信息、化学信息和健康信息，它们通过网页的颜色编码来区分。

<table>
<caption><h4>KEGG Database</h4></caption>
<thead>
<tr>
<th>Category</th>
<th>Database</th>
<th>Content</th>
<th>Color</th>
</tr>
</thead>
</tbody>
<tr>
<td rowspan="3">Systems information</td>
<td>KEGG PATHWAY</td>
<td>KEGG pathway maps</td>
<td rowspan="3"><font color=green>kegg3</br>green</font></td>
</tr>
<tr>
<td>KEGG BRITE</td>
<td>BRITE hierarchies and tables</td>
</tr>
<tr>
<td>KEGG MODULE</td>
<td>KEGG modules and reaction modules</td>
</tr>
<tr>
<td rowspan="3">Genomic information</td>
<td>KEGG ORTHOLOGY (KO)</td>
<td>Functional orthologs</td>
<td><font color=yellow>kegg4 yellow</font></td>
</tr>
<tr>
<td>KEGG GENES</td>
<td>Genes and proteins</td>
<td rowspan="2"><font color=red>kegg1</br>red</font></td>
</tr>
<tr>
<td>KEGG GENOME</td>
<td>KEGG organisms and viruses</td>
</tr>
<tr>
<td rowspan="4">Chemical information</td>
<td>KEGG COMPOUND</td>
<td>Metabolites and other chemical substances</td>
<td rowspan="4"><font color=blue>kegg2</br>blue</font></td>
</tr>
<tr>
<td>KEGG GLYCAN</td>
<td>Glycans</td>
</tr>
<tr>
<td>KEGG REACTION</br>KEGG RCLASS</td>
<td>Biochemical reactions</br>Reaction class</td>
</tr>
<tr>
<td>KEGG ENZYME</td>
<td>Enzyme nomenclature</td>
</tr>
<tr>
<td rowspan="4">Health information</td>
<td>KEGG NETWORK</td>
<td>Disease-related network variations</td>
<td rowspan="4"><font color=purple>kegg5</br>purle</font></td>
</tr>
<tr>
<td>KEGG VARIANT</td>
<td>Human gene variants</td>
</tr>
<tr>
<td>KEGG DISEASE</td>
<td>Human diseases</td>
</tr>
<tr>
<td>KEGG DRUG</br>KEGG DGROUP</td>
<td>Drugs</br>Drug groups</td>
</tr>
</tbody>
</table>

### 2.2.3. KEGG PATHWAY Database
KEGG PATHWAY Database是KEGG资源的核心，是一组手工绘制的KEGG通路图，代表细胞和生物体的新陈代谢和各种其他功能的实验知识。每个通路图都包含一个分子相互作用和反应网络，旨在将基因组中的基因与通路中的基因产物（主要是蛋白质）联系起来。

# 3. 富集分析程序
目前有许多程序可以用于富集分析。包括NASQAR,PlantRegMap,MSigDB,Broad Institute,WebGestalt,Enrichr,GeneSCF,DAVID,Metascape,AmiGO 2,GREAT,FunRich,FuncAssociate,InterMine,ToppGene Suite,QuSAGE,Blast2GO,g:Profiler。

可以根据需要分析的物种类别和数据库更新选择分析平台，根据需要选择先验基因集。

大部分富集分析程序都只支持已有数据库的物种的富集分析，如果是不在支持物种列表里的物种，可以用富集分析的R包做富集分析。

下面介绍几种常见的。

## 3.1. NASQAR
NASQAR (Nucleic Acid SeQuence Analysis Resource)是一个开源的网页平台，可以用R包clusterProfiler做GSEA分析，支持[Org.Db数据库](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)的所有物种的GO Term和KEGG Pathway富集分析。
## 3.2. PlantRegMap
[PlantRegMap](http://plantregmap.gao-lab.org/go.php)，支持165种植物的GO注释和GO富集分析。
## 3.3. Enrichr
Enrichr是针对哺乳动物的基因富集分析工具。可以通过API使用，并提供可视化结果。
## 3.4. GeneSCF
GeneSCF支持多个物种，实时的功能富集分析工具。不需要额外更新数据库，GeneSCF是实时最新的数据库，并且支持多物种富集分析。结果以文本呈现。
## 3.5. DAVID
DAVID是做注释，富集和可视化的整合的数据库。但注释数据库自从2016年10月就没有更新。
## 3.6. Blast2GO
Blast2GO可以做组学数据的功能注释和GSEA分析。
## 3.7. KOBAS-i
### 3.7.1. KOBAS-i简介
1. KOBAS是中科院和北大联合开发的做功能注释和GSEA分析的工具。
2. 目前已经发表了第三个版本，KOBAS-intelligence(KOBAS-i)，KOBAS-i引入了之前发布的基于机器学习的新方法CGPS，并扩展了可视化功能，支持的物种增加到5944个。
3. KOBAS包含注释模块和富集模块。
- 注释模块接受基因列表作为输入，包括 ID 或序列，并根据通路、疾病和GO信息等多个数据库为每个基因生成注释。
- 富集模块给出了关于哪些通路和 GO 术语与输入基因列表或表达在统计上显着相关的结果。有两种不同的富集分析可用，命名为基因列表富集和 exp-data 富集。
4. KOBAS-i有网页版，也有本地版。

### 3.7.2. KOBAS-i网页版【推荐】
给了两个可用的网址：http://kobas.cbi.pku.edu.cn/，http://bioinfo.org/kobas。
[KOBAS-i](http://kobas.cbi.pku.edu.cn/)。
#### 3.7.2.1. 注释(annotation)和富集(Enrichment)的步骤
1. 输入
注释只有三个输入项：
- 选择物种。目前支持5944个物种。
- 选择输入基因集数据类型。支持核苷酸或者蛋白质的fasta序列，blast的Tabular输出结果，Ensembl Gene ID，Entrez Gene ID，UniProtKB AC，Gene Symbol。
- 输入基因集。可以粘贴文本，也可以上传文件。

富集除了需要上面三个输入项外，还需要选择用哪写先验基因集的数据库来做富集分析。包括PATHWAY,DISEASE,GO。其中KEGG Pathway是所有物种都可以用，其他的只有部份物种可以选。
- PATHWAY。有四个选项，KEGG Pathway(K),Reactome(R),BioCyc(B),PANTHER(p)。
- DISEASE。有三个选项，OMIM(o),KEGG Disease(k),NHGRI GWAS Catalog(N)
- GO(G)。
此外还有高级选项，可以选择统计学方法和纠错方法。

然后点击Run，就可以等待运行结果了。

2. 结果
运行结束后，点击右上角的Download total terms就可以下载到结果。
富集分析还可以点击Visualization得到结果的可视化图。

## 3.8. GOEAST
在线工具[GOEAST](http://omicslab.genetics.ac.cn/GOEAST/index.php)

# 4. 富集分析的R包
如果是模式物种，或者已有数据库的物种，推荐在线网站[PlantRegMap](http://plantregmap.gao-lab.org/go.php)和[KOBAS-i](http://kobas.cbi.pku.edu.cn/)做GO和KEGG的富集分析。

常见的有topGO，clusterProfiler，有一些进行富集分析的程序使用了这些包。

## 4.1. topGO
用topGO做富集分析的具体教程可以查看博文[blog_topGO](https://yanzhongsino.github.io/2021/11/13/bioinfo_enrichment_topGO/)。

topGO是一个R包，用于半自动的GO terms的富集分析。topGO的结果可以展示为有向无环图。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/fe/Tred-G.svg/800px-Tred-G.svg.png" width=40% title="DAG图" alt="DAG图" align=center/>

**<p align="center">Figure 2. DAG图示例**
from [wikipedia:directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph)</p>

注：对每个GO节点进行富集，在图中用方框表示显著度最高的10个节点，图中还包含其各层对应关系。每个方框（或椭圆）内给出了该GO节点的内容描述和富集显著性值。不同颜色代表不同的富集显著性，颜色越深，显著性越高。

## 4.2. clusterProfiler
clusterProfiler包的具体使用参考博文[blog_clusterProfiler](https://yanzhongsino.github.io/2021/12/13/bioinfo_enrichment_clusterProfiler/)。
### 4.2.1. clusterProfiler
clusterProfiler是一个R包，是一个解释组学数据的通用富集工具，支持Gene Ontology(GO), Kyoto Encyclopedia of Genes and Genomes(KEGG), Disease Ontology(DO), Disease Gene Network(DisGeNET), Molecular Signatures Database(MSigDb), wikiPathways和许多其他的基因集的功能注释和富集分析，以及富集分析结果的可视化。2021年07月发布了clusterProfiler 4.0版本。

### 4.2.2. clusterProfiler支持的基因集(gene sets)
1. Gene Ontology(GO)
2. Kyoto Encyclopedia of Genes and Genomes(KEGG)
3. Disease Ontology(DO)
4. Disease Gene Network(DisGeNET)
5. Molecular Signatures Database(MSigDb)
6. wikiPathways

### 4.2.3. clusterProfiler功能 —— enrichment analysis
1. Over Representation Analysis, ORA
ORA是用于判断已知的生物功能或过程在实验产生的基因列表（例如差异表达基因列表, differentially expressed genes, DEGs）中是否过表达(over-represented=enriched)的常用方法。
2. Gene Set Enrichment Analysis, GSEA
3. Leading edge analysis and core enriched genes

# 5. references
1. GSEA wiki：https://en.wikipedia.org/wiki/Gene_set_enrichment_analysis
2. paper-Ten Years of Pathway Analysis: Current Approaches and Outstanding Challenges：https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002375
3. paper-Progress in Gene Functional Enrichment Analysis：https://www.sciengine.com/SSV/article?doi=10.1360/N052016-00139&scroll=
4. enrichment analysis：https://www.jianshu.com/p/be8fe1318850
5. enrichment analysis methods：https://www.jianshu.com/p/042b888d5520
6. enrichment analysis methods：https://blog.csdn.net/fjsd155/article/details/103064166
7. GOEAST introduction：https://mp.weixin.qq.com/s?__biz=MzI5MTcwNjA4NQ==&mid=2247484456&idx=1&sn=bbcd0b5d10ba9312d92b7baae777ccde&scene=21#wechat_redirect
8. topGO tutorial：https://bioconductor.org/packages/release/bioc/vignettes/topGO/inst/doc/topGO.pdf
9. topGO blog：https://datacatz.wordpress.com/2018/01/19/gene-set-enrichment-analysis-with-topgo-part-1/
10. R topGO：https://www.codenong.com/cs105162324/
11. enrichment：https://www.jianshu.com/p/47b5ea646932
12. GO explanation：https://www.jianshu.com/p/7177c372243f
13. GO overview：http://geneontology.org/docs/ontology-documentation/
14. KEGG：https://en.wikipedia.org/wiki/KEGG
15. clusterProfiler github：https://github.com/YuLab-SMU/clusterProfiler
16. universal enrichment analysis using clusterProfiler：http://yulab-smu.top/biomedical-knowledge-mining-book/universal-api.html
17. clusterProfiler paper：https://www.cell.com/the-innovation/fulltext/S2666-6758(21)00066-7?_returnURL=https%3A%2F%2Flinkinghub.elsevier.com%2Fretrieve%2Fpii%2FS2666675821000667%3Fshowall%3Dtrue


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>