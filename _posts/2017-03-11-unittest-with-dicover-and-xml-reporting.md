---
layout: default
title: Creando tests con unittest, discover y xml reporting
---

Creando tests con unittest, discover y xml reporting
===

{{ page.date | date: "%-d %B %Y"}}

Supongamos que tenemos una calculadora muy elemental que solamente suma y multiplica:

<code>calculator.py</code>

{% highlight python %}
class Calculator:
    ADDITION_IDENTITY = 0
    MULTIPLICATION_IDENTITY = 1

    @staticmethod
    def addition(a, b):
        return a + b

    @staticmethod
    def multiplication(a, b):
        return a * b
{% endhighlight %}

y los siguientes tests:

<code>test_addition.py</code>

{% highlight python %}
import random
import unittest

from calculator import Calculator


class TestAddition(unittest.TestCase):
    def test_identity(self):
        random_int = random.randint(1, 100)
        self.assertEqual(
            Calculator.addition(random_int, Calculator.ADDITION_IDENTITY), 
            random_int)
        self.assertEqual(
            Calculator.addition(Calculator.ADDITION_IDENTITY, random_int), 
            random_int)


if __name__ == '__main__':
    unittest.main()
{% endhighlight %}

<code>test_multiplication.py</code>

{% highlight python %}
import random
import unittest

from calculator import Calculator


class TestMultiplication(unittest.TestCase):
    def test_identity(self):
        random_int = random.randint(1, 100)
        self.assertEqual(
            Calculator.multiplication(random_int, Calculator.MULTIPLICATION_IDENTITY),
            random_int)
        self.assertEqual(
            Calculator.multiplication(Calculator.MULTIPLICATION_IDENTITY, random_int),
            random_int)


if __name__ == '__main__':
    unittest.main()
{% endhighlight %}

Ahora bien, si quisiéramos ejecutar los tests de a uno, haríamos:

{% highlight bash %}
> python test_addition.py TestAddition.test_identity
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
{% endhighlight %}

Si quisiéramos ejecutar todos los tests del fichero, haríamos:

{% highlight bash %}
> python test_addition.py
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
{% endhighlight %}


Y si quisiéramos ejecutar todos los tests, de todos los archivos, haríamos:

{% highlight bash %}
> python -m unittest discover
..
----------------------------------------------------------------------
Ran 2 tests in 0.000s

OK
{% endhighlight %}

Hasta aquí perfecto. El problema surge si queremos utilizar la librería [unittest-xml-reporting](https://github.com/xmlrunner/unittest-xml-reporting){:target="_blank"} junto con el comando <code>discover</code>, debido a que no se le puede especificar un test runner diferente al que tiene por defecto, que es precisamente lo que necesitamos hacer.

Por lo tanto, lo que debemos hacer es reimplementar el método que ejecuta <code>discover</code>, como nos indica la documentación de [docs.python.org](https://docs.python.org/2/library/unittest.html#load-tests-protocol){:target="_blank"}.

<code>runner.py</code>

{% highlight python %}
import os
from unittest import TestSuite, TestLoader

import xmlrunner


def load_tests(loader, standard_tests, pattern):
    this_dir = os.path.dirname(os.path.abspath(__file__))
    package_tests = loader.discover(start_dir=this_dir, pattern=pattern)
    standard_tests.addTests(package_tests)
    xmlrunner.XMLTestRunner(output='test-reports').run(standard_tests)


load_tests(loader=TestLoader(), standard_tests= TestSuite(), pattern="test_*.py")
{% endhighlight %}

Ahora podemos ejecutar directamente:

{% highlight bash %}
> python runner.py
Running tests...
----------------------------------------------------------------------
..
----------------------------------------------------------------------
Ran 2 tests in 0.001s

OK

Generating XML reports...
{% endhighlight %}

Se crearán ficheros xml por cada uno de los ficheros de tests, dentro de la carpeta <code>test-reports</code>.

{% highlight python %}
test-reports
    |-- TEST-test_addition.TestAddition-20170311013304.xml
    |-- TEST-test_multiplication.TestMultiplication-20170311013304.xml
{% endhighlight %}

Si quisiéramos que el reporte de los tests se guardase en un único fichero, el código cambia ligeramente:

{% highlight python %}
import os
from unittest import TestSuite, TestLoader

import xmlrunner


def load_tests(loader, standard_tests, pattern):
    this_dir = os.path.dirname(os.path.abspath(__file__))
    package_tests = loader.discover(start_dir=this_dir, pattern=pattern)
    standard_tests.addTests(package_tests)
    with open('all-tests-report.xml', 'wb') as output:
        xmlrunner.XMLTestRunner(output=output).run(standard_tests)


load_tests(loader=TestLoader(), standard_tests=TestSuite(), pattern="test_*.py")
{% endhighlight %}


