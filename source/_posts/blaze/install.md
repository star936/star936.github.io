---
title: Blaze(二):安装
date: 2018-12-04 20:56:28
tags: [Python, Blaze]
categories: Blaze
toc: true
---


* **conda方式**

  ```bash
  conda install blaze
  # 更多最新的构建
  conda install -c blaze blaze
  ```

* **pip方式**

  ```bash
  pip install blaze --upgrade
  or
  pip install git+https://github.com/blaze/blaze  --upgrade
  ```

* **源码方式**

  ```bash
  git clone git@github.com:blaze/blaze.git
  cd blaze
  python setup.py install
  ```

**必要依赖：**

- [numpy](http://www.numpy.org/) >= 1.7
- [datashape](https://github.com/blaze/datashape) >= 0.4.4
- [odo](https://github.com/blaze/odo) >= 0.3.1
- [toolz](http://toolz.readthedocs.org/) >= 0.7.0
- [cytoolz](https://github.com/pytoolz/cytoolz/)
- [multipledispatch](http://multiple-dispatch.readthedocs.org/) >= 0.4.7
- [pandas](http://pandas.pydata.org/)

**可选依赖：**

- [sqlalchemy](http://www.sqlalchemy.org/)
- [h5py](http://docs.h5py.org/en/latest/)
- [spark](http://spark.apache.org/) >= 1.1.0
- [pymongo](http://api.mongodb.org/python/current/)
- [pytables](http://www.pytables.org/moin)
- [bcolz](https://github.com/Blosc/bcolz)
- [flask](http://flask.pocoo.org/) >= 0.10.1
- [pytest](http://pytest.org/latest/) (for running tests)