---
title: Everything I Know About SSDs 2019
authorURL: ""
originalURL: https://kcall.co.uk/ssd/index.html
translator: ""
reviewer: ""
date: "2019-3-1"
---

# 我所知道的关于固态硬盘的一切

使用 NAND 闪存的固态硬盘，它们与机械硬盘的区别，以及它们如何影响文件删除和恢复

## 介绍

我开始写这篇相当长的文章是为了自己的学习收益，当时我从 2006 年的旧款 Win 8 250GB 硬盘驱动的电脑升级到了一台带有 128GB SSD 的戴尔 Optiflex 3010。在系统盘上，我从未使用过超过 30 或 40GB 的空间，而且我既不是游戏玩家，也不是狂热的电影或音乐收藏家。至少在电脑上不是。随着我对我的新设备的不断摸索，我越来越意识到我对 SSD 中的 NAND 闪存了解甚少，不知道 SSD 是如何工作的，它们是如何读取、写入和存储数据的，以及它们使用了哪些技巧？我可以想象硬盘驱动器（HDD）在旋转的表面上写入微小的磁性记录，但 SSD 完全不同，差别巨大。

关于 SSD，还有一些长期的误解，如果能够有些思考，甚至消除其中一些，那将是很好的。也许我自己也曾怀有不少误解。不管它是如何开始的，这篇文章逐渐发展成了一篇中级技术讨论。如果你只需要知道 SSD 安静、可靠、快速，并且能够使用多年，那么就没必要再读下去了。然而，如果你认为了解如何读取 3D TLC NAND 闪存单元是有趣的，那么你别无选择，只能继续深入了解。

尽可能多的详细信息来源于企业和私人技术文章，特别是来自希捷(Seagate)和西部数据(WD)，以及名字很有趣的 Flash Memory Summit（闪存峰会）。我所做的一些结论是尝试运用我能理解的逻辑和常识。NAND 闪存控制器的复杂性，它们操作方法的多样性，以及它们发展的速度，使得理解它们，更不用说跟上它们的步伐，至少可以说是困难的。我不能说我写的内容没有混淆或者完全正确，但它更像是一本指南而不是圣经。也会有一些重复的内容。而且很快它就会过时。

我感激那些我借鉴的人，也将感激那些无需任何回报（除了贡献的满足感）就指出任何错误的人。我试图解释固态硬盘（SSD）与众不同的地方，以及为什么我们根深蒂固的硬盘驱动器（HDD）思维很难理解它。

第一个误解可能是 SSD 的复数：从语法上讲，应该是 SSDs，但 SSD's 几乎同样常见。在这里，我将坚持使用 SSD、SSDs。

## 软件和硬件

本文写于 2019 年及之后，几乎完全围绕以个人电脑或笔记本电脑存储设备形式出现的 NAND 闪存展开，我们将其称为固态硬盘 (SSD)。我不会通过提及无处不在的闪存驱动器或其他 NAND 闪存设备来使事情变得更加复杂。如果存在显著差异，我将尝试在发生差异时进行说明，但默认情况下指的是内置驱动器。**本文不涉及手机等设备中的闪存存储**。

本文的大部分细节内容是在我的电脑运行 Windows 10 家庭版系统时编写的，该系统配备了一个相当普通的内置 2.5 英寸西部数据 Green 120 GB 固态硬盘。该硬盘使用慧荣科技 SM2258XT 主控和四个 32 GB 的闪迪 05497 032G 15nm 3D TLC 闪存芯片，并内置了容量未知的 SLC 缓存。由于本文试图讨论固态硬盘作为一个整体的行为，因此使用什么主机操作系统或文件系统应该无关紧要，但在我的例子中是 Windows 和 NTFS。本文中的内容并非针对特定品牌或类型的固态硬盘，而应该是通用的。我们真正要讨论的是固态硬盘的工作原理。

我唯一使用的额外软件应用程序是 Recuvae（Piriform 出品），HexDen (HxD)。Recuva 可以列出活动文件和已删除文件及其簇分配，而 HexDen 是一款非常实用且功能强大的十六进制编辑器。Recuva 可从 www.piriform.com（现在是 https://www.ccleaner.com/） 免费获取，HexDen 也可从 www.mh-nexus.de 免费获取。我使用这两款软件的便携版（portable versions）。

所有的结论和观点完全是来自我个人的工作经历，以及从我自己的电脑上获取的任何数据。在接受这些话作为真理之前，最好验证或至少同意我的推理。这里的很多内容是对一个非常复杂主题的简化解释。

## SSD 物理内部结构

打开固态硬盘，你会有些失望，只有一块小型印刷电路板，上面有几颗 NAND 闪存芯片和一颗主控芯片，重量轻，而且有点脆弱。至于主控内部的软件，我只能概括其基本任务。主控和内存芯片一样，通常都是从外部制造商处购买的，这似乎是司空见惯的事情。固态硬盘主控软件是专有的，非常复杂，而且受到高度保护，但所有主控都必须执行基本任务，即使我们不太清楚它是如何实现的。这里只能讨论这些任务，真正聪明的调整和技巧只能由制造商知晓。我将从一些基础知识开始。

## NAND 闪存

我不打算深入探讨 NAND 闪存的内部结构，在维基百科上已经有足够多令人困惑的文章了。你真正需要知道的是，NAND（非与门）闪存利用浮动栅晶体管阵列来存储信息。浮动栅可以没有电子的充电状态，处于`空（empty）`逻辑状态，或者在不同的电压阈值下充满电子，处于表示某个值的逻辑状态。NAND 闪存是非易失性的，即使 SSD 没有通电，它也能保持其状态。哦，对了，之所以称之为 flash（闪存），是因为可以同时擦除（flashed）大量的单元。

但如果你想了解更多，继续吧。这里的术语`单元（cell）`和`晶体管（transistor）`指的是同一物理实体，可以互换使用，我不会一直重复使用 NAND 这个词。

闪存包含多个二维晶体管阵列，并支持三种基本操作：读取、编程（写入）和擦除。除了闪存阵列外，闪存芯片还包括命令和状态寄存器、控制单元、解码器、模拟电路、缓冲器以及地址和数据总线。一个单独的芯片承载着 SSD 控制器，它向闪存芯片发送读取、编程或擦除命令。在读取操作中，控制器将物理地址传递给闪存芯片，闪存芯片定位数据并将其发送回控制器。在编程操作中，数据和物理地址都会传递给芯片。在擦除操作中，只有物理地址会传递给芯片。

闪存芯片的锁存器存储从闪存阵列传输到芯片和从芯片传输出的数据，而在读取操作期间，感应放大器用于检测比特线上的电压。控制器使用状态寄存器监控发送到芯片的命令。控制器还包括错误检查和纠正（EEC）算法，来管理芯片中的错误和可靠性问题，并确保读取或写入的数据是正确的。

阵列的每一行通过字线（Word line）连接，每一列通过比特线（Bit line）连接。在行和列的交叉点是一个浮动栅晶体管，或称为单元，逻辑数据就存储在这里。字线并行连接到晶体管，比特线串行连接。比特线的端部连接到感应放大器。

闪存阵列被划分为块，块又被划分为页。在一个块内，与每条字线相连的单元构成一个页。与位线相连的单元则表示块中的页数。常见的页面大小为 4k、8k 或 16k，128 到 256 个页面组成一个 512k 到 4mb 大小的块。页是芯片控制单元可寻址的最小数据粒度。

读取或编程操作涉及芯片控制器使用块解码器选择相关块，然后使用页解码器选择块中的页。芯片控制器还负责激活正确的模拟电路，以产生编程和擦除操作所需的电压。

虽然每行的单元格数名义上与页面大小相等，但每行的实际单元格数却高于每页的标称容量。这是因为每个页面都包含一组备用单元和数据单元。备用单元存储该页面的 ECC 位以及页面的物理地址到逻辑地址映射。控制器还可在备用区中保存有关页面的其他元数据信息。在读取操作过程中，整个页面（包括备用区中的位）都会传输到控制器。控制器中的 ECC 逻辑会检查并纠正读取的数据。在编程操作期间，控制器会将用户数据和 ECC 位传送到闪存。

系统启动时，控制器会扫描整个闪存阵列中每个页面的空闲区域，将逻辑地址到物理地址的映射加载到自己的内存中（控制器中可能有其他保存映射数据的技术）。控制器在闪存转换层（FTL）中保存逻辑地址到物理地址的映射。FTL 还执行垃圾回收，清除写入后的无效页面，并执行损耗均衡，确保所有闪存块都得到均衡使用。

由于闪存不支持原地更新，所以在内容可以被编程之前，页面需要被擦除；但不同于以页面粒度进行的编程或读取操作，擦除操作是以块粒度执行的。

## 2D 和 3D, 和 Layers（层）

在闪存架构中，一个平面闪存块，也就是一个二维的存储单元阵列，被称为 2D 闪存。如果一个或多个这样的阵列叠放在一起，那么就是 3D 闪存。3D NAND 闪存建立在一块芯片上，最多可达 32 层，是为了在平面闪存达到尺寸限制时降低成本而设计的：3D 闪存的生产成本几乎与 2D 相当，但存储容量大大增加。在 2D 和 3D 中，每个页面（即行）中的存储单元由单词线连接，而页面中的每个偏移量（即列）中的存储单元则通过位线连接（简单地说）。

3D 闪存与层叠式闪存不同，后者是将多个独立的超薄芯片堆叠在一起。这种做法成本高得令人望而却步。大多数现代消费级固态硬盘（2010 年代）都使用 3D TLC 闪存。

## 我能看到其中一个单元吗？

终端用户闪存的单元尺寸极小，通常为 15 纳米，范围从 43 纳米到 12 纳米不等。实际上，单元尺寸或单元直径具有误导性，因为所述尺寸并非单元任何尺寸的测量值，而是芯片上离散组件之间距离的度量。芯片上的硅层厚度约为 0.5 至 3 纳米：相比之下，氢原子的直径为 0.1 纳米，而芯片制造中使用的硅原子为 0.2 纳米。一纳米 (nm) 确实非常小，仅为一米的十亿分之一，打个比方，如果一毫米相当于一颗标准弹珠的大小（约 13 毫米），那么一米就相当于地球的大小。十亿倍的威力令人印象深刻。

## SLC, MLC, TLC, QLC 及其未来发展

单级单元（SLC）有一个电子电荷阈值来指示一个位的状态，即一或零。多级单元（MLC）存储一个电压来表示两个位的状态，有三个不同的阈值分别代表 11、10、00 和 01。三级单元（TLC）存储三个位的状态，包括 111、110、100、101、001、000、010 和 011。如果有人感兴趣的话，可以推导出四级单元（QLC）使用的 15 个阈值。（我见过其他版本的阈值表示位的含义。）

不幸的是，当双级单元被开发出来时，它被称为多级单元，并被赋予了 MLC 这个缩写，这迫使每个人在想要指代多个级别的存储单元时都要费力地打出“多级单元”这个词。如果它被称为双级单元，我们就可以自由地使用 DLC、TLC 和 QLC，并用 MLC 来描述所有这些，但现在为时已晚。如果闪存只停留在 SLC 阶段，有其是/否、一/零的状态，这些解释将更容易写，希望也更容易理解。

在多级单元中，物理 NAND 页面代表两个或更多的逻辑页面。属于 MLC 的两个位分别映射到两个逻辑页面。奇数编号的页面（包括零）映射到最不重要的（右手）位，偶数编号的页面映射到最重要的（左手）位。同样，属于 TLC 的三个位分别映射到三个逻辑页面，而 QLC 映射到四个逻辑页面（TLC 和 QLC 的页面编号是未知的）。

多层单元需要支持的位数越多，其性能就越受影响。对于 SLC，控制器只需要检查是否超过了一个阈值。而 MLC 单元可以有四个值，TLC 有八个，QLC 有 16 个。为了读取单元的正确值，SSD 控制器需要使用精确的电压和多次读取来确定单元中的电荷。很明显，如果一个物理页面支持多个逻辑页面，那么该页面的读写频率将高于 SLC 页面，从而影响其使用寿命。此外，显而易见的是，TLC SSD 只需要 SLC 设备三分之一的物理单元，因此我的 120 GB TLC SSD 实际上只包含 40 GB 的 NAND 单元。

高使用率的企业级 SSD 曾经是 SLC 的天下，因为它具有更高的速度、耐用性、可靠性和读/写能力，而 MLC 和 TLC 也逐渐被企业级应用所接受。终端消费级 SSD 市场则获得了更便宜、容量更大，但速度更慢、更脆弱的多层单元。

**为什么空代表一？**

任何仍在关注这一点的人可能已经注意到，单层和多层单元都有一个共同点，那就是一个空的单元格（其中浮栅没有电荷）代表一。与 HDD 不同，HDD 中任何位模式都可以写入任何位置，而 SSD 页面上存在一个默认的逻辑状态 1。这是因为单元上只有一个编程功能，即跨浮栅移动电子。NAND 闪存单元只能被编程为零状态，而没有能力编程为一。对于多层单元，所有页面的默认值仍然为 1，但即使在单元格被编程并且栅极上有电子存在之后，逻辑 1 仍然可以表示。

自从斐波那契在 1202 年将带有零概念的印度-阿拉伯数字系统引入欧洲数学以来，人类的思维就将零与空联系在一起，将一与满联系在一起。空着却代表一，这让人相当困惑，而且似乎主要来自于惯例（空状态*可以*代表零，但需要在数据线上使用反相器）。可能是电路不那么复杂，也可能是空单元格传导电荷的能力意味着它是一。

## 它们都是 SLC

在讨论了这么多之后，或许有必要强调一点：无论预期用途如何，所有 NAND 闪存的物理结构都是 SLC（单层存储）。即使你能观察 TLC（三层存储）单元的内部，你也不会看到 101、011 或任何类似的数字。一个单元格中永远只可能存在一定数量的电子，无论这个数量如何解释。SSD 控制器知道应该将单元格视为 SLC、MLC 还是其他类型，并相应地对其进行编程、测量电子数量，并确定它代表的逻辑值。但即使是四层单元格也只能包含一个值，就像 SLC 单元格一样。

## 迷思与误解

现在我们来谈谈迷思、误解以及撰写本文的真正原因：当 SSD 页面被读取、写入和重写时会发生什么？这将如何影响已删除文件的恢复？一方面，我们有 NTFS，它专为 HDD 设计，远早于 SSD 的普及；另一方面，我们有 NAND 闪存，它有其独特的运作方式；此外，还有数十亿人多年来已经习惯了 HDD 的使用方式和预期。在这里，如果没有特别说明，我将交替使用 SSD 和 NAND 闪存，尽管这样说并不完全准确。

## 存储设备控制器

所有 HDD、SSD 和闪存驱动器都有一个内部控制器。用微软的话来说，控制器使得存储设备能够对主机进行“抽象化”。这种抽象是通过逻辑块寻址（LBA）实现的，其中存储设备上每个能够被寻址的簇都由主机通过一个递增的数字（LBA）来识别。存储设备控制器将该数字映射到设备上的扇区或页面。对于主机来说，这种映射是恒定的，一个簇会一直映射到同一个 LBA，直到主机更改它。在 HDD 上，这种关系是物理的、固定的：简单来说，HDD 控制器只是读取和写入主机要求的任何扇区。它不需要考虑以前那里有什么，它只是按照指令行事，将新数据写入旧数据之上。它之所以这样做，是因为它可以这样做，没有什么可以阻止新的簇直接写入旧簇的相同扇区之上。而在 SSD 上，情况就不同了。

对于 SSD，主机仍然使用 LBA 寻址系统，并不断协调 LBA 和簇号之间的关系。它知道该设备是一个 SSD，并有一些技巧来适应这一点，但这些技巧将在后面介绍。然而，SSD 控制器有许多技巧来协调为 HDD 编写的文件系统与 NAND 闪存的需求。

## 闪存转换层（FTL）

主机仍然使用 LBA 寻址来对 SSD 进行读写操作，因为它不知道其他方式。这些命令会被 SSD 控制器上的闪存转换层 (FTL) 拦截。FTL 维护着一个 LBA 到物理块地址 (PBA) 的映射表，并将转换后的 PBA 传递给控制器。之所以需要这个映射表，是因为与 HDD 不同，LBA 到 PBA 的关系是易失的。之所以易失，是因为 NAND 闪存的写入数据方式。

一个空的页面，所有单元都未充电，默认情况下包含全 1。然而，如果使用十六进制编辑器查看 SSD 的空扇区，它将显示为一组零。这是因为空页面没有分配给 LBA/PBA 映射表。相反，如果对一个空页面发出读取请求，则会返回一个默认的零页面。这适用于未分配的簇和属于文件的簇：SSD 不会分配一个页面并将所有单元从 1 改为 0。

## 浮栅晶体管

在深入了解读取和写入操作之前，这一部分可能会有所帮助，在这里，单元格和（FG）晶体管可以互换使用（一个单元格就是一个晶体管）。如果想了解更多关于浮栅金属氧化物半导体场效应晶体管（Metal-Oxide-Semiconductor Field-Effect Transistors，简称 MOSFET）的信息，可以随时查阅维基百科。

浮栅 MOS 晶体管有三个端子：栅极、漏极和源极。当一个电压被施加到栅极上时，电流就可以从源极流向漏极。施加到栅极的低电压会导致从源极到漏极的电压按比例变化。当电压升高时，这种比例响应会停止，不管怎样栅极都会关闭。

浮栅中的电荷会改变晶体管的电压阈值，即栅极关闭的点。当栅极电压高于某个值，大约 0.5 伏时，栅极总是会关闭。当电压低于这个值时，栅极的关闭由浮栅电压决定。

如果浮栅没有电荷，那么施加到栅极的低电压就会关闭栅极，并允许电流从源极流向漏极。如果浮栅带有电荷，那么需要施加更高的电压到栅极上，栅极才会关闭并且电流才会流动。浮栅中的电荷改变了为了使栅极关闭并导电所必须施加到栅极上的电压。

## SSD 读取

NAND 闪存的设计并没有固有的特性来阻止对单个单元格的读取和写入。然而，为了符合 NAND 闪存设计的目标——即简单和小巧——NAND 芯片接受的标准命令被构造成页面是最小的可寻址单元。这样就省去了存储额外指令和单元格到页面映射所需的空间。

要读取单个页面及其内部的单元格，需要将该页面与块内的其他页面隔离开来。为此，未被读取的页面会被临时禁用。

同一块内的同一行（一个页面）的所有单元格/晶体管通过字线（Word Line）并联连接到晶体管的栅极。同一列（单元格偏移）的所有晶体管串联在一起，通过位线（Bit Line）连接，将一个的漏极与下一个的源极相连。在每个位线的末端是一个感应放大器。进行读取操作时，除了正在读取的页面外，所有字线上都会施加一个穿透电压。穿透电压接近或高于可能的最高阈值电压，并迫使*所有未被读取页面中的*晶体管关闭，不管它们是否存储了电荷。所有位线都会通入低电流。

正在读取的页面的字线会被给予一个参考电压，所有位线感应放大器进行读取。储存了足够电子数量的晶体管不会被参考电压关闭，位线电流不会通过源/漏链传递到感应放大器。没有电荷，或电荷低于阈值的晶体管会被参考电压关闭，并将位线电流传导到感应放大器。为了确定多级单元格的逻辑状态，需要进行多次不同阈值电压的读取。

（为了增加或避免更多的混淆，浮栅晶体管可以是开启或关闭状态。开启状态的栅极不导电，关闭状态的栅极则导电。所以如果栅极是开启的，就没有电流能通过；如果关闭了，就能通过。难怪我们会感到困惑。）

由此可见，要读取块中的一个页面，块中每个页面的所有晶体管都需要接收一个穿透电压或一个或多个参考电压。即使块中的其他页面部分或全部为空，这一点似乎仍然适用。这在下面的读取干扰中变得很重要。

## 读取存储

理解 SLC 读取背后的概念相当容易。SLC 闪存只应用一个阈值，因此只需要一个测试电压——浮栅要么关闭，要么不关闭。如果阈值电压使栅极关闭，则位线电流流向读出放大器，存储值为 1。如果没有，则为 0。

多级单元则不同，存储值位序背后的原因也变得明显。在 MLC 中，可能的用户信息位组合为 11、10、00 和 01，由三个阈值分隔。读取最高有效位 (l/h) 只需要读取一次中间阈值电压。如果栅极关闭，则最高有效位为 1，如果未关闭，则最高有效位为 0，无论最低有效位是什么。要读取最低有效位 (r/h)，需要读取两次，一次读取阈值一，一次读取阈值三。如果读取阈值一使栅极关闭，则最低有效位设置为 1，不需要读取第二次。如果栅极打开，则读取阈值三。如果它关闭，则最低有效位设置为 0。如果没有，则值为 1。

TLC 单元中的位组合为 111、110、100、101、001、000、010 和 011，由七个阈值分隔，更难以理解。与 MLC 一样，最高有效位仍然只需要读取一次中间阈值。中间位需要读取两次，分别在阈值二和六，最低有效位需要读取四次，分别在阈值一、三、五和七。

所有基于最高有效位 (l/h) 的多单元页面都被视为 SLC，只需要读取一次即可确定用户信息位值。

## SSD 写入

NAND 闪存最重要的方面，也是 HDD/SSD 路径上最大的分叉点，以及接下来所有内容的基础和关键因素是，数据只能写入空的 SSD 页面。这并不是什么新鲜事，也不是什么未知的事情，但它对数据安全和恢复有着最大的影响。

虽然 SSD 可以读写单个页面，但它们不能覆盖页面，因为将 0 变回 1 所需的电压会损坏相邻的单元。所有写入和重写都需要一个空白页。与 HDD 不同，HDD 会将完整的簇写入磁盘，无论之前有什么内容，写入 SSD 页面的行为是分配一个默认值为全 1 的空白页，并对需要更改为 0 的单元施加电荷。这对多级单元和 SLC 都是一样的，因为无电荷全 1 的模式要么被替换为代表另一种模式的电荷，要么保持不变。这是一个一次性的过程。

当发出写入请求时，会分配一个空页面（通常在同一个块内），然后写入数据。FTL 中的 LBA/PBA 映射会更新，以便将新页面分配给相关的 LBA。LBA 对主机来说始终保持不变：无论分配哪个页面，主机都不会知道。如果用户数据被重写或分配了一个新文件，这个过程是相同的：唯一的区别是重写的工作量会稍微多一些。旧页面将被标记为无效，主机将无法访问，但仍会占用其块内的空间，因为它无法被重用。

虽然很容易理解写入 SLC 页面的过程，但多级单元页面的情况更难想象。控制器会将新的写入累积到 SSD 缓存中，直到收集到足够多的逻辑页面来填充一个物理页面，然后才会写入物理页面。这需要对页面进行最少的写入次数。如果修改了多级页面中的一个逻辑页面，则需要分配一个新页面并重写所有逻辑页面，因为物理页面中的各个值无法更改。如果删除了一个逻辑页面，那么我推测该逻辑页面会被标记为无效，当该块成为垃圾回收的候选对象时，任何有效的逻辑页面都会在写入之前进行合并。换句话说，一个多级页面，或者至少是其中的大部分，将始终包含一整套逻辑页面。

很明显，如果 NAND 闪存以这种方式处理数据写入（事实也确实如此），那么 SSD 最终将充满有效和无效的页面，性能也会逐渐降低到爬行的程度。虽然单个 SSD 页面无法擦除，但一个块可以擦除，这种方法用于将块恢复到可写状态。为了加快这一过程，并确保始终有一组空块可用于写入，SSD 控制器会使用垃圾回收。

## 垃圾回收

从最简陋到容量最高的 SSD，垃圾回收功能都已启用：如果没有它，NAND 闪存将无法使用。垃圾回收是 SSD 控制器的一部分，主机并不知道它的工作。在最简单的形式中，GC 会选取一个包含有效和无效页面的块，将有效页面复制到一个新的空块中，更新 LBA 映射表，并将旧块放入无效块池中。在那里，该块及其页面将被重置为空状态，并将该块添加到可用块池中。因此，应该始终有一组可用块可用于写入活动。只要 SSD 通电，GC 就会执行其工作，无法停止。GC 例程有各种复杂的技术，所有这些技术都是专有的，而且主要只有制造商才知道。

当一个全新的 SSD 从工厂出厂时，写入操作会以渐进的线性模式逐渐填满驱动器，直到可寻址存储空间被完全写入。然而，一旦垃圾回收开始，数据写入的方式（顺序写入还是随机写入）就会影响性能。顺序写入的数据会写入整个块，当数据被替换时，整个块会被标记为无效。在垃圾回收期间，不需要将任何数据移动到另一个块。这是最快的垃圾回收方式，也就是说，没有垃圾需要回收。当数据被随机写入时，无效页面会分散在整个 SSD 中。当垃圾回收作用于包含随机写入数据的块时，必须将更多数据移动到新块，然后才能擦除该块。

## 垃圾回收难题

垃圾回收可以在后台进行（当主机空闲时），也可以在前台进行（当写入需要时）。虽然后台垃圾回收看起来更可取，但它也有缺点。如果主机在空闲时使用节能模式，垃圾回收要么会等待设备重新启动，从而导致用户延迟以完成垃圾回收，要么会唤醒设备并缩短电池寿命，而此时主机处于“空闲”状态。此外，垃圾回收并不知道它正在回收的数据。不可避免的是，一些数据在进行垃圾回收后不久就会被删除，从而导致另一轮垃圾回收和随之而来的额外和不必要的写入（写入放大，即实际写入与数据写入的比率）。前台垃圾回收看似与性能背道而驰，但它避免了节能问题，只在实际需要时才进行写入，并且借助快速缓存和高度发达的垃圾回收算法，不会给用户带来明显的性能损失。现代垃圾回收的趋势似乎是前台回收，或者前台和后台回收的组合。

基于前台垃圾回收，以及大多数用户活动都是随机的，那么不可避免的结论是，SSD 将在其大部分生命周期中处于满容量状态，如果我们指的是可用块，即使分配给主机的空间看起来很低。

然而，SSD 还存在另一个潜在问题，这与一个历史事件有关：文件系统的设计方式。

## 文件系统 - 非眼见为实

主机文件系统的设计可以追溯到 HDD 称霸的时代，因为 SSD 那时还没有以可用且价格合理的形态出现。文件系统没有考虑到 NAND 闪存的需求。文件在不断地更新：它们被分配、移动和删除，大小也在增长和缩减。文件系统处理这些操作的方式与 NAND 闪存的工作机制不兼容。

值得强调的是，存储设备对主机操作系统是抽象的。虽然资源管理器以人类完全可以理解的形式显示了一系列文件夹和文件，但这只是一种假象。资源管理器显示的是一个完全由文件系统表中的元数据创建的逻辑结构。存储设备控制器对文件、文件夹、表或操作系统一无所知：HDD 或 SSD 看到的只是读取或写入特定扇区的命令，而它们也忠实地执行了这些命令。然而，SSD 与 HDD 相比有一个优势，它知道哪些页面保存着数据并映射到 LBA，哪些页面是空的、不保存有效数据且未映射到 LBA。相反，HDD 不需要知道这些，对 HDD 来说，所有扇区都是一样的。

## 文件删除

在 NTFS 文件系统中，当一个文件被删除时，主文件表 (MFT) 中的对应条目会被标记为已删除，并且集群位图也会被修改，将该文件占用的集群标记为可重用。整个删除过程只在 MFT 和集群位图中进行。这对 HDD 来说完全足够了，因为 NTFS 可以随时重用 MFT 条目和集群。

在 SSD 上，从 NTFS 的角度来看，这个过程完全相同，因为 NTFS 没有其他删除文件的方法。然而，SSD 看到的只是 HDD 看到的内容，即对几个页面的更新。HDD 和 SSD 都不知道正在更新的是 MFT 和集群位图，因为它们对这些东西一无所知。由于已删除文件的集群上没有任何活动，因此 SSD 中保存这些集群的页面仍然映射到 FTL 中的 LBA。SSD 的 FTL 无法知道这些页面不再被 NTFS 分配：对 SSD 来说，这些页面仍然有效，不会被垃圾回收清理。

由于这些“死”页面被分配给了 LBA，因此当分配或扩展文件并且主机使用该 LBA 时，它们可能会被释放。在这种情况下，该页面将被标记为无效，并使用一个新页面。然而，最终不可避免地会出现大量未被标记为垃圾回收的未使用和不需要的数据，这些数据会被 SSD 控制器毫无意义地维护，并且无法重用。为了克服这个问题，并将主机对已分配和未分配页面的视图与 SSD 的视图关联起来，从 Windows 7 开始，NTFS 获得了 TRIM 命令。

## SSD Detection

Although the storage device is abstracted from the File System, to enable some of the file system's SSD tweaks it needs to know whether the device is an HDD or SSD. There are various ways to do this, including querying the rotational speed of the device, which on an SSD should be zero (or perhaps one). This seems the most widely used and most proficient method.

## TRIM

TRIM (it isn't an acronym) is a SATA command sent by the file system to the SSD controller to indicate that particular pages no longer contain live data, and are therefore candidates for garbage collection. TRIM is only supported in Windows on NTFS volumes. It is invoked on file deletion, partition deletion, and disk formatting. TRIM has to be supported by the SSD and enabled in NTFS to take effect. The command 'fsutil behavior query disabledeletenotify' returns 0 if TRIM is enabled in the operating system. It does not mean that the SSD supports it (or even if an SSD is actually installed) but all modern SSDs support a version of it.

There are three different types of TRIM defined in the SATA protocol and implemented in SSD drives. Non-deterministic TRIM: where each read command after a TRIM may return different data; Deterministic TRIM (DRAT): where all read commands after a TRIM return the same data (i.e. become determinate) and do not change until new data is written; and Deterministic Read Zero after TRIM (DZAT): where all read commands after a TRIM return zeroes until the page is written with new data. By the way whilst DRAT returns data on a read it is not the userdata that was ptrviously there before the TRIM: it is random.

Fortunately Non-Deterministic TRIM is rarely used, and Windows does not support DRAT, so a read of a trimmed page - which is easily done with a hex aditor - invokes DZAT and returns zeroes immediately after the TRIM command is issued. The physical pages may not have been cleaned immediately following the TRIM command, but the SSD controller knows that there is no valid data held at the trimmed page address.

TRIM tells the FTL that the pages allocated to specific LBAs are to be classed as invalid. When a block no longer has any free pages, or a specific threshold is reached, the block is a candidate for garbage collection. Live data is copied to a new empty block, and the original block is erased and made available for reuse.

TRIM is an asynchronous command that is queued for low-priority operation. It does not need or send a response. The size of the TRIM queue is limited and in times of high activity some TRIM commands may be dropped. There is no indication that this takes place, so some unwanted pages may escape garbage collection.

## RETRIM

Windows Defragger - now called Storage Optimiser - has an option to Optimise SSDs. This does not defrag the SSD but sends a series of TRIM commands to all unallocated pages identified in NTFS's cluster bitmap. This global TRIM (or RETRIM) command is run at a granularity that the TRIM queue will never exceed its permitted size and no RETRIM commands will be dropped. A RETRIM is run automatically once a month by the storage optimiser.

## Over-provisioning

All NAND flash devices use over-provisioning, additional capacity for extra write operations, controller firmware, failed block replacements, and other features utilised by the SSD controller. This capacity is not physically separate from the user capacity but is simply an amount of space in excess of that which can be allocated by the host. The specific pages within this excess space will vary dynamically as the SSD is used. According to Seagate, the minimum reserve is the difference between binary and decimal naming conventions. An SSD is marketed as a storage device and its capacity is measured in gigabytes (1,000,000,000 Bytes). NAND flash however is memory and is measured in gibibytes (1,073,741,824 bytes), making the minimum overprovisioning percentage just over 7.37%. Even if an SSD appears to the host to be full, it will still have 7.37% of available space with which to keep functioning and performing writes (although write performance will be diabolical). Manufacturers may further reduce the amount of capacity available to the user and set it aside as additional over-provisioning, in addition to the built-in 7.37%. Additional over-provisioning can also be created by the host by allocating a partition that does not use the drive's full capacity. The unallocated space will automatically be used by the controller as dynamic over-provisioning.

My humble WD SSD has four 32 gb chips but a specified capacity of 120 gb, meaning that it has 8 gb set aside as additional over-provisioning. Add this to the 7.37% minimum (9.4 gb) and the 17.4 gb equates to almost 15% over-provisioning space.

## Wear Levelling

Some files are written once and remain untouched for the rest of their life. Others have few updates, some very many. As a consequence some blocks will hardly ever see the invalid block pool and have a very low erase/write count, and some will be in the pool every few minutes and have a very heavy count. To spread the wear so that all blocks are subject to erase/writes equally, and the performance of the SSD is maintained over its life, wear levelling is used. Wear levelling uses algorithms to identify blocks with the lowest erase count and move the contents to high erase count blocks; and to select low erase count blocks for new allocations. As with garbage collection, wear levelling is far more complex than I could possibly deduce, let alone explain.

## Read Disturbance

SSD reads are not quite free, there is a price to pay. As described above, a read of one page generates a pass-through voltage on all other cells in the block. This voltage is likely to be below the highest threshold value that could be held by the cell, but it still generates a weak programming effect on the cells, which can unintentionally shift their threshold voltages. The pass-through voltage induces electric tunnelling that can shift the voltages of the unread cells to a higher value, disturbing the cell contents. As the size of flash cells is reduced the transistor oxide becomes thinner and in turn increases this tunnelling effect, with fewer read operations required to neighbouring pages for the unread flash cells to become disturbed, and move into a different logical state. Cells holding lower threshold values are more susceptible to read disturbance.

Thus each read can cause the threshold voltages of other unread cells in the same block to shift to a higher value. After a significant amount of reads this can cause read errors for those cells. A read count is kept for each block and if it is exceeded the block is rewritten. The count is high for SLC cells, around 1m, lower for 25 nm MLC at around 40,000, and much lower for 15 nm TLC cells.

## File Recovery

And now we come to deleted file recovery. NTFS goes through exactly the same process to delete a file on an SSD as it does on an HDD, with the exception of the additional TRIM command. And the TRIM command (assuming it's executed) and a few SSD quirks destroys any practicable chance of deleted file recovery.

TRIM commands, as described above, have a complimentary setting within the SSD controller in the form of DRAT and DZAT. (I don't believe that non-deterministic TRIM is used in any reputable SSD, and I don't think that Windows supports DRAT, but I have no proof.) The implementation of DZAT means that immediately on successful execution of the TRIM command (which will in most cases be immediately on file deletion) any attempt to read the TRIMed page will return zeroes. The data on the page will still exist until the block is processed by the garbage collector, but that data is not accessible from the host by any practicable means, or any general software.

Garbage collection is independent of the host device and will be invoked at the will of the SSD's controller. Once the process is started it cannot be stopped, apart from powering off the SSD. Once powered up again the garbage collector will resume its duties to completion.

Deleted file recovery on a modern SSD is next to impossible for the end user, and under Windows as close to impossible as you can get. A theoretical examination of the chips would most likely show compressed and encrypted data, striped over multiple blocks, and no possibility of relating one page of data to another across the multiple millions of pages. There is a very small possibility of recovering recently deleted files by powering off the SSD immediately and sending it to a professional data recovery company. They may recover some data, given enough time and money.

After a session of file deletion, such as running Piriform's CCleaner, run Recuva on the SSD. The headers of the deleted files found (and presumably the rest of the file) will all be zeroes. This is TRIM and DZAT doing their work in a few seconds, killing any chance of deleted file recovery. Of course TRIM can be disabled, at the cost of performance, but it's probably better to be a little less cavalier when deleting files that might be wanted later.

## Deleted File Security

The notion of secure file deletion - overwriting a file's data before deletion - is irrelevant, and if any other pattern except zeroes is chosen is just additional and pointless wear on the SSD. Even overwriting with zeroes will cause transaction log and other files to be written, so secure file deletion on an SSD should never be used. Wiping Free Space is far worse for pointless writes, and is even more futile than secure file deletion. The deleted files just aren't there any more.

## The OCZ Myth

Some years ago (as a little light relief to all these acres of text) the OCZ forums were buzzing with the latest method of regaining performance on their SSDs: run Piriform's CCleaner Wipe Free Space, with one overwrite pass of zeroes. Although performance may have been regained, logic, and common sense, went out of the window. The theory was that overwriting the pages with zeroes was equivalent to erasing blocks (this was before the days of TRIM). This was nonsense, and should have been apparent from the start. The default state of an empty page is all ones, not zeroes, and how could a piece of software possibly erase NAND flash?. The real reason was that as CCleaner was filling the pages with zeroes the SSD controller simply unmapped the pages and showed default pages of zeroes to the host. The invalid pages were then candidates for garbage collection, which gave a much greater pool of blocks to call upon on writes, and hence a better performance. A sort of RETRIM before that was invented.

## SSD Defragmentation

One of the SSD mantras is that an SSD should never be defragged. Whilst there is little (there is a little) to be gained from rearranging clusters into adjacent pages - an SSD has no significant overhead in random reads - an SSD defrag is not entirely verboten. In fact from Windows 8 onwards the Storage Optimiser will defrag an SSD if certain conditions are met. If System Restore is enabled, the fragmentation level is above 10%, and at least one month has passed since the last defrag, Windows Storage Optimiser Scheduled Maintenance will defrag the SSD. This is what Microsoft calls a Traditional Defrag, it is not an Optimise (RETRIM). The defrag is required to reduce the extents on the volume snapshot files when system restore is enabled.

There is nothing to be afraid of in a monthly defrag. Most users won't hit the 10% fragmented criteria so a simple RETRIM will be run, and Windows 10 users won't get defragged anyway (System Restore is disabled in Widows 10 by default). The reduction in life of an SSD will not be noticed. Furthermore, although SSDs are not fazed by random reads, files do get fragmented and that means a significant increase in I/Os. An occasional clearup is a boon.

**SSD Lifetime:** There are many users worried about the life expectancy of their SSDs. Yes, continuous write/erase cycles, and the added and unseen write amplification, do take a toll on the life of NAND flash. Using an SSD does wear it out. My WD Green 120 gb SSD, a TLC SSD from a reputable manufacturer but at the very lowest cost, has an estimated life of 1 million+ hours and a write limit if 40 terabytes. One million hours is 114 years, so we can forget that. As for writes, at 1 gb a day - far more than my current rate of data use - it would take the same 114 years to reach 40 tb. Even with massive write overhead this SSD is not going to wear out in the foreseeable future. If all 128 gib of available flash is used equally, the 40 tb equates to 312 writes per cell, a very conservative number.

## The End

The only thing to add is that NAND flash, SSDs, and especially SSD controllers, are far more sophisticated, complex and incomprehensible than what has been written here, what I know, what I could possibly comprehend, and what I could possibly explain. I should also add secret, as their software is proprietary. Whilst an HDD is a marvel of complex electro-mechanical engineering at a ridiculously low cost, the SSD is an equally marvellous and complex piece of electronics and software at a minimally higher cost. We should be thankful for both.

You can return to my home page [_here_][1]

If you have any questions, comments or criticisms at all then I'd be pleased to hear them: please email me at kes at kcall dot co dot uk.

© Webmaster. All rights reserved.

[1]: http://kcall.co.uk/
