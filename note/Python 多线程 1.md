# Python 多线程

由于全局解释器锁（GIL）的存在，Python的多线程性能一直饱受诟病。在传统的实践中，我们常常会用`multiprocessing`模块来执行可并行任务，但这种做法本质上是一种多进程解决方案，比较适合CPU密集型任务。对于I/O密集型任务，我们仍然希望使用多线程方案来处理。

自`python 3.13`以后，官方开始逐步推出无GIL的Python解释器。从`python 3.14`开始更是将这一功能的构建列为受支持选项，而非实验性功能，这意味着我们可以利用Python写出真正的多线程代码，并处理相关的任务了。

目前，无GIL的Python解释器仍然是可选状态，在运行多线程任务时，需要手动开启该特性，并且在创建项目时有一些事项需要注意，在本文的后面会提到。

## 如何安装无GIL的Python解释器？

### GUI 环境

在GUI环境中，可以直接使用安装包安装Python，只需要在安装时勾选开启`Free-threaded Python`特性即可。这里以Mac OS为例：

到这一步，点击“自定义”

<p align="center">
  <img src="https://github.com/0x-0cd/0x-0cd.github.io/blob/draft/large_file/images/note_4_1.png" alt="pic1" style="max-width: 40%; height: auto;">
</p>

勾选`Free-threaded Python`

<p align="center">
  <img src="https://github.com/0x-0cd/0x-0cd.github.io/blob/draft/large_file/images/note_4_2.png" alt="pic2" style="max-width: 40%; height: auto;">
</p>

### 无 GUI 环境

在服务器等无GUI环境中，你需要下载对应版本的Python源码（tag中通常带有`freethreaded`），并在编译时使用`--disable-gil`参数进行编译安装。

解压源码后，执行如下操作：

```bash
./configure --enable-optimizations --disable-gil
make -j $(nproc)
sudo make altinstall
```

## 如何使用无GIL的Python解释器？

即使安装了无GIL的Python解释器，这个特性也不是默认开启的。需要用`python3t`替代`python3`命令来创建项目。

```bash
python3t -m venv .venv
```

创建并启动虚拟环境后，再使用`python3`就会默认启用无GIL的Python解释器了。

```bash
source .venv/bin/activate
python3 main.py
```

有时候，当你执行程序，可能会在终端看到这样的输出：

```text
<frozen importlib._bootstrap>:491: RuntimeWarning: The global interpreter lock (GIL) has been enabled to load module 'ckzg', which has not declared that it can run safely without the GIL. To override this behavior and keep the GIL disabled (at your own risk), run with PYTHON_GIL=0 or -Xgil=0.
```

这是因为你的项目中导入了某个模块，无法确保其能在无GIL的环境下安全运行，解释器被强制退回开启GIL的版本。如果你希望无视风险关闭GIL，可以这样启动程序：

```bash
python3 -X gil=0 main.py
```

## 使用`uv`

使用`uv`进行python项目管理时，可以将默认解释器版本固定为某个版本，以启用无GIL的解释器：

```bash
uv python list # 查看并选择一个带有 freethreaded 字样的解释器版本
uv python pin cpython-3.14.2+freethreaded-macos-aarch64-none
```

## 验证环境

如果你不确定当前虚拟环境下的Python解释器是否关闭GIL，可以用下面的代码进行验证：

```python
'''
验证当前环境下的 python 解释器是否关闭了 GIL
'''
import sys


def main():
    if sys._is_gil_enabled():
        print("⚠️ GIL 已开启，多线程性能可能受到限制")
    else:
        print("✅ GIL 已关闭")


if __name__ == "__main__":
    main()

```

## 对比多线程性能差距

我们可以写一段程序来验证开启和关闭GIL时，Python的性能差距。

```python
import threading
import multiprocessing


def worker():
    while True:
        1 + 1


def main():
    cpu_thread = multiprocessing.cpu_count()
    for i in range(cpu_thread):
        th = threading.Thread(target=worker)
        th.start()


if __name__ == '__main__':
    main()

```

上面这段程序在`cpython-3.13.5-macos-aarch64-none`版本的解释器下运行时，CPU利用率如下：

<p align="center">
  <img src="https://github.com/0x-0cd/0x-0cd.github.io/blob/draft/large_file/images/note_4_3.png" alt="pic1" style="max-width: 50%; height: auto;">
</p>

切换到`cpython-3.14.2+freethreaded-macos-aarch64-none`版本，CPU利用率如下：

<p align="center">
  <img src="https://github.com/0x-0cd/0x-0cd.github.io/blob/draft/large_file/images/note_4_4.png" alt="pic1" style="max-width: 50%; height: auto;">
</p>

可以看到提升还是很明显的，可以充分利用CPU的性能了。

## ⚠️ 需要注意的问题

> 这条注意事项的日期为2026年1月13日，该问题可能在相关程序的后续的更新中被修复

如果你使用`VS Code`或基于它开发的编辑器编辑代码，请注意不要使用`3.15.0`这个最新的Python解释器。这个版本似乎修改了`typing`模块中的一些接口，导致`autopep8`代码格式化插件无法正常工作。把解释器版本退回`3.14.2`即可。

p.s. 如果你解决了这个问题，欢迎通过邮件告知我 :)
