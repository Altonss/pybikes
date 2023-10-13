pybikes [![Build Status](https://github.com/eskerda/pybikes/actions/workflows/test.yml/badge.svg)](https://github.com/eskerda/pybikes/actions/workflows/test.yml)
=======
![pybikes](http://citybik.es/files/pybikes.png)

pybikes provides a set of tools to scrape bike sharing data from different
websites and APIs, thus providing a coherent and generalized set of classes
and methods to access this sort of information.

The library is distributed and intended mainly for statistics and data
sharing projects. More importantly, it powers the [CityBikes][1] project, and
is composed of a set of classes and a pack of data files that provide instances
for all different systems.

Installation
------------

Install directly from GitHub:
```bash
pip install git+https://github.com/eskerda/pybikes.git
```

Or after downloading/cloning the source:
```bash
python setup.py install
```

The following dependencies are required (example using Ubuntu package manager):
```
sudo apt-get install python
sudo apt-get install python-setuptools
sudo apt-get install libxml2 libxml2-dev libxslt1-dev libgeos-dev
```

Usage
-----
```python
>>> import pybikes

# Capital BikeShare instantiation data is in bixi.json file
>>> capital_bikeshare = pybikes.get('capital-bikeshare')

# The instance contains all possible metadata regarding this system
>>> print(capital_bikeshare.meta)
{
    'name': 'Capital BikeShare',
    'city': 'Washington, DC - Arlington, VA',
    'longitude': -77.0363658,
    'system': 'Bixi',
    'company': ['PBSC'],
    'country': 'USA',
    'latitude': 38.8951118
}
# The update method retrieves the list of stations
>>> print(len(capital_bikeshare.stations))
0
>>> capital_bikeshare.update()
>>> print(len(capital_bikeshare.stations))
191
>>> print(capital_bikeshare.stations[0])
--- 31000 - 20th & Bell St ---
bikes: 7
free: 4
latlng: 38.8561,-77.0512
extra: {
    'installed': True,
    'uid': 1,
    'locked': False,
    'removalDate': '',
    'installDate': '1316059200000',
    'terminalName': '31000',
    'temporary': False,
    'name': '20th & Bell St',
    'latestUpdateTime': '1353454305589'
}
```

Some systems might require an API key to work (for instance, Cyclocity). In
these cases, the instance factory can take an extra API key parameter.

```python
>>> key = "This is not an API key"
>>> dublinbikes = pybikes.get('dublinbikes', key)
```

Note that pybikes works as an instance factory and, choicely, instances can be
generated by passing the right arguments to the desired class

```python
>>> from pybikes.cyclocity import BixiSystem
>>> capital_bikeshare = BixiSystem(
        tag = 'foo_tag',
        root_url = 'http://capitalbikeshare.com/data/stations/',
        meta = {'foo':'bar'}
    )
```

The way information is retrieved can be tweaked using the PyBikesScraper class
included on the utils module thus allowing session reusing and niceties such as
using a proxy. This class uses [Requests][2] module internally.

```python
>>> scraper = pybikes.utils.PyBikesScraper()
>>> scraper.enableProxy()
>>> scraper.setProxies({
        "http" : "127.0.0.1:8118",
        "https": "127.0.0.1:8118"
    })
>>> scraper.setUserAgent("Walrus™ v3.0")
>>> scraper.headers['Foo'] = 'bar'
>>> capital_bikeshare.update(scraper)
```

[1]: http://www.citybik.es              "CityBikes"
[2]: http://docs.python-requests.org    "Requests"

Tests
-----
Tests are separated between unit tests and integration tests with the different
sources supported.

To run unit tests simply

```bash
make test
```

To run integration tests

```bash
make test-update
```

Note that some systems require authorization keys, tests expect these to be
set as environment variables like:

```bash
PYBIKES_CYCLOCITY='some-api-key'
PYBIKES_DEUTSCHEBAHN_CLIENT_ID='some-client-id'
PYBIKES_DEUTSCHEBAHN_CLIENT_SECRET='some-client-secret'

# or if using an .env file
# source .env

make test-update
```

This project uses pytest for tests. Test a particular network by passing a
filter expresson

```bash
pytest -k bicing
pytest -k gbfs
```

To speed up tests execution, install [pytest-xdist][3] to specify the number of
CPUs to use

```bash
pytest -k gbfs -n auto
```

To use Makefile steps and pass along pytest arguments, append to the `T_FLAGS`
variable

```bash
make test-update T_FLAGS+='-n 10 -k gbfs'
```

Integration tests can generate a json report file with all extracted data stored
as geojson. Using this json report file, further useful reports can be generated
like a summary of the overall health of the library or a map visualization of
all the information.

For more information on reports see [utils/README.md][4]

[3]: https://pypi.org/project/pytest-xdist/
[4]: utils/README.md
