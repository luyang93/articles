Comparative Analysis of Droplet-Based Ultra-High-Throughput Single-Cell RNA-Seq Systems
---

# Highlights
1. 比较了三种droplet-based的高通量单细胞分析系统，这三种系统（inDrop，Drop-seq，10X）的特点。
2. 建立对于这三个系统的一个通用评估方法
3. 灵敏度，精确度，偏差，花费（sensitivity precision bias costs）
4. 展示了inDrop平台的Smart-seq2的方案

![选择方式](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/11/30/选择方式.png)
# Summary
通过一样的细胞和生信分析流程，我们可以直接比较这三个平台的数据集。

# Introduction
- 2009年发布scRNA-seq
- cell type identification in tissues and organs（细胞分类）
- tracing cell lineage and fate commitment in embryonic development and cell differentiation（追踪胚胎发育和细胞分化的细胞谱系和fate commitment）
- drawing inferences on transcriptional dynamics and regulatory networks（描述转录动力学和调控网络之间的关系）
- identifying the development, evolution, and heterogeneity of tumors（识别肿瘤的发展，演化和异质性）
- labz-on-a-chip operations，更简单的分离单细胞
- micorwell-based，低花费高通量
- droplet microfluidics，三种流行的系统，inDrop，Drop-seq和10X Genomics Chromium
- 通过onbead primers上的barcode去区分不同细胞，用UMI去简并bias correction
- 三者的区别，bead的制造方式，barcode的设计，cDNA的富集
- GM12891
- 50k reads per cell barcode
- cell capture efficiency，effective read proportion，cell barcode error rate，transcript detection sensitivity四个方面去分析比较。
# Results
- PCR handle + cell barcode + UMI + poly-T

![barcode prime bead](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/11/30/barcode_prime_bead.png)
- inDrop的primer是光解的moiety和T7的promoter
- 10X和inDrop使用的是水凝胶，Drop-seq使用的是树脂
- one bead on cell，Drop-seq服从两个泊松分布，inDrop和10X是服从超泊松分布

![droplet generation](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/11/30/droplet_generation.png)