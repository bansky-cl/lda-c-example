

# lda-c 使用说明

官方主页：https://www.cs.columbia.edu/~blei/lda-c/

官方包：https://www.cs.columbia.edu/~blei/lda-c/lda-c-dist.tgz

官方readme：https://www.cs.columbia.edu/~blei/lda-c/readme.txt



## 环境需求：

`linux os，gcc=5.4.0，python2，jieba`



## step1 数据预处理

修改`preprocess.ipynb`的 `*_file` 文件名，换成自己的数据，一行一个句子或者一个文档，行号很重要，这个数据文件训练集+验证集+测试集，后面主题推理要用上其子集。

运行`preprocess.ipynb`得到符合lda-c输入 的 `ouput.txt` 和 词表 `vocab.txt`，后续查看主题词需要用上
```
数据个数，每行代表一个文档
[M] [term_1]:[count] [term_2]:[count] ...  [term_N]:[count]

一行有M个pair， 每个pair其实就是单词：词频。中间用空格分开，使用下来，当 1<词的长度<8， 直观感受好一点
```
然后词表可以去词频较高靠前的词，否则太多词频为1的词。词表文件一行一个词，行号就是词的id

## step 2 编译

下载 `lda-c-dist.tgz`

进入包目录，在shell 里面 输入 `make` 命令编译

```
$ make
gcc -O3 -Wall -g   -c -o lda-data.o lda-data.c
gcc -O3 -Wall -g   -c -o lda-estimate.o lda-estimate.c
gcc -O3 -Wall -g   -c -o lda-model.o lda-model.c
gcc -O3 -Wall -g   -c -o lda-inference.o lda-inference.c
gcc -O3 -Wall -g   -c -o utils.o utils.c
gcc -O3 -Wall -g   -c -o cokus.o cokus.c
gcc -O3 -Wall -g   -c -o lda-alpha.o lda-alpha.c
gcc -O3 -Wall -g lda-data.o lda-estimate.o lda-model.o lda-inference.o utils.o cokus.o lda-alpha.o -o lda -lm
```


## step3 训练

修改 settings.txt 的参数 alpha 那行 ,`把 fixed 换成 estimate`

```
k是主题个数，alpha是超参
lda est [alpha] [k] [settings] [data] [random/seeded/*] [directory]

example
$ ./lda est 0.001 50 settings.txt ouput.txt est_output
```
得到`final.beta`，`final.gamma`，用于推理



## step 4 推理

可以 同样修改 inf-settings.txt
```
倒数第二个参数是大小为S的子集，最后是输出保存路径
lda inf [settings] [model] [data] [name]

example
$./lda inf inf-settings.txt est_output/final ouput.txt.train inf_train
```
得到 `*.gamma` 文件，是 S x k 的一个数组，每行代表一个文档，可以找出里面最大值来判断每个文档最可能是是哪个主题



## step5 查看 主题词

```
词表文件上面已经得到，最后的参数是主题词个数
python topics.py <beta file> <vocab file> <n words>

example
python2 topics.py est_output/final.beta vocab.txt 20
```


## tips:
 - 出现段错误 Segmentation fault (core dumped)，可能是命令输错了，也可能是数据处理没做好，可自行修改preprocess.ipynb的处理逻辑
 - 数据本身决定主题效果，alpha和k是超参，需要具体调
 - 代码挺老的，但是挺快，系统编译上容易出错
