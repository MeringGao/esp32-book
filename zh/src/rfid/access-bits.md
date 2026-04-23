# 访问控制

标签中包含访问位（access bits），用于控制标签中存储数据的访问权限。本章将探讨这些访问位的工作原理。本节可能有点复杂，所以我会尽量让它简单易懂。

> [!Caution]
> 写入访问位时要小心，错误的值可能导致扇区无法使用。

## 权限

这些是用于定义访问条件的基本权限。下表解释了每种权限操作，并指明其适用的块类型：普通数据块（读写）、数值块（value block）或扇区尾部块（sector trailer）。

| **操作** | **描述** | **适用块类型** |
|----------|----------|----------------|
| **读取（Read）** | 读取一个存储块 | 读写块、数值块、扇区尾部块 |
| **写入（Write）** | 写入一个存储块 | 读写块、数值块、扇区尾部块 |
| **增值（Increment）** | 将块内容增加并将结果存入内部传输缓冲区 | 数值块 |
| **减值（Decrement）** | 将块内容减少并将结果存入内部传输缓冲区 | 数值块 |
| **恢复（Restore）** | 将块内容读入内部传输缓冲区 | 数值块 |
| **传输（Transfer）** | 将内部传输缓冲区的内容写入块 | 数值块、读写块 |

## 访问条件

让我们来面对这个"房间里的大象"：访问条件。在研究过程中，我发现很多人对数据手册中的访问条件部分感到困惑。这里是我尝试用简单易懂的方式解释它。

你可以使用每个块 3 个比特组合来控制其权限。在官方数据手册中，这使用 CX<sub>Y</sub>（C1₀、C1₂... C3₃）这样的符号表示访问位。这个符号中的第一个数字（X）表示访问位编号，范围为 1 到 3，每个对应一种特定的权限类型。然而，这些权限的含义取决于该块是数据块还是尾部块。下标中的第二个数字（Y）表示相对块号，范围为 0 到 3。

### 表 1：扇区尾部块的访问条件

在原始数据手册中，下标编号未在表中指定。我添加了 "3" 作为下标，因为扇区尾部块位于块 3。

> [!Important]
> 如果你能读取密钥，它就不能用作认证密钥。因此，在这个表中，只要 Key B 可读，它就不能作为认证密钥。如果你注意到了，是的，Key A 永远不能被读取。

<table class="table-bordered">
  <thead>
    <tr >
      <th colspan="3" rowspan="2">访问位</th>
      <th colspan="6">访问条件对应</th>
      <th rowspan="3">备注</th>
    </tr>
    <tr >
      <th colspan="2">Key A</th>
      <th colspan="2">访问位</th>
      <th colspan="2">Key B</th>
    </tr>
    <tr >
      <th>C1<sub>3</sub></th>
      <th>C2<sub>3</sub></th>
      <th>C3<sub>3</sub></th>
      <th>读取</th>
      <th>写入</th>
      <th>读取</th>
      <th>写入</th>
      <th>读取</th>
      <th>写入</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>never</td>
      <td>key A</td>
      <td>key A</td>
      <td>never</td>
      <td>key A</td>
      <td>key A</td>
      <td>Key B may be read</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>never</td>
      <td>never</td>
      <td>key A</td>
      <td>never</td>
      <td>key A</td>
      <td>never</td>
      <td>Key B may be read</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>never</td>
      <td>key B</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>key B</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td></td>
    </tr>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>never</td>
      <td>key A</td>
      <td>key A</td>
      <td>key A</td>
      <td>key A</td>
      <td>key A</td>
      <td>Key B may be read; Default configuration</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>never</td>
      <td>key B</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>never</td>
      <td>key B</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td></td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td></td>
    </tr>
  </tbody>
</table>

**如何理解这个表？**

这是一个简单的表，显示了比特组合与权限之间的对应关系。

例如：
假设你选择 "1 0 0"（表中的第 3 行），那么你不能读取 KeyA 和 KeyB。但是，你可以使用 KeyB 修改 KeyA 和 KeyB 的值。你可以使用 KeyA 或 KeyB 读取访问位（Access Bits），但你永远不能修改访问位。

那么，这些位应该存储在哪里？我们将把它们放在第 6、7、8 字节中的特定位置，具体位置将在稍后解释。

### 表 2：数据块的访问条件

这适用于所有数据块。原始数据手册不包含下标 "Y"，我为了上下文添加了它。这里的 "Y" 表示块号（范围为 0 到 2）。

此处的默认配置表示 Key A 和 Key B 都可以执行所有操作。然而，如上一张表所示，默认配置中 Key B 是可读的，这使得它不能用于认证。因此，只能使用 Key A。

<table class="table-bordered">
  <thead>
    <tr >
      <th colspan="3">访问位</th>
      <th colspan="4">访问条件对应</th>
      <th rowspan="2">应用</th>
    </tr>
    <tr >
      <th>C1<sub>Y</sub></th>
      <th>C2<sub>Y</sub></th>
      <th>C3<sub>Y</sub></th>
      <th>读取</th>
      <th>写入</th>
      <th>增值</th>
      <th>减值/传输/恢复</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>key A|B</td>
      <td>key A|B</td>
      <td>key A|B</td>
      <td>key A|B</td>
      <td>默认配置</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>读写块</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td>读写块</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>key A|B</td>
      <td>key B</td>
      <td>key B</td>
      <td>key A|B</td>
      <td>数值块</td>
    </tr>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>key A|B</td>
      <td>never</td>
      <td>never</td>
      <td>key A|B</td>
      <td>数值块</td>
    </tr>
    <tr>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>key B</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td>读写块</td>
    </tr>
    <tr>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>key B</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>读写块</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>never</td>
      <td>读写块</td>
    </tr>
  </tbody>
</table>

注意："如果 KeyB 在扇区尾部块中可读，它就不能用于认证。因此，如果读卡器使用 KeyB 去认证一个访问条件中使用了 KeyB 的块，卡片在认证后会拒绝任何进一步的存储器访问。"

**如何理解这个表？**

它与上一张表类似；显示了比特组合与权限之间的关系。

例如：
如果你选择 "0 1 0"（表中的第 2 行）并将此权限用于块 1，你可以使用 KeyA 或 KeyB 读取块 1。但是，不能对块 1 执行任何其他操作。

其符号表示如下：块号作为下标写在比特标签后面（例如，C1<sub>1</sub>、C2<sub>1</sub>、C3<sub>1</sub>）。这里的下标 "1" 表示块 1。对于选定的组合 "0 1 0"，这意味着：
- C1<sub>1</sub> = 0
- C2<sub>1</sub> = 1
- C3<sub>1</sub> = 0

这些位也将放在第 6、7、8 字节中的特定位置，具体位置将在稍后解释。

### 表 3：访问条件表

让我们给原始表上色，以便更好地可视化每个位代表什么。每个字节中的第 7 位和第 3 位与扇区尾部块相关。第 6 位和第 2 位对应块 2。第 5 位和第 1 位与块 1 相关。第 4 位和第 0 位与块 0 相关。

符号上的上划线表示取反值。这意味着如果 CX<sub>y</sub> 的值为 0，那么 <span style="text-decoration: overline;">CX</span><sub>y</sub> 就变成 1。

<table >
  <thead>
    <tr>
      <th>字节</th>
      <th>7</th>
      <th>6</th>
      <th>5</th>
      <th>4</th>
      <th>3</th>
      <th>2</th>
      <th>1</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>字节 6</td>
      <td style="color:white;background-color:#8B0000"><span style="text-decoration: overline;">C2</span><sub>3</sub></td>
      <td style="color:white;background-color:#002D62"><span style="text-decoration: overline;">C2</span><sub>2</sub></td>
      <td style="color:white;background-color:#78184A" ><span style="text-decoration: overline;">C2</span><sub>1</sub></td>
      <td style="color:white;background-color:#234F1E"><span style="text-decoration: overline;">C2</span><sub>0</sub></td>
      <td style="color:white;background-color:#8B0000"><span style="text-decoration: overline;">C1</span><sub>3</sub></td>
      <td style="color:white;background-color:#002D62"><span style="text-decoration: overline;">C1</span><sub>2</sub></td>
      <td style="color:white;background-color:#78184A"><span style="text-decoration: overline;">C1</span><sub>1</sub></td>
      <td style="color:white;background-color:#234F1E"><span style="text-decoration: overline;">C1</span><sub>0</sub></td>
    </tr>
    <tr>
      <td>字节 7</td>
      <td style="color:white;background-color:#8B0000">C1<sub>3</sub></td>
      <td style="color:white;background-color:#002D62">C1<sub>2</sub></td>
      <td style="color:white;background-color:#78184A">C1<sub>1</sub></td>
      <td style="color:white;background-color:#234F1E">C1<sub>0</sub></td>
      <td style="color:white;background-color:#8B0000"><span style="text-decoration: overline;">C3</span><sub>3</sub></td>
      <td style="color:white;background-color:#002D62"><span style="text-decoration: overline;">C3</span><sub>2</sub></td>
      <td style="color:white;background-color:#78184A"><span style="text-decoration: overline;">C3</span><sub>1</sub></td>
      <td style="color:white;background-color:#234F1E"><span style="text-decoration: overline;">C3</span><sub>0</sub></td>
    </tr>
    <tr>
      <td>字节 8</td>
      <td style="color:white;background-color:#8B0000">C3<sub>3</sub></td>
      <td style="color:white;background-color:#002D62">C3<sub>2</sub></td>
      <td style="color:white;background-color:#78184A">C3<sub>1</sub></td>
      <td style="color:white;background-color:#234F1E">C3<sub>0</sub></td>
      <td style="color:white;background-color:#8B0000">C2<sub>3</sub></td>
      <td style="color:white;background-color:#002D62">C2<sub>2</sub></td>
      <td style="color:white;background-color:#78184A">C2<sub>1</sub></td>
      <td style="color:white;background-color:#234F1E">C2<sub>0</sub></td>
    </tr>
  </tbody>
</table>

默认访问位 "FF 07 80"。让我们尝试理解它的含义。

<table border="1">
  <thead>
    <tr>
      <th>字节</th>
      <th>7</th>
      <th>6</th>
      <th>5</th>
      <th>4</th>
      <th>3</th>
      <th>2</th>
      <th>1</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>字节 6</td>
      <td style="color:white;background-color:#8B0000">1</td>
      <td style="color:white;background-color:#002D62">1</td>
      <td style="color:white;background-color:#78184A" >1</td>
      <td style="color:white;background-color:#234F1E">1</td>
      <td style="color:white;background-color:#8B0000">1</td>
      <td style="color:white;background-color:#002D62">1</td>
      <td style="color:white;background-color:#78184A">1</td>
      <td style="color:white;background-color:#234F1E">1</td>
    </tr>
    <tr>
      <td>字节 7</td>
      <td style="color:white;background-color:#8B0000">0</td>
      <td style="color:white;background-color:#002D62">0</td>
      <td style="color:white;background-color:#78184A">0</td>
      <td style="color:white;background-color:#234F1E">0</td>
      <td style="color:white;background-color:#8B0000">0</td>
      <td style="color:white;background-color:#002D62">1</td>
      <td style="color:white;background-color:#78184A">1</td>
      <td style="color:white;background-color:#234F1E">1</td>
    </tr>
    <tr>
      <td>字节 8</td>
      <td style="color:white;background-color:#8B0000">1</td>
      <td style="color:white;background-color:#002D62">0</td>
      <td style="color:white;background-color:#78184A">0</td>
      <td style="color:white;background-color:#234F1E">0</td>
      <td style="color:white;background-color:#8B0000">0</td>
      <td style="color:white;background-color:#002D62">0</td>
      <td style="color:white;background-color:#78184A">0</td>
      <td style="color:white;background-color:#234F1E">0</td>
    </tr>
  </tbody>
</table>

我们可以从上面的表中推导出 CX<sub>Y</sub> 的值。注意只有 C3<sub>3</sub> 被设置为 1，而所有其他值都是 0。现在，参考表 1 和表 2 来理解这对应哪种权限。

<table border="1">
  <thead>
    <tr>
      <th>块</th>
      <th>C1<sub>Y</sub></th>
      <th>C2<sub>Y</sub></th>
      <th>C3<sub>Y</sub></th>
      <th>访问权限</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>块 0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>使用 Key A 拥有所有权限</td>
    </tr>
    <tr>
      <td>块 1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>使用 Key A 拥有所有权限</td>
    </tr>
    <tr>
      <td>块 2</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>使用 Key A 拥有所有权限</td>
    </tr>
    <tr>
      <td>块 3（尾部块）</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>你可以使用 Key A 写入 Key A。访问位和 Key B 只能使用 Key A 读取和写入。</td>
    </tr>
  </tbody>
</table>

由于 Key B 是可读的，你不能用它进行认证。

### 下一页的计算器

还是不太明白？使用下一页的计算器来尝试不同的组合。调整每个块的权限，观察访问位（Access Bits）的值如何相应变化。

### 参考资料

- [数据手册第 11 页](https://www.nxp.com/docs/en/data-sheet/MF1S50YYX_V1.pdf)
