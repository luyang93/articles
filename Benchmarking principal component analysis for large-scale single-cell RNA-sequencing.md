Benchmarking principal component analysis for large-scale single-cell RNA-sequencing

---

# Background

对于大规模的单细胞测序数据集，PCA的计算时间很长并且需要很高内存。

# Result

1. Krylov subspace和randomized singular value decomposition在速度，内存和精确度上优于其他算法。
2. 对seurat使用者的建议: seurat能分析的规模(10^4^ genes × 10^5^ cells), PCA降维的算法选择性较少, 就prcomp(full-rand SVD), IRLBA(TruncatedSVD, Krylov)
3. 对于scanpy使用者的建议: scanpy能分析的规模(10^4^ genes × 10^6^ cells), 根据一些目的区分, PCA时选择的参数:

	- 数据规模超级大, 可以忍受一些稀有细胞类型丢失或者亚群丢失, 选择chunked方式, 渐进式(IncrementalPCA)的out-of-core计算PCA, chunk_size设置为大一些, 最少100(>num of PCs)
	- 数据规模很大(稀疏矩阵), 又不太想一些稀有细胞类型丢失或者亚群丢失, 选择arpack/lobpcg(操作稀疏矩阵), zero_center=True
	- 数据规模很大(稀疏矩阵), 常规分析, zero_center=False, PCA降维(TruncatedSVD)得到的PCs可能会受到某些基因表达量的影响.
	- 数据规模不是很大(稠密矩阵), 选择randomized, zero_center=True

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606819796972.png)

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606820984456.png)

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606814124058.png)

# 必要性

快速和节省内存的PCA算法

一些近似算法导致的信息丢失, 稀有的细胞丢失

# 现存PCA

## full-rank SVD

协方差矩阵的特征值分解(EVD), 矩阵的奇异值分解(SVD)

prcomp(R), PCA(python sklearn) Golub-Kahan method, full-rank SVD based on LAPACK(DGESVD, DGESVD)

- 大规模数据，out of memory, 蓝色的部分需要稠密矩阵

![image-20201201062331557](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201062331557.png)

## truncated SVD, 最高的几个PC会被计算

- similarity transformation-based(SimT)
- downsampling-based(DS)
- singular value decomposition update-based(SU)
- Krylov subspace-based(Krylov)
- gradient descent-based(GD)
- random projection-based(Rand)

DS: 随机抽取列(细胞), 形成小矩阵, 然后求SVD. 剩余细胞矩阵投影到特征向量的低维度空间, 得到剩余细胞的特征值.(mouse brain)

SU: 对矩阵的局部不断计算SVD, 渐进的更新整个结果.(loompy)

![image-20201201073425220](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201073425220.png)

Krylov: 由于一些限制, 结果只会无限趋近与最大特征值(PC1)相关的特征向量. 用其他一些方法IRLBA, IRAM计算了其他较高的PCs. 相比于GK method, 不需要临时的大稠密矩阵, 也可以对稀疏矩阵操作.(Cellranger, Seurat2, Scanpy, SAFE, Scran, Giniclust2, MAGIC, Harmony, Scater)

![image-20201201073450938](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201073450938.png)

GD: 初始梯度向量会不断被反向梯度更新. 在scRNA-seq中目前没有实际应用.

![image-20201201074516760](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201074516760.png)

Rand: 原始矩阵投影到低维度空间, 然后对小矩阵进行QR和SVD(时间空间效率), 然后通过小矩阵的SVD重建原始矩阵的SVD(较低的误差). Li's Method(algorithm971, CellFishing.jl ), Halko's method(randomized SVD, scanpy, SIMLR, SEQC)

![image-20201201080132395](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201080132395.png)

## 总结

加速方案都是大矩阵变小矩阵, scalable: out-of-core(分批次处理小矩阵, 渐进更新计算). 相反, 空间复杂度O(NM), N: genes, M:cells

![image-20201201081958603](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201081958603.png)



# Accuracy(真实数据)

数据集: PBMCs, Pancreas, BrainSpinalCord, Brain

![image-20201201082743165](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201082743165.png)

## 对于PBMCs和Pancreas数据集, procomp作为标准(full-rank SVD), 其他方案与此比较.

t-SNE: Downsampling会忽略一些subgroups

![image-20201201105602264](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201105602264.png)

### ARI(adjusted Rand index)值：

![image-20201201114909134](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201114909134.png)

![image-20201201113732942](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201113732942.png)

结论: Louvain cluster中sgd(OnlinePCA.jl)和downsampling的ARI值较差

### PCs之间的关联性

![image-20201201124710496](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201124710496.png)

![image-20201201130348713](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201130348713.png)

结论: 
- downsampling和GD的结果不是很好.
- k-means/GMM的聚类方式容易导致一些异常cell聚类成单例簇
- Krylov(IRLBA IRAM)和SVD的结果与标准相似, dask的方法为特例, Direct Tall-and-Skinny QR algorithm

### 比较loading vectors

基因上的特征向量(loading vectors)容易受矩阵和细胞上的特征向量(PCs)影响
取top 500 genes, 比较两个loading vectors之间共同的基因.
![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606802589332.png)
结论:
- downsampling, GD, dask的精确度较差

### 比较每个PC的贡献

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606803204541.png)

结论:
- downsample, IncrementalPCA (sklearn), sgd的结果与其他的不同


## 对于BrainSpinalCord和Brain数据集, 使用特征值的大小和分群准确性比较。
无法使用full-rank SVD

### 不同算法PCs之间的关联性

![大数据集](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606801302606.png)

结论: 
- downsampling和GD的结果与其他PCA实现方式差异较大



# Time, Memory, scalability(人造数据)

downsampling: BrainSpinalCord的PCA较快, 但是前处理花时间. Brain的PCA就很慢了(full-rank SVD)

out-of-core methods: dask-ml超时, IncrementalPCA (sklearn), orthiter/gd/sgd/halko/algorithm971 (OnlinePCA.jl), and oocPCA_CSV (oocRPCA)在在3天之内完成.

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606803814875.png)

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606806999425.png)

![时间](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606807372479.png)

![空间](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606807405918.png)

## 数据格式与性能关系
主要测ooc实现方案(out-of-core)
- oocPCA_CSV(R, oocRPCA)
- IncrementalPCA(Python, sklearn)
- orthiter/gd/sgd/halko/algorithm971 (Julia,OnlinePCA.jl)
 
![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606815192122.png)
结论: 
- csv读取的时间较长, 处理过的二进制文件加载起来较快.
- 稀疏矩阵读起来比较快, 内存占用也比更低(???)
- 稀疏矩阵对计算的加速
- OnlinePCA.jl实现了一些ooc的方案, 并可以把PCA计算的结果导出.

# Guideline for User
GC = num of genes × num of cells

- GC<10^7^, full-ranked SVD
- 10^7^<GC<10^8^, Krylov(IRLBA, IRAM), Rand(Li, Halko methods)
- 10^8^<GC<10^10^, 稀疏矩阵读取入内存
- GC>10^10^, out-of-core

![enter description here](https://raw.githubusercontent.com/bedforimg/imgbed/master/images/2020/12/1/1606818813267.png)

IncrementalPCA (sklearn): 对chunksize调优, 较大的chunksize会增加PCA结果的准确性, 但是会增加时间和空间
Rand(LI, halko): 对迭代次数调优, 至少3次, sklearn默认5

# Guideline for Dev
注意点: 
- 某些操作会让稀疏矩阵"失去"稀疏性, 比如中心化, X-Xmean. 解决方案 (X − Xmean) W = XW − XmeanW, 渐进式计算
- lazy loading, out-of-core的方式进行centering, scaling, 


其他: 
- 1.out-of-core方案, HDF5/loom 2.稀疏矩阵方案
- PCA算法 EVD(稠密矩阵, 稀疏矩阵) truncated SVD
- 集群方案, MPI OpenMP等等

# 拓展

CCA的数学原理也和PCA类似, 优化 randomized CCA or SGD of CCA
downsampling比率的问题