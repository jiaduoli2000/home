---
title: R语言-6-泊松回归实例
author: 阿狸的Blog
date: '2021-06-06'
slug: possion
categories:
  - r
tags:
  - 统计
---
# 1.模型简介
 [[泊松分布]]的方差和均值相等。当响应变量观测的方差比依据泊松分布预测的方差大时，泊松
回归可能发生过度离势。由于处理计数型数据时经常发生过度离势，且过度离势会对结果的可解释性造成负面影响。

 在泊松回归中，因变量以条件均值的对数形式ln(λ)来建模。泊松模型中的指数化参数对响应 变量的影响都是成倍增加的，而不是线性相加。

广义线性模型中除逻辑回归外最常用的回归分析模型。反应变量的期望值的对数可以用线性回归分析（即连接函数为对数函数），泊松回归也被称为对数线性回归。

# 2.适用条件
泊松回归假设结果变量服从泊松分布，通过一系列连续型和/或类别型预测变量来预测**计数型**结果变量。对于泊松回归的讨论，我们一直将响应变量局限在一个固定长度时间段中进行测量（例如，八周内的癫痫发病数、过去一年内交通事故数、一天中亲近社会的举动次数），整个观测集中时间长度都是不变的。不过，你也可以拟合允许时间段变化的泊松回归模型。

# 3.案例解析
## 1.Breslow癫痫数据
我们将讨论在治疗初期的八周内，抗癫痫药物对癫痫发病数的影响。我们就遭受轻微或严重间歇性癫痫的病人的年龄和癫痫发病数收集了数据，包含病人被随机分配到药物组或者安慰剂组前八周和随机分配后八周两种情况。
- **结果变量：****sumY（随机化后八 周内癫痫发病数）**
- **预测变量：****治疗条件（Trt）、年龄（Age）和前八周内的基础癫痫发病数 （Base）

### 1.1.分析过程
```
rm(list = ls())
##R package
require(robust)
require(ggplot2)

##1.了解数据
data(breslow.dat, package="robust")
names(breslow.dat)
summary(breslow.dat[c(6,7,8,10)])

##2.图解数据
opar <- par(no.readonly=TRUE)
par(mfrow=c(1,2))
attach(breslow.dat)
hist(sumY, breaks=20, xlab="Seizure Count",
     main="Distribution of Seizures")
boxplot(sumY ~ Trt, xlab="Treatment", main="Group Comparisons")
par(opar)

##3.拟合模型
fit <- glm(sumY ~ Base + Age + Trt, data=breslow.dat, family=poisson())
summary(fit)

##4.解释模型参数
coef(fit)
exp(coef(fit))

##5.过度离势检验
deviance(fit)/df.residual(fit)
library(qcc)
qcc.overdispersion.test(breslow.dat$sumY, type="poisson")

##6.类泊松方法解决过度离势问题
fit.od <- glm(sumY ~ Base + Age + Trt, data=breslow.dat,
                family=quasipoisson())
summary(fit.od)

##7.拟合时间段变化的泊松回归
fit <- glm(sumY ~ Base + Age + Trt, data=breslow.dat,
           offset= log(time), family=poisson)
		   
##8.零膨胀的泊松回归
##9.稳健泊松回归

```
### 1.2.数据可视
![](https://drek4537l1klr.cloudfront.net/kabacoff2/Figures/13fig01.jpg)
从图13-1中可以清楚地看到因变量的偏倚特性以及可能的离群点。初看图形时，药物治疗下 癫痫发病数似乎变小了，且方差也变小了（泊松分布中，较小的方差伴随着较小的均值）。与标 准最小二乘回归不同，泊松回归并不关注方差异质性。

### 1.3.参数解释
_解释回归系数（对数形式） ([note on p.752](zotero://open-pdf/library/items/6IWG6J5Z?page=315))_
_解释回归系数（指数化后） ([note on p.752](zotero://open-pdf/library/items/6IWG6J5Z?page=315))_
_过度离势 ([note on p.753](zotero://open-pdf/library/items/6IWG6J5Z?page=316))_
_过度离势检验P值 ([note on p.753](zotero://open-pdf/library/items/6IWG6J5Z?page=316))_
_类泊松重拟合后，P值的解释 ([note on p.754](zotero://open-pdf/library/items/6IWG6J5Z?page=317))_

## 2.儿童出生数据
这些是斐济关于曾经出生的儿童的[数据](https://data.princeton.edu/wws509/datasets/#ceb)
- **结果变量：****“y” = 曾经出生的孩子数量。
- **预测变量：****“dur”=婚姻持续时间(1=0-4，2=5-9，3=10-14，4=15-19，5=20-24，6=25-29)，“res”=居住(1=苏瓦，2=城市，3=农村)，“educ”=受教育程度(1=无，2=小学低年级，3=小学高年级，4=中学+)，“mean”=平均生育子女数(例如0.50)，“var”=生育子女的差异(例如0.50)

### 2.1.分析过程
```
rm(list = ls())
##R package

##1.了解数据
ceb <- read.table("http://data.princeton.edu/wws509/datasets/ceb.dat") 
names(ceb)

##2.图解数据
hist(ceb$y, breaks = 50, xlab = "children ever born", main = "Distribution of CEB")

##3.拟合模型
fit <- glm(y ~ educ + res + dur, offset = log(n), family = poisson(), data = ceb) summary(fit)

##4.解释模型参数
coef(fit)
exp(coef(fit))

##5.过度离势检验
require(qcc) 
qcc.overdispersion.test(ceb$y, type = "poisson")


##6.类泊松方法解决过度离势问题
fit2 <- glm(y ~ educ + res + dur, offset = log(n), family = quasipoisson(), data = ceb) summary(fit2)


```
### 2.2.数据可视
![[下载.png]]

### 2.3.参数解释
-   可见随着婚龄的增长，期望的育子数将相应增长；教育程度越高，期望育子数越低；农村预期育子数比城市高等。
-   p值为0，果然该数据存在过度离势的情况，可以用类泊松模型对数据进行分析。

## 3.高中获奖数据
学生在一所高中获得的奖项数量。获奖数量的预测因素包括学生注册的项目类型(例如，职业、普通或学术)和他们数学期末考试的分数。
- **结果变量：****“num\_wards” =高中生在一年中获得的奖项数量。
- **预测变量：****“math”=数学是连续预测变量，表示学生在数学期末考试中的分数；"prog"是分类预测变量，有三个级别，表示学生注册的课程类型。它的编码为1=“一般”，2=“学术”，3=“职业”。

### 3.1.分析过程
```
rm(list = ls())
##R package

##1.了解数据
p <- read.csv("https://stats.idre.ucla.edu/stat/data/poisson\_sim.csv") 
p <- within(p, { prog <- factor(prog, levels=1:3, labels=c("General", "Academic", "Vocational")) id <- factor(id) }) 
summary(p)

##2.图解数据
with(p, tapply(num_awards, prog, function(x) {
  sprintf("M (SD) = %1.2f (%1.2f)", mean(x), sd(x))
}))

ggplot(p, aes(num_awards, fill = prog)) +
  geom_histogram(binwidth=.5, position="dodge")

##3.拟合模型
summary(m1 <- glm(num_awards ~ prog + math, family="poisson", data=p))

##4.解释模型参数
cov.m1 <- vcovHC(m1, type="HC0")
std.err <- sqrt(diag(cov.m1))
r.est <- cbind(Estimate= coef(m1), "Robust SE" = std.err,
"Pr(>|z|)" = 2 * pnorm(abs(coef(m1)/std.err), lower.tail=FALSE),
LL = coef(m1) - 1.96 * std.err,
UL = coef(m1) + 1.96 * std.err)
r.est

##5.过度离势检验
with(m1, cbind(res.deviance = deviance, df = df.residual,
  p = pchisq(deviance, df.residual, lower.tail=FALSE)))

##6.多个模型比较
#update m1 model dropping prog
m2 <- update(m1, . ~ . - prog)
#test model differences with chi square test
anova(m2, m1, test="Chisq")

##7.回归系数指数化
s <- deltamethod(list(~ exp(x1), ~ exp(x2), ~ exp(x3), ~ exp(x4)), 
                                                coef(m1), cov.m1)
# exponentiate old estimates dropping the p values
rexp.est <- exp(r.est[, -3])
# replace SEs with estimates for exponentiated coefficients
rexp.est[, "Robust SE"] <- s
rexp.est

##8.模型预测
(s1 <- data.frame(math = mean(p$math),
  prog = factor(1:3, levels = 1:3, labels = levels(p$prog))))
predict(m1, s1, type="response", se.fit=TRUE)

##9.预测可视化
## calculate and store predicted values
p$phat <- predict(m1, type="response")

## order by program and then by math
p <- p[with(p, order(prog, math)), ]

## create the plot
ggplot(p, aes(x = math, y = phat, colour = prog)) +
  geom_point(aes(y = num_awards), alpha=.5, position=position_jitter(h=.2)) +
  geom_line(size = 1) +
  labs(x = "Math Score", y = "Expected number of awards")

```
### 3.2.数据可视
![[poissonreg-unnamed-chunk-4.png]]

### 3.3.参数解释
略。

# 4.参考文献
1.[[如何理解泊松回归]]-第四章：泊松回归
2.[[《R语言实战》 ]]-"泊松回归" ([Dangeti :750](zotero://open-pdf/library/items/6IWG6J5Z?page=313))
3.-   \[R语言中的泊松回归 - CCM的博客 | CCM Blog\](http://iccm.cc/poisson-regression-in-r/) 
4.[Poisson Regression | R Data Analysis Examples (notion.so)](https://www.notion.so/Poisson-Regression-R-Data-Analysis-Examples-894192133595478896cc0393604ef3dd)
5.[Learn to Use Poisson Regression in R – Dataquest (notion.so)](https://www.notion.so/Learn-to-Use-Poisson-Regression-in-R-Dataquest-ff9d098b1f6a4f28819957010120a953)
6.[医学生从零学习R语言进阶——临床预测模型构建\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1Kp4y1h7ZM?p=3)
