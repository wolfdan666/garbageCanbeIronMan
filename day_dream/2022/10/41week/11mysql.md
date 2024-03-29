# 技术
2.5小时
## mysql
2.5小时
### 19章 事务简介
#### AICD
- 原子性（Atomicity），对**单一事务**保证是原子的，要么完成要么没有完成
- 隔离性（Isolation），是**多个事务之间**是隔离的，不会相互影响，也就是mysql内部执行需要做到不相互影响
- 一致性（Consistency），是**数据状态的合理性，符合现实世界的取值范围。**
我们前面介绍的原子性和隔离性都会对一致性产生影响，比如我们现实世界中转账操作完成后，有一个一致性需求就是参与转账的账户的总的余额是不变的。如果数据库不遵循原子性要求，也就是转了一半就不转了，也就是说给狗哥扣了钱而没给猫爷转过去，那最后就是不符合一致性需求的；类似的，如果数据库不遵循隔离性要求，就像我们前面介绍隔离性时举的例子中所说的，最终狗哥账户中扣的钱和猫爷账户中涨的钱可能就不一样了，也就是说不符合一致性需求了。所以说，**数据库某些操作的原子性和隔离性都是保证一致性的一种手段，在操作执行完成后保证符合所有既定的约束则是一种结果。**那满足原子性和隔离性的操作一定就满足一致性么？那倒也不一定，比如说狗哥要转账20元给猫爷，虽然在满足原子性和隔离性，但转账完成了之后狗哥的账户的余额就成负的了，这显然是不满足一致性的。那不满足原子性和隔离性的操作就一定不满足一致性么？这也不一定，只要最后的结果符合所有现实世界中的约束，那么就是符合一致性的。
- 持久性（Durability），**在没有发生事务时，每个数据状态都是持久不变的。**

#### 事务的概念
为了方便大家记住我们上面介绍的现实世界状态转换过程中需要遵守的4个特性，我们把原子性（Atomicity）、隔离性（Isolation）、一致性（Consistency）和持久性（Durability）这四个词对应的英文单词首字母提取出来就是A、I、C、D，稍微变换一下顺序可以组成一个完整的英文单词：ACID。想必大家都是学过初高中英语的，ACID是英文酸的意思，以后我们提到ACID这个词儿，大家就应该想到原子性、一致性、隔离性、持久性这几个规则。另外，设计数据库的大佬为了方便起见，**把需要保证原子性、隔离性、一致性和持久性的一个或多个数据库操作称之为一个事务（英文名是：transaction）。**

我们现在知道事务是一个抽象的概念，它其实对应着一个或多个数据库操作，设计数据库的大佬根据这些操作所执行的不同阶段把事务大致上划分成了这么几个状态：

- 活动的（active）
事务对应的数据库操作正在执行过程中时，我们就说该事务处在活动的状态。
- 部分提交的（partially committed）
当事务中的最后一个操作执行完成，但由于操作都在内存中执行，所造成的影响并没有刷新到磁盘时，我们就说该事务处在部分提交的状态。
- 失败的（failed）
当事务处在活动的或者部分提交的状态时，可能遇到了某些错误（数据库自身的错误、操作系统错误或者直接断电等）而无法继续执行，或者人为的停止当前事务的执行，我们就说该事务处在失败的状态。
- 中止的（aborted）
如果事务执行了半截而变为失败的状态，比如我们前面介绍的狗哥向猫爷转账的事务，当狗哥账户的钱被扣除，但是猫爷账户的钱没有增加时遇到了错误，从而当前事务处在了失败的状态，那么就需要把已经修改的狗哥账户余额调整为未转账之前的金额，换句话说，就是要撤销失败事务对当前数据库造成的影响。书面一点的话，我们把这个撤销的过程称之为回滚。当回滚操作执行完毕时，也就是数据库恢复到了执行事务之前的状态，我们就说该事务处在了中止的状态。
- 提交的（committed）
当一个处在部分提交的状态的事务将修改过的数据都同步到磁盘上之后，我们就可以说该事务处在了提交的状态。

随着事务对应的数据库操作执行到不同阶段，事务的状态也在不断变化，一个基本的状态转换图如下所示：

![transaction](/assets/transaction.png)

从图中大家也可以看出了，**只有当事务处于提交的或者中止的状态时，一个事务的生命周期才算是结束了。**对于已经提交的事务来说，该事务对数据库所做的修改将永久生效，对于处于中止状态的事务，该事务对数据库所做的所有修改都会被回滚到没执行该事务之前的状态。

>小贴士：贴士处纯属扯犊子，与正文没什么关系，纯属吐槽。大家知道我们的计算机术语基本上全是从英文翻译成中文的，事务的英文是transaction，英文直译就是交易，买卖的意思，交易就是买的人付钱，卖的人交货，不能付了钱不交货，交了货不付钱把，所以交易本身就是一种不可分割的操作。不知道是哪位大神把transaction翻译成了事务（我想估计是他们也想不出什么更好的词儿，只能随便找一个了），**事务这个词儿完全没有交易、买卖的意思，所以大家理解起来也会比较困难，外国人理解transaction可能更好理解一点吧～**

#### 隐式提交
建议看原文，主要的注意点是: **隐私提交后，是无法回滚到事务begin前的。**

### 第20章 说过的话就一定要办到-redo日志（上）
#### redo日志是什么
我们知道InnoDB存储引擎是以页为单位来管理存储空间的，我们进行的增删改查操作其实本质上都是在访问页面（包括读页面、写页面、创建新页面等操作）。我们前面介绍Buffer Pool的时候说过，在真正访问页面之前，需要把在磁盘上的页缓存到内存中的Buffer Pool之后才可以访问。但是在介绍事务的时候又强调过一个称之为持久性的特性，就是说对于一个已经提交的事务，在事务提交后即使系统发生了崩溃，这个事务对数据库中所做的更改也不能丢失。但是如果我们只在内存的Buffer Pool中修改了页面，假设在事务提交后突然发生了某个故障，导致内存中的数据都失效了，那么这个已经提交了的事务对数据库中所做的更改也就跟着丢失了，这是我们所不能忍受的（想想ATM机已经提示狗哥转账成功，但之后由于服务器出现故障，重启之后猫爷发现自己没收到钱，猫爷就被砍死了）。那么如何保证这个持久性呢？一个很简单的做法就是**在事务提交完成之前把该事务所修改的所有页面都刷新到磁盘**，但是这个简单粗暴的做法有些问题：

- 刷新一个完整的数据页太浪费了
有时候我们仅仅修改了某个页面中的一个字节，但是我们知道在InnoDB中是以页为单位来进行磁盘IO的，也就是说我们在该事务提交时不得不将一个完整的页面从内存中刷新到磁盘，我们又知道一个页面默认是16KB大小，只修改一个字节就要刷新16KB的数据到磁盘上显然是太浪费了。
- 随机IO刷起来比较慢
一个事务可能包含很多语句，即使是一条语句也可能修改许多页面，倒霉催的是该事务修改的这些页面可能并不相邻，这就意味着在将某个事务修改的Buffer Pool中的页面刷新到磁盘时，需要进行很多的随机IO，随机IO比顺序IO要慢，尤其对于传统的机械硬盘来说。
咋办呢？再次回到我们的初心：**我们只是想让已经提交了的事务对数据库中数据所做的修改永久生效，即使后来系统崩溃，在重启后也能把这种修改恢复出来。**所以我们其实没有必要在每次事务提交时就把该事务在内存中修改过的全部页面刷新到磁盘，只需要**把修改了哪些东西记录一下就好**，比方说某个事务将系统表空间中的第100号页面中偏移量为1000处的那个字节的值1改成2我们只需要记录一下：

>将第0号表空间的100号页面的偏移量为1000处的值更新为2。

这样我们在事务提交时，把上述内容刷新到磁盘中，即使之后系统崩溃了，重启之后只要按照上述内容所记录的步骤重新更新一下数据页，那么该事务对数据库中所做的修改又可以被恢复出来，也就意味着满足持久性的要求。因为在系统奔溃重启时需要按照上述内容所记录的步骤重新更新数据页，所以上述内容也被称之为重做日志，英文名为redo log，我们也可以土洋结合，称之为redo日志。与在事务提交时将所有修改过的内存中的页面刷新到磁盘中相比，只将该事务执行过程中产生的redo日志刷新到磁盘的好处如下：

- redo日志占用的空间非常小
存储表空间ID、页号、偏移量以及需要更新的值所需的存储空间是很小的，关于redo日志的格式我们稍后会详细介绍，现在只要知道一条redo日志占用的空间不是很大就好了。
- redo日志是顺序写入磁盘的
在执行事务的过程中，每执行一条语句，就可能产生若干条redo日志，这些日志是按照产生的顺序写入磁盘的，也就是使用顺序IO。

### 第21章 说过的话就一定要办到-redo日志（下）
看到这里: 21.1.2 redo日志文件组

**本章都非常精妙，需要通读全文才能体会，真感谢作者讲得这么细以及设计mysql的大佬这么细，666**

#### FXIED: 疑问点: redo日志是否会在内存中掉电丢失，好像是有可能的，这样就不做这个事务了吧
应该是这样的

#### redo日志在恢复的过程中，如果mysql又挂了，那怎么再次恢复呢
并且自己还想到一个问题: redo日志在恢复的过程中，如果mysql又挂了，那怎么再次恢复呢？然后看到下文得到了答案:
**是有页面结构中的lsn保证了可重入**

- 跳过已经刷新到磁盘的页面
我们前面说过，checkpoint_lsn之前的redo日志对应的脏页确定都已经刷到磁盘了，但是checkpoint_lsn之后的redo日志我们不能确定是否已经刷到磁盘，主要是因为在最近做的一次checkpoint后，可能后台线程又不断的从LRU链表和flush链表中将一些脏页刷出Buffer Pool。这些在checkpoint_lsn之后的redo日志，如果它们对应的脏页在奔溃发生时已经刷新到磁盘，那在恢复时也就没有必要根据redo日志的内容修改该页面了。
那在恢复时怎么知道某个redo日志对应的脏页是否在奔溃发生时已经刷新到磁盘了呢？这还得从页面的结构说起，我们前面说过每个页面都有一个称之为File Header的部分，在File Header里有一个称之为FIL_PAGE_LSN的属性，该属性记载了最近一次修改页面时对应的lsn值（其实就是页面控制块中的newest_modification值）。如果在做了某次checkpoint之后有脏页被刷新到磁盘中，那么该页对应的FIL_PAGE_LSN代表的lsn值肯定大于checkpoint_lsn的值，凡是符合这种情况的页面就不需要重复执行lsn值小于FIL_PAGE_LSN的redo日志了，所以更进一步提升了奔溃恢复的速度。

# 运动
10分钟

60仰卧起坐，30卧抬起

# 总结
+2.5自信币+2.5自信点