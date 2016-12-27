#MCMCtree的使用方法 
当用4dtv位点画出来的树形正确后，需要估计分化时间，用的是paml中包含的软件mcmctree
##1.运行baseml
###1.1 树文件:
```
11 1
((((((((pil,peu),ptr),wil),rco),(ath,cpa)'B(0.54, 0.90)')'B(0.95, 1.05)',(csa,fve)),vvi)'B(1.06, 1.19)',osa)'B(1.30, 1.80)''@1.48';
```
11：11个物种 1：1颗树  
B：分化时间 @1.48：根部的年龄(化石标记时间)  注:@1.48不能写成'(@1.48)',里面不能加括号，会导致结果不正确  
时间可以从文献中查出，单位是百万年

###1.2 basemlctl文件：
修改*号的行，其他为默认参数
```
**     seqfile = rm.miss.pil.total.4dtv.phyml.base * sequence data file name  ##phyml格式的输入序列文件
**      outfile = mlb        * main result file ##baseml的输出结果文件
**     treefile = RAxML_bipartitions.pil.4dtv.rmmiss.phb  * tree structure file name  ##树文件
```

###1.3 运行命令：
```
/share/work/user124/software/05.gene_analyze/paml4.8/src/baseml baseml.ctl;
```

##2.运行mcmctree第一步
###2.1 树文件：
```
11 1
((((((((pil,peu),ptr),wil),rco),(ath,cpa)'B(0.54, 0.90)')'B(0.95, 1.05)',(csa,fve)),vvi)'B(1.06, 1.19)',osa)'B(1.30, 1.80)''@1.48';
```
用第一步用的树即可
 
###2.2 计算range参数：
在mlb结果中，有一行输出是:
 ```
 Substitution rate is per time unit
                                 0.648944 +- 0.004968
 ```
range : 1,0.648944/(0.648944*0.648944)

###2.3 mcmctreectl文件
```
**       seqfile = rm.miss.pil.total.4dtv.phyml.base
**      treefile = RAxML_bipartitions.pil.4dtv.rmmiss.phb
**       outfile = out_usedata2
**         ndata = 1  #有1棵树，序列文件里面，把所有得核苷酸都连成一条，即一个物种一条序列
**       usedata = 3    * 0: no data; 1:seq like; 2:normal approximation #跑第一步mcmctree的时候此值为3
**       RootAge = '<10'  * constraint on root age, used if no fossil for root. #根部的化石年龄
**   rgene_gamma = 1 1.55     * gamma prior for rate for genes。 #之前计算出来的值
**       nsample = 10000  #调整取样的次数
```

###2.4 运行命令  
注：要加入baseml的路径，或者导入环境变量中
```
export PATH=$PATH:/share/work/user124/software/05.gene_analyze/paml4.8/src/;/share/work/user124/software/05.gene_analyze/paml4.8/src/mcmctree mcmc.1.ctl
```

##3.运行mcmctree第二步
###3.1 树文件：
```
11 1
((((((((pil,peu),ptr),wil),rco),(ath,cpa)'B(0.54, 0.90)')'B(0.95, 1.05)',(csa,fve)),vvi)'B(1.06, 1.19)',osa)'B(1.30, 1.80)';
```
注：第三步的tree不能有最后的化石标记时间，不能出现@

###3.2 mcmctree.ctl文件
```
**       usedata = 2    * 0: no data; 1:seq like; 2:normal approximation #这里要换成2
**   rgene_gamma = 1 1.54  #同上
**      finetune = 1: 0.06  0.5  0.006  0.12 0.4 ##跑的过程中要看下面的几个值
 5% 0.37 0.26 0.29 0.32 0.00  1.600 1.149 1.055 0.983 0.670 - 0.250 -12.9 26:14
 10% 0.37 0.26 0.29 0.31 0.00  1.600 1.149 1.055 0.983 0.670 - 0.249 -9.4 52:30
 15% 0.37 0.26 0.29 0.31 0.00  1.600 1.149 1.055 0.983 0.670 - 0.249 -13.5 1:18:46
 20% 0.37 0.26 0.29 0.31 0.00  1.600 1.149 1.055 0.983 0.668 - 0.249 -9.0 1:45:01
 25% 0.37 0.26 0.29 0.31 0.00  1.601 1.149 1.055 0.983 0.668 - 0.249 -6.6 2:11:17
 ……
```
这个值第一次为默认参数，在跑的过程中，屏幕上会出现：这样的提示，当前几列的值在0.2-0.4之间时，表明可信度很高，在这个范围以外时，需要调整这个参数，让他接近0.3

###3.3 运行命令
```
/share/work/user124/software/05.gene_analyze/paml4.8/src/mcmctree pil.2.mcmc.ctl
```

###3.4 结果
结果文件为figtree的文件，可以直接打开，会显示出分化时间以及枝长等信息
