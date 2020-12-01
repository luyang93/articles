Benchmarking principal component analysis for large-scale single-cell RNA-sequencing

---

# Background

对于大规模的单细胞测序数据集，PCA的计算时间很长并且需要很高内存。

# Result

Krylov subspace和randomized singular value decomposition在速度，内存和精确度上优于其他算法。

# 必要性

快速和节省内存的PCA算法

一些近似算法导致的信息丢失, 稀有的细胞丢失

# 现存PCA

## full-rank SVD

协方差矩阵的特征值分解(EVD), 矩阵的奇异值分解(SVD)

prcomp(R), PCA(python sklearn) Golub-Kahan method, full-rank SVD based on LAPACK(DGESVD, DGESVD)

- 大规模数据，out of memory

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

## 对于BrainSpinalCord和Brain数据集, 使用特征值的大小和分群准确性比较。

### ARI(adjusted Rand index)值：

![image-20201201114909134](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201114909134.png)

![image-20201201113732942](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201113732942.png)

结论: Louvain cluster中sgd(OnlinePCA.jl)和downsampling的ARI值都不是很好

### PCs之间的关联性

![image-20201201124710496](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201124710496.png)

![image-20201201130348713](https://raw.githubusercontent.com/bedforimg/imgbed/master/img/2020/12/01/image-20201201130348713.png)

downsampling和GD的结果不是很好。




# Time, Memory, scalability(人造数据)

