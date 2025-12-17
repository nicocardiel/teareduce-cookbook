# The package teareduce

The Python package **teareduce** has been initially developed by Nicolás
Cardiel to facilitate the reduction of astronomical images within the course
*Experimental Techniques in Astrophysics*, which is part of the [Master's
Degree in Astrophysics](https://www.ucm.es/masterastrofisica) at the
[Universidad Complutense of Madrid (UCM)](https://www.ucm.es/).

Different people at UCM have contributed to the development and testing of
this package: Sergio Pascual, María Chillarón, Cristina Cabello, Jesús Gallego
and Jaime Zamorano. Acknowledgment is also given to Maite Ceballos (IFCA) for
her help in setting up this documentation website.

Thanks are also extended to many students of the Master's in Astrophysics at
UCM who, in recent years, have used this code in their practical work
associated with the reduction of astronomical observations obtained with
different instruments and telescopes.

## Purpose

This package is not intended to be a general-purpose image reduction code. It
only includes specific operations required in certain steps of the traditional
astronomical image reduction process that, at the time of its creation, were
not available in more established packages such as
[ccdproc](https://ccdproc.readthedocs.io/en/latest/). Students can examine the
Python code and introduce modifications in order to gain a deeper understanding
of the operations performed during the astronomical image reduction process.
In addition, it also offers alternative ways to perform certain tasks that we
have found to be more practical for use in Master's level classes.

## Installation


It is advisable to install this package in a Python environment. For example:

```console
$ python3 -m venv venv_tea
$ . tea/bin/activate
(venv_tea) $ pip install teareduce
```

````{warning}
If you are planning to use **tea-cleanest**, you need to install this
package with extra dependencies. In this case employ:

```console
(venv_tea) $ pip install 'teareduce[cleanest]'
```
````

````{note}
On PyPI there is a package called **tea** that provides utilities unrelated to
**teareduce**. However, throughout this documentation we will often use
```python
import teareduce as tea
```
as a convenient alias when working with our package.
````

The repository of the code is available in
[GitHub](https://github.com/nicocardiel/teareduce).

## Table of contents

The different sections of this document consist of Jupyter notebooks that
demonstrate the practical use of the defined functionality. Each notebook can
be easily downloaded using the download icon that appears at the top of each
web page.


```{tableofcontents}
```

This document has been generated with the help of the [Jupyter
Book](https://jupyterbook.org/en/stable/intro.html) package.

