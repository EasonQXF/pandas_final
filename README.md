# pandas_final

# 数据分析描述markdown 12%

## 数据源 3%
> 数据源：raw_data 基本栏位的概述
- 数据的来源是国家数据库中的分省数据，例如各地区的城镇人员平均工资等数据。（[请点击此处前往数据下载链接处](https://github.com/EasonQXF/pandas_final)）
- 数据的格式为tsv数据格式。
- 用df_raw.shape查看数据规格得到输出为：(908300, 1)。可以看出数据规格足够庞大，能够得到有效的数据分析结果。

#### 数据分析目标 3%
> 数据分析目标：在你的数据分析项目中，主要分析的目标数据介绍，相关数据整理、清洗、重塑的数据流程介绍。（建议用数据流的方式进行描述）
- **主要分析的目标数据**：指标是：“年”、“行业”、“城市”，主要分析上海市、北京市、广州市三个热门城市的交通运输、仓储和邮政业、住宿和餐饮业、计算机服务和软件业等不同的行业在近十年的分布变化、发展趋势。
- **相关数据整理、清洗、重塑的数据流程**：

1. 首先是国家分省数据库的准备
```python
df_raw = pd.read_csv("fsnd_zb_data.tsv", encoding='utf8', sep='\t',
                    keep_default_na=False, na_values = 'na_rep',
                    index_col = [0,1,2] )
df_m = pd.read_csv("fsnd_zb_meta.tsv", encoding='utf8', sep='\t',)
df_r = pd.read_csv("reg_treeId_level2.tsv", encoding='utf8', sep='\t')
display(df_raw)
# 可以用df_raw.shape查看数据规格
```

2. 创建指标字典
```python
指标字典 = df_m.set_index("code")['cname'].to_dict()
指标字典
```

3. 可分为指标字典以及以指标作为索引，变为指标的字典
```python
地区字典 = df_r.set_index("id")['name'].to_dict()
地区字典
df = df_raw.reset_index().set_index("zb").rename(index=指标字典)
df# zb作为索引，变为指标的字典
df = df.reset_index().set_index("reg").rename(index=地区字典)
df
df_zh = df.reset_index().rename(columns={"zb":"指标","reg":"地区","sj":"年","data":"数据"})
df_zh
```

4. 按行业分城镇单位就业人员，全国各地区的历年变化之统计值
```python
df_zh.set_index('指标').loc["城镇单位就业人员工资总额","年":"数据"]
dslice = df_zh [ df_zh.指标.str.contains("城镇单位就业人员")]
dslice
指标分的可能性 = dslice.指标.unique()
指标分的可能性
指标分的可能性 = [ x.split("城镇单位就业") for x in dslice.指标.unique()]
指标分的可能性
```

5. 取‘人员’，‘人员平均工资’来分析，并且进行进一步的切分至最终剩下需要的11780条数据
```python
指标分的可能性_取 = [ [x,y] for (x,y) in 指标分的可能性 if (y=='人员平均工资' or y=='人员') and x!='']
指标分的可能性_取
指标分的可能性_取_all = ["城镇单位就业".join(x) for x in 指标分的可能性_取]
指标分的可能性_取_all
df_就业切片 = df_zh.set_index("指标").loc[指标分的可能性_取_all].reset_index()
df_就业切片
# 剩下11780条数据
```

6. 接下来要做分进合击
```python
df_就业切片.groupby(['指标']).agg({"数据":["min","mean","max"]})
df_就业切片['行业'] = [x.split("城镇单位就业")[0] for x in df_就业切片.指标]
df_就业切片['行业指标'] = [x.split("城镇单位就业")[1] for x in df_就业切片.指标]
df_就业切片
```

```python
print (list(df_就业切片.行业指标.unique()))
报表 = dict()
报表['人员平均工资_原'] = df_就业切片.query("行业指标=='人员平均工资'")\
                                   .drop(["指标","行业指标"], axis=1)
报表['人员_原'] = df_就业切片.query("行业指标=='人员'")\
                               .drop(["指标","行业指标"], axis=1)
报表['人员平均工资'] = 报表['人员平均工资_原'].set_index(['地区','年','行业']).unstack(1)

报表['人员'] = 报表['人员_原'].set_index(['地区','年','行业']).unstack(1)

报表['人员平均工资_原']
```

```python
with pd.ExcelWriter("报表_原.xlsx") as writer:
    for sheet_name in sorted(报表.keys()):
        报表[sheet_name].to_excel(writer,sheet_name=sheet_name)
        print (sheet_name, 报表[sheet_name].shape)
```

```python
原 = 报表['人员平均工资_原']
报 = 报表['人员平均工资']
报表['人员'] = 报表['人员_原'].set_index(['地区','年','行业']).unstack(0).unstack(1)
报表['人员']
```

```python
print ("原.index", 原.index, "\n", "原.columns", 原.columns)
print ("报索/行:", 报.index, "\n", "报索/栏/变:", 报.columns)
display(原)
display(报)
原.set_index(["地区","年","行业"])\
  .unstack(0)
```

**至此，完成国家分省数据库数据中对于各地区的不同行业的近十年来的发展变化的整理、清洗、重塑。**

#### 数据分析结果价值宣言 3%
> 数据分析结果价值宣言：数据分析的最终目标带来的分析价值、加值，作为数据结果展示和数据故事描述的基础
- 现在的人们很少会终生在一个地方或者同一个公司或者部门工作，更多的人在自己的经济情况稳定，不安于现状，想要去更大的城市、更大的天地去拼搏，从而提升那个自己的生活质量、提高自己的眼界、可以到世界的各处去看看面对新的挑战。这样的生活方式对于某些人来说就是人生的乐趣之一。

那么针对这样的人群，去哪里的城市和往哪一个行业去发展，这就很关键了。本项目是通过国家分省数据库作为数据源，将数据整理、清晰，以可视化表格的方式呈现出热门城市不同行业在近十年的变化发展趋势的数据分析结果，从而让想要去热门城市就业的人们，了解各行业的基本发展趋势，在选择就业行业、城市的时候能够有所依据，且能够做出更正确的选择。

#### 数据分析结果可视化 3%
> 数据分析结果可视化：用到的可视化模块介绍，可视化模块的具体API使用介绍，预计可视化展示的结果以及数据故事的阐述


# 数据分析电子档/ipynb文档（请参考廖、许电子讲义） 12%
## 项目介绍 3%
> 项目介绍：项目人、时间、数据源、目标
- 项目人：丘小峰
- 时间：2020/7/10
- 数据源：国家分省数据库数据
- 目标：通过国家分省数据库作为数据源，将数据整理、清晰，以可视化表格的方式呈现出热门城市不同行业在近十年的变化发展趋势，从而让想要去热门城市就业的人们，了解各行业的基本发展趋势，在选择就业行业、城市的时候能够有所依据，且做出更正确的选择。

#### 加分：清晰、整洁，css样式设计合理
- 代码没有多余的符号，增加部分代码处的解释说明注释文字。
- 利用jupyter notebook的代码美化工具进一步优化代码排版。
![美化工具](https://images.gitee.com/uploads/images/2020/0714/000636_cb213a57_2229377.png)
- 参考平时廖老师和许老师的课件讲义添加CSS样式优化阅读效果。

#### markdown细节规范 3%
> markdown细节规范：标题清晰，关键步骤有具体的描述，包含数据分析模块的方法和处理的主要内容等
- 每一步都有醒目的标题写清楚，形成目录方便查看。
- 步骤在关键步骤的时候有进一步的说明文字、链接等。

#### 代码干净整洁 3%
> 代码干净整洁：可视化图清晰 3%

# 主要项目作品方案一（应至少包含基本的数据分析结果：如分进合击出报表的结果可视化，数据重塑后的时间、空间对比等） 15%
- 数据分析结果的基本的对比图 5%
![基本](https://images.gitee.com/uploads/images/2020/0714/000637_79e20ac4_2229377.png)
![基本](https://images.gitee.com/uploads/images/2020/0714/000635_bdadf66f_2229377.png)
![基本](https://images.gitee.com/uploads/images/2020/0714/000635_890be5b9_2229377.png)

- 数据分析结果的时空交互 5%
![可视化](https://gitee.com/EdisonQXF/pandas_final/raw/master/images/%E5%8F%AF%E8%A7%86%E5%8C%961.png)

- 数据分析结果的地图 5%
![可视化地图](https://images.gitee.com/uploads/images/2020/0714/000638_2851a292_2229377.png)

- 加分项：数据分析结果的时空交互的地图
