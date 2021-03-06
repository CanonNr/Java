# **第一章：记忆公司**

昨晚当你的妻子感激你记得她生日还给她买了礼物时，你突然冒出一个想法。人们的记性总是很糟而你又是如此善于记忆，所以为什么不利用你的天赋干出一番事业呢？你越想越觉得这个点子不错。事实上你甚至想出一份新闻广告来解释你的想法。



记忆公司！ *-* 不用记忆，从不忘记

您是否曾为健忘而感到沮丧？不用担心，只需一通电话。当您需要记住什么时，拨打*555-55*，告诉我们您想记住的东西。例如，来电让我们知道您老板的电话号，然后忘了这事。当您想要知道已忘了的东西时，拨打同样的号码，我们将告诉您老板的电话号。费用：每次仅需*0.1*美元。

* 客户：嗨！能记一下我邻居的生日吗？

- 你：当然可以，他生日是什么时候？

- 客户：*1*月*2*日

- 你：*(*在笔记本上该客户的那一页记下*)*记好了！想知道他生日时随时打给我们！

- 客户：谢谢！

* 客气，您的信用卡将支付 0.1 美元。



# **第二章：公司扩张**

你拿到了*YC*的投资。这个点子是如此简单，只需纸笔和电话，但却像燎原之火一样发展迅猛。你每天逐渐有上百个来电。但问题也随之而来。你发现越来越多的客户要排队等你应答。他们都受够了等待。而且当你生病没法工作时，你将损失一整天的生意，更不用说那些客户的不满。于是你决定：是时候把你的妻子拉过来帮忙了。

你的计划很简单：

*1.*      你和你妻子都有分机号

*2.*      客户还是拨打原来的号码

*3.*      交换机将把客户来电发给你们俩空闲的那位。

 



# **第三章：第一次Bad Service**

就在扩张后两天，你接到了老客户*Jhon*的来电：

* Jhon：嗨！
* 你：感谢致电记忆公司，有什么可以帮您？

- Jhon：能告诉我第一次飞新德里的时间吗？
- 你：当然可以，请稍等*(*你翻阅你的笔记本，在*Jhon*那页却没发现航班这一项*)*。
- 先生，抱歉，我想您没有告诉过我们您去新德里的航班信息。
- Jhon：什么！？我昨天就打给过你们！*(*挂断了*)*



怎么可能发生这种事？*Jhon*在说谎吗？你一下子想到了原因！可能*Jhon*昨天打给了你的妻子。于是你到她桌上检查她的笔记本。没错，就是这个原因。你告诉了你妻子，她也意识到了这个问题。多么糟糕的问题！**你的分布式系统不是一致性的！****客户打给你或你妻子，再来电时却是另一个人接的，你的记忆公司没法给出一个**一致性的答复！**

 

# **第四章：修复一致性问题**

你的竞争者可能会忽视这个问题，你却不会。你妻子入睡时你想了一整晚，终于在清晨时想出了一个美丽的方案。你叫醒她说：

*“*亲爱的，这是我们从今往后要做的事：*”*

+ 不论我俩谁接到客户要求记东西的电话，打完电话前我们要告诉另一个人

* 这样我俩都能记下任何的更新

* 当客户要求查找时，我们不用互相问，因为我俩的笔记本上都记录了所有更新。

但是还是有一个问题，客户要求记东西的来电需要我们俩人同时处理，所以那段时间就没法并行工作了。例如，当你接到保存或更新请求时也会告诉我，于是我就不能接听其他来电了。但是因为多数来电都是查找的，所以也没什么大问题*(*客户更新一次然后查找多次*)*。而且，我们要不惜一切代价避免记错东西。

*“*很好！*”*你的妻子说*“*但是还有个问题你没想到。要是我俩中的一个哪天不能来工作呢？在那天我们就不能一起接受更新的来电了，因为另一个人不能记下这个更新。所以我们会有**可用性问题！例如，我接到更新来电时就没法完成这个请求，****因为即使我在我的本子上记下了，我也没法告诉你。所以我没法完成这个来电”**

 

# **第五章：最棒的解决方法**

你有点儿意识到了为什么分布式系统不像你一开始想的那样简单。想出一个既一致又可用的方案很难吗？对其他人也许很难，但对你则不是！第二天早上，你想出了一个你的竞争者们做梦都想不到的解决方法。你再一次急切地叫醒了你妻子。

*“*看！*“*你跟她说，*”*这就是我们既一致又可用的做法，方案与我昨天告诉你的很像*“*：

* 不论我俩谁接到客户要求记东西的电话，打完电话前我们要告诉另一个人，这样我俩都能记下任何的更新。

* 但是如果另一个人请假不在，我们给他发一封更新的邮件。
* 第二天他来工作时，在处理任何来电前先看一遍这些邮件，更新到他的本子上。

天才啊！你的妻子说。这个方案想不到任何的问题的。咱们就这样来吧！记忆公司现在**既是一致的又是可用的！**

 



# **第六章：妻子的愤怒**

一切都很顺利。你的系统是一致的，当你俩中的一人不能工作时也能运行良好。但是假如你们都来工作了，但是却没把更新告诉另一个人呢？想想你吵醒你妻子，说你那些所谓的伟大的点子。**要是哪天她接电话却很生气而不告诉你去更新呢？**你的方法完全毁了。到目前为止，你的方法是一致而可用的，但却不是**分区容忍的。****你可以在与你妻子和好前不接听任何来电，于是你的系统在那段时间又是不可用的了…

 

# 第七章：总结

让我们现在看一下*CAP*理论。它说：当你设计分布式系统时，你只能实现一致性、可用性和分区容忍中的两者：

* 一致性：你的客户再次来电时总能查到他们刚来电更新的信息，不论相隔多短
* 可用性：不论你和你妻子谁来工作，记忆公司总能接听来电，处理客户请求
* 分区容忍：即便你和你妻子失联，记忆公司依然能正常运转



# **奖励：最终一致性和东奔西跑的员工**

你可以雇一个员工负责当一个本子上内容更新时去更新另一个本子。最大的好处就是，他能在后台工作，你或你妻子更新时不用等待另一个完成更新。这就是许多*NoSQL*数据库系统的工作方式。一个结点进行本地更新，后台进程将更新内容同步到其他所有结点。唯一的问题是：你将会失去一定时间的一致性。例如，客户先打给你妻子，在负责更新的员工未完成更新你的本子时，客户打给了你。那么他就无法得到一个一致性的回复。但即便如此，在某些情况下倒也不算是个坏主意。例如，一个客户不会忘东西忘得这么快，一般五分钟左右才会再次来电。

*That’s CAP and eventual consistency for you in simple english :)*

这就是给您的*CAP*理论和最终一致性的极简解释：）



# **感谢**

转自 [cdai CSDN](https://blog.csdn.net/dc_726/article/details/42784237)