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

## 构建文档

### Docker

```shell
# 进入文档目录
$ cd guzzle-docs-cn
# 构建Docker镜像
# 因为有不少依赖，所以不能使用Sphinx官方的镜像
$ docker build -t guzzle:sphinx .
# 运行新生成的Docker镜像，并生成HTML文档
$ docker run --rm -v $(pwd):/docs guzzle:sphinx make html
```

### 本机环境

```shell
# 进入文档目录
$ cd guzzle-docs-cn
# 安装相关依赖
$ python3 -m pip install --no-cache-dir -r requirements.txt
# 生成HTML文档
$ make html
```

生成的文档在 **`guzzle-docs-cn/__build/html`** 目录

## 资源

- [官方Github](https://github.com/guzzle/guzzle/tree/master/docs)
- [翻译参考](https://guzzle-cn.readthedocs.io/zh_CN/latest/index.html)
