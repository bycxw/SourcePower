ref: https://zhuanlan.zhihu.com/p/597247221
# Serving
## Text Encoder
+ 将人类语言转换成机器能理解的向量。输入：人类语言，输出：语义向量（77, 768）
+ CLIPText
## Image Information Creator
+ 结合语义向量，从纯噪声开始逐步去除噪声，生成图片信息隐变量。输入：噪声隐变量（4,64,64）+语义向量（77,768），输出：去噪的隐变量（4,64,64）
+ UNet + Scheduler
## Image Decoder
+ 将图片喜喜隐变量转换为一张真正的图片。输入：去噪的隐变量（4,64,64），输出：一张真正的图片（3,512,512）
+ Autoencoder decoder

# Training
## diffusion怎么训练
1. 从训练集中选取一张加噪过的图片和噪声强度
2. 输入unet，让unet预测噪声图
3. 计算和真正的噪声图之间的误差
4. 通过反向传播更新unet的参数
## Diffusion怎么生成图片
1. 在知道噪声强度的情况下，给unet输入一张有噪声图，unet就输出有噪图上面加过的噪声
2. 把加噪后的图片减去这个噪声图，得到一张略微去噪的图片
3. 不断重复

# Control
+ CLIP模型介绍
	+ 训练集：4亿张图片及对应的标签或注释
	+ 随机取一张图片和一段文字，分别输入image encoder得到image emb和text encoder得到text emb，计算余弦相似度，预测是否匹配
	+ 将image emb和text emb训练到同一个向量空间中
+ 给图片注入语义
	+ 输入：加过噪的图片+噪声强度+text emb，输出：噪声图
	+ 没有文字时的Unet
		+ 整个unet是由一系列Resnet构成的
		+ 每一层的输入都是上一层的输出
		+ 一些输出用residual connection直接跳跃到后面去
		+ 噪声强度被转换成一个embedding向量，输入到每一个子Resnet中（噪声强度控制unet）
	+ 有文字时的Unet
		+ Resnet不再和相邻的Resnet直接相连，而是在中间新增了Attention模块。CLIP Encoder得到的语义embedding用这个Attention模块来处理