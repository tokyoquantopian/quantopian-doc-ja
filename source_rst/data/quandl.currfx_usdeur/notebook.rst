Quandl: US vs. EUR Exchange Rate
================================

In this notebook, we’ll take a look at data set , available on
`Quantopian <https://www.quantopian.com/data>`__. This dataset spans
from 1999 through the current day. It contains the daily exchange rates
for the US Dollar (USD) vs. the European EURO (EUR) We access this data
via the API provided by `Quandl <https://www.quandl.com>`__. `More
details <https://www.quandl.com/data/CURRFX/USDEUR-Currency-Exchange-Rates-USD-vs-EUR>`__
on this dataset can be found on Quandl’s website.

Blaze
~~~~~

Before we dig into the data, we want to tell you about how you generally
access Quantopian partner data sets. These datasets are available using
the `Blaze <http://blaze.pydata.org>`__ library. Blaze provides the
Quantopian user with a convenient interface to access very large
datasets.

Some of these sets (though not this one) are many millions of records.
Bringing that data directly into Quantopian Research directly just is
not viable. So Blaze allows us to provide a simple querying interface
and shift the burden over to the server side.

To learn more about using Blaze and generally accessing Quantopian
partner data, clone `this tutorial
notebook <https://www.quantopian.com/clone_notebook?id=561827d21777f45c97000054>`__.

With preamble in place, let’s get started:

.. code:: ipython2

    # import the dataset
    from quantopian.interactive.data.quandl import currfx_usdeur
    # Since this data is public domain and provided by Quandl for free, there is no _free version of this
    # data set, as found in the premium sets. This import gets you the entirety of this data set.
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd
    import matplotlib.pyplot as plt

.. code:: ipython2

    currfx_usdeur.sort('asof_date')




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>asof_date</th>
          <th>rate</th>
          <th>high__est_</th>
          <th>low__est_</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1999-09-06</td>
          <td>0.941019</td>
          <td>0.95269</td>
          <td>0.92949</td>
          <td>1999-09-06</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1999-09-07</td>
          <td>0.945500</td>
          <td>0.95636</td>
          <td>0.93476</td>
          <td>1999-09-07</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1999-09-08</td>
          <td>0.944376</td>
          <td>0.95588</td>
          <td>0.93301</td>
          <td>1999-09-08</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1999-09-09</td>
          <td>0.943697</td>
          <td>0.95412</td>
          <td>0.93339</td>
          <td>1999-09-09</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1999-09-10</td>
          <td>0.948008</td>
          <td>0.00000</td>
          <td>0.00000</td>
          <td>1999-09-10</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1999-09-13</td>
          <td>0.959838</td>
          <td>0.97143</td>
          <td>0.94838</td>
          <td>1999-09-13</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1999-09-14</td>
          <td>0.964873</td>
          <td>0.97623</td>
          <td>0.95365</td>
          <td>1999-09-14</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1999-09-15</td>
          <td>0.965378</td>
          <td>0.97708</td>
          <td>0.95382</td>
          <td>1999-09-15</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1999-09-16</td>
          <td>0.963142</td>
          <td>0.97393</td>
          <td>0.95248</td>
          <td>1999-09-16</td>
        </tr>
        <tr>
          <th>9</th>
          <td>1999-09-17</td>
          <td>0.961971</td>
          <td>0.00000</td>
          <td>0.00000</td>
          <td>1999-09-17</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1999-09-20</td>
          <td>0.960917</td>
          <td>0.00000</td>
          <td>0.00000</td>
          <td>1999-09-20</td>
        </tr>
      </tbody>
    </table>



The data goes all the way back to 1999 and is updated daily.

Blaze provides us with the first 10 rows of the data for display. Just
to confirm, let’s just count the number of rows in the Blaze expression:

.. code:: ipython2

    currfx_usdeur.count()




.. raw:: html

    4240



Let’s go plot it for fun. This data set is definitely small enough to
just put right into a Pandas DataFrame

.. code:: ipython2

    usdeur_df = odo(currfx_usdeur, pd.DataFrame)
    
    usdeur_df.plot(x='asof_date', y='rate')
    plt.xlabel("As Of Date (asof_date)")
    plt.ylabel("Exchange Rate")
    plt.title("USD vs. EUR Exchange Rate")
    plt.legend().set_visible(False)



.. image:: notebook_files/notebook_6_0.png


