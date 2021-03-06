
A Generic CSV Data Feed
-----------------------

.. post:: Aug 4, 2015
   :author: mementum
   :image: 1
   :redirect: posts/2015-08-04

An issue has led to the implementation of **GenericCSVData** which can be used
to parse different CSV formats.

The issue in GitHub, `Issue #6
<https://github.com/mementum/backtrader/issues/6>`_ clearly shows there is a
need to have something that can actually handle any incoming CSV data feed.

The sauce is in the params declaration::

    class GenericCSVData(feed.CSVDataBase):
        params = (
            ('nullvalue', float('NaN')),
	    ('dtformat', '%Y-%m-%d %H:%M:%S'),
	    ('tmformat', '%H:%M:%S'),

	    ('datetime', 0),
	    ('time', -1),
	    ('open', 1),
	    ('high', 2),
	    ('low', 3),
	    ('close', 4),
	    ('volume', 5),
	    ('openinterest', 6),
	)

Because the class inherits from CSVDataBase, some standard paramenters are
available:

  - ``fromdate`` (takes a datetime object to limit the starting date)
  - ``todate`` (takes a datetime object) to limit the ending date)

  - ``headers`` (default: True, indicating if the CSV data has a headers row)
  - ``separator`` (default: ",", the character separating the fields)

  - ``dataname`` (name of the file which has the CSV data or a file-like object)

Some other parameters like ``name``, ``compression`` and ``timeframe`` are just
informative unless you plan to execute resampling.

Of course more importantly, the meaning of the newly define parameters:

  - ``datetime`` (default: 0) column containing the date (or datetime) field

  - ``time`` (default: -1) column containing the time field if separate from the
    datetime field (-1 indicates it's not present)

  - ``open`` (default: 1) , ``high`` (default: 2), ``low`` (default: 3),
    ``close`` (default: 4), ``volume`` (default: 5), ``openinterest``
    (default: 6)

    Index of the columns containing the corresponding fields

    If a negative value is passed (example: -1) it indicates the field is not
    present in the CSV data

  - ``nullvalue`` (default: float('NaN'))

    Value that will be used if a value which should be there is missing (the CSV
    field is empty)

  - ``dtformat`` (default: %Y-%m-%d %H:%M:%S)

    Format used to parse the datetime CSV field

  - ``tmformat`` (default: %H:%M:%S)

    Format used to parse the time CSV field if "present" (the default for the
    "time" CSV field is not to be present)

This should probably suffice to cover many different CSV formats and the absence
of values.

An example usage covering the following requirements:

  - Limit input to year 2000
  - HLOC order rather than OHLC
  - Missing values to be replaced with zero (0.0)
  - Daily bars are provided and datetime is just the day with format YYYY-MM-DD
  - No ``openinterest`` column is present

The code::

  import datetime
  import backtrader as bt
  import backtrader.feeds as btfeed

  ...
  ...

  data = btfeed.GenericCSVData(
      dataname='mydata.csv',

      fromdate=datetime.datetime(2000, 1, 1),
      todate=datetime.datetime(2000, 12, 31),

      nullvalue=0.0,

      dtformat=('%Y-%m-%d'),

      datetime=0,
      high=1,
      low=2,
      open=3,
      close=4,
      volume=5,
      openinterest=-1
  )

Slightly modified requirements:

  - Limit input to year 2000
  - HLOC order rather than OHLC
  - Missing values to be replaced with zero (0.0)
  - Intraday bars are provided, with separate date and time columns
    - Date has format YYYY-MM-DD
    - Time has format HH.MM.SS
  - No ``openinterest`` column is present

The code::

  import datetime
  import backtrader as bt
  import backtrader.feeds as btfeed

  ...
  ...

  data = btfeed.GenericCSVData(
      dataname='mydata.csv',

      fromdate=datetime.datetime(2000, 1, 1),
      todate=datetime.datetime(2000, 12, 31),

      nullvalue=0.0,

      dtformat=('%Y-%m-%d'),
      tmformat=('%H.%M.%S'),

      datetime=0,
      time=1,
      high=2,
      low=3,
      open=4,
      close=5,
      volume=6,
      openinterest=-1
  )
