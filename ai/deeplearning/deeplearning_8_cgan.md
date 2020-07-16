# CGAN

Conditional Generative Adversarial Nets 有条件的生成对抗网络

相较于原始的GAN的无序性来说，CGAN具有更强的可控性：即，通过标签，生成指定类型的对象。

## 如何工作

 在原始GAN的基础上，让C的信息对Discriminator与Generator产生影响

![image-20200716215601098](.\dp_8_1.png)