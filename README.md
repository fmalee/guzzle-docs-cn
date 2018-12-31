# Guzzle中文文档

在[nullest/guzzle](https://guzzle-cn.readthedocs.io/zh_CN/latest/quickstart.html)的翻译基础上增加了翻译文档。

那为什么还在造轮子：

- 该项目在GitHub上已经不存在。
- 再次建立一个翻译项目也是为了防止该中文文档的托管也消失。
- 大多数都没有翻译，所以如果有条件会进行翻译。

限于有限的水平：

- 英语一般，仅能尽可能将原语句翻译明确，但中文下的语句可能不够通顺。
- 非程序猿、设计狮，翻译的术语可能有些变味，已经尽量避免，并整理出术语对译。
- Guzzle水平有限，有些不确定的句子没有翻译。
- 时间有限，仅翻译当下本人会用上的章节，其余回头再译（一般情况下是回不了头的）。
- 每次修改、调整本文档时，都会顺手同步官方的文档。

## 同步记录

- `master`
  - `c5da4c3128db9f892f0911043cfffb6c559bd6da`（2018-12-30）

## 未译章节

- `testing.rst`

## 术语约定

- `Application`：应用
- `Header`：标头
- `Body`：正文、主体
- `Handler`：处理器
- `Middleware`：中间件
- `Transfer`：传输
- `Stream`：流
- `Metadata`：元数据
- `Callable`：回调
- `Decorator`：装饰器

## 生成主题

官方有自己的[专用主题](https://github.com/guzzle/guzzle_sphinx_theme)，但是需要手动安装：

```bash
$ pip install guzzle_sphinx_theme
```

### 如何生成

1. 按照[pip安装](https://pip.pypa.io/en/stable/installing/)文档中的说明安装 [pip](https://pip.pypa.io/en/stable/)；

2. 安装 [Sphinx](http://sphinx-doc.org/) 和 [PHP和Symfony的Sphinx扩展](https://github.com/fabpot/sphinx-php) （根据你的系统，你可能需要以root用户身份执行此命令）：

```bash
$ pip install sphinx~=1.3.0 git+https://github.com/fabpot/sphinx-php.git
```

3. 运行以下命令以HTML格式构建文档：

```bash
# 跳转到项目目录
$ cd ~/guzzle-cn
$ make html
```

生成的文档可在 `_build/html` 目录中找到。

## 资源

- [官方Github](https://github.com/guzzle/guzzle/tree/master/docs)
- [翻译参考](https://guzzle-cn.readthedocs.io/zh_CN/latest/index.html)
