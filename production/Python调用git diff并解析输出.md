# Python调用git diff并解析output

## 应用场景

git diff命令会生成一个output，如下图所示：

<img src="https://cdn.jsdelivr.net/gh/ArCyanic/Gener/20220810032739.png" style="zoom:50%;" />

本程序可以利用Python，处理git仓库中的文件，最终得到存在commit记录的两个日期的任意文件的差异数据。

其返回结果的结构如下：

```python
diffs: Dict[str, List] = {'add': [], 'delete': [], 'former': [], 'latter': []}
```

四个属性分别表示信息如下：

- add: 全新增加的行
- delete：彻底删除的行
- former：部分改变之前的行
- latter：部分改变之后的行

## 关键代码

- os.chdir() 与 os.system()

  os.system() 能够执行终端命令，在本例中的需求是执行git命令，例如：

  ```python
  os.system('git pull')
  ```

  则python会在其路径下执行git pull命令，但是注意这里的路径并不一定是文件所在的路径，其视开发环境和调用文件的不同而定。

  os.chdir() 则可以修改python文件执行的终端路径，例如执行：

  ```python
  os.chdir('/usr/local/projects/django/testCenter/testMonitor/oerv_obsdata')
  ```

  则python的路径相关的代码，例如os.system() 和open() 都会直接受到影响，再次使用相对路径时可以发现是在oerv_obsdata目录下了。

  借助这两个命令，我们可以在后端环境中，比如Django开发环境中，轻松完成git指令操作和文件读写。

- 两个序列的元素匹配问题

  观察git diff的输出文件，我们不难发现，对于改变了的连续的几行，其总是先展示全部的改变之前的行，再展示全部的改变之后的行，且这两片‘-’ 和 ‘+’也是连续的。但是我们被彻底删除或者全新加入的行则行迹不定。

  如果我首先根据每一行前面的‘-’ 和 ‘+’，按顺序过滤出删除的行和增加的行，那么我似乎可以将这个问题抽象成一个数据结构与算法问题：

  对于两个有序序列X，Y，同一序列内元素都不同，设下标a小于b，则两个序列的元素存在如下匹配关系：

  - 一个元素至多匹配一个元素。
  - 若X[a]与Y[b]匹配，则X[b]只可能和Y[b]之后的元素匹配，反之亦然。

  请找出所有各匹配项对应的下标。

  如果把这个题解了，那么匹配项则是对应差异项，X，Y中剩下的就是全新加入的和彻底删除的。

  我穷尽我那惨兮兮的只有几十道题容量的力扣记忆库，最终也只能想出个改良爆搜法：

  1. 设一个序列为主序列X，另一个序列为被搜序列Y，对Y设立一个辅助变量mark = 0，用来代表匹配关系第二条里的b。
  2. 依次取出X中的元素。则对于X中的某一元素，从mark位置开始检索Y一直到最后，如果到了Y[i]匹配成功，则说明Y序列从mark到i的位置都没有匹配项，进行对应操作后，更新mark；如果到了末尾都匹配不到，则说明该元素没有匹配项，进行对应操作。

## 程序设计思路

```python
def diff():
  	os.chdir('/usr/local/projects/django/testCenter/testMonitor/oerv_obsdata') # change dir
    os.system('git pull') # update repository
    os.system('git log --pretty=format:"%cd,%H,%s" --date=format:"%Y%m%d" --after="{}" > ./commitlog.txt'.format(date_after))	# generate commit log file
    {
      ... # generate date-commitid file based on log file, make it one commit id a date
    }

    id1, id2 = ... # get two ids according to two dates
    # generate diff output file based on two commit id
    os.system('git diff {} {} ./obsData/projectStatistics.txt > {}'.format(id1, id2, './diff.txt'))
    # generate two sequences that require match operation
    added = [...]
    deleted = [...]
    {
      ... # solve match problem
    }
    return res
```

## 完整代码示例

以下为Django工程部分实例代码，用户需要改变部分路径和变量

```python
import os
import pandas as pd
from typing import List, Dict
import string

def getDiffs():
    # change path for both os and file operation
    os.chdir('/usr/local/projects/django/testCenter/testMonitor/oerv_obsdata')

    def update(date_after: str = '2022-06-08') -> None:
        # get lists of commit dates and commit ids
        os.system('git pull')
        os.system(
            'git log --pretty=format:"%cd,%H,%s" --date=format:"%Y%m%d" --after="{}" > ./commitlog.txt'.format(date_after))
        dates: List[str] = []
        commitID: List[str] = []
        with open('./commitlog.txt') as f:
            for line in f.readlines():
                temp = line[:-1].split(',')
                if temp[0] not in dates:
                    dates.append(temp[0])
                    commitID.append(temp[1])
        df = pd.DataFrame({'date': dates, 'id': commitID})
        df.to_csv('./date_id.csv', index=False)

    def getDiffs(date1: str = '20220726', date2: str = '20220807'):

        df = pd.read_csv('./date_id.csv', index_col='date')
        # change dtype of index
        df.index = df.index.map(str)
        id1, id2 = df.loc[date1, 'id'], df.loc[date2, 'id']
        os.system('git diff {} {} ./obsData/projectStatistics.txt > {}'.format(id1, id2, './diff.txt'))
        deleted: List = []
        added: List = []
        with open('./diff.txt') as f:
            lines = f.readlines()
            # start from 6th line
            for i in range(6, len(lines)):
                if lines[i][0] == '-':
                    deleted.append(lines[i][1:-1].split('  ')) # strip off + / - and new line character
                elif lines[i][0] == '+':
                    added.append(lines[i][1:-1].split('  '))
        added = formatData(added)
        deleted = formatData(deleted) 

        diffs: Dict[str, List] = {'add': [], 'delete': [], 'former': [], 'latter': []}
        # given two sequence, find the matching items.
        res = []
        mark = 0
        for i in range(len(deleted)):
            indicator = True
            for j in range(mark, len(added)):
                if deleted[i]['Project Name'] == added[j]['Project Name'] and deleted[i]['Repo Name'] == added[j]['Repo Name']:
                    diffs['former'].append(deleted[i])
                    diffs['latter'].append(added[j])
                    for k in range(mark, j): diffs['add'].append(added[k])
                    mark = j + 1
                    indicator = False
                    break
            if indicator: diffs['delete'].append(deleted[i])
        
        # item['condition'] means type when data type is 'add' for 'delete', otherwise means index in their own type sequence
        latterCount = 0
        for key, values in diffs.items():
            for value in values:
                if key == 'former': value['addition'] = -1
                elif key == 'latter':
                    value['addition'] = latterCount
                    latterCount += 1
                else: value['addition'] = key 
                res.append(value)
        return res 

    update()
    return JsonResponse({'data': getDiffs()})
```

