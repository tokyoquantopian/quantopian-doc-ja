Quandl: ADP National Employment Report
======================================

In this notebook, we’ll take a look at data set , available on
`Quantopian <https://www.quantopian.com/data>`__. This dataset spans
from 2001 through the current day. It contains the value for employment
levels as provided by ADP, the payroll service provider. We access this
data via the API provided by `Quandl <https://www.quandl.com>`__. `More
details <https://www.quandl.com/data/ADP/EMPL_SEC-Employment-by-Sector>`__
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
    from quantopian.interactive.data.quandl import adp_empl_sec 
    # Since this data is public domain and provided by Quandl for free, there is no _free version of this
    # data set, as found in the premium sets. This import gets you the entirety of this data set.
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd
    import matplotlib.pyplot as plt

.. code:: ipython2

    adp_empl_sec.sort('asof_date')




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>asof_date</th>
          <th>total_private</th>
          <th>goods_producing</th>
          <th>service_providing</th>
          <th>timestamp</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>2001-04-30</td>
          <td>111573.000000</td>
          <td>24275.294168</td>
          <td>87297.757625</td>
          <td>2001-04-30</td>
        </tr>
        <tr>
          <th>1</th>
          <td>2001-05-31</td>
          <td>111398.132821</td>
          <td>24140.307995</td>
          <td>87257.824826</td>
          <td>2001-05-31</td>
        </tr>
        <tr>
          <th>2</th>
          <td>2001-06-30</td>
          <td>111167.552706</td>
          <td>23992.631803</td>
          <td>87174.920903</td>
          <td>2001-06-30</td>
        </tr>
        <tr>
          <th>3</th>
          <td>2001-07-31</td>
          <td>110964.904589</td>
          <td>23854.365510</td>
          <td>87110.539079</td>
          <td>2001-07-31</td>
        </tr>
        <tr>
          <th>4</th>
          <td>2001-08-31</td>
          <td>110719.120440</td>
          <td>23699.879744</td>
          <td>87019.240696</td>
          <td>2001-08-31</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2001-09-30</td>
          <td>110457.629117</td>
          <td>23562.811692</td>
          <td>86894.817424</td>
          <td>2001-09-30</td>
        </tr>
        <tr>
          <th>6</th>
          <td>2001-10-31</td>
          <td>110078.236801</td>
          <td>23390.394579</td>
          <td>86687.842222</td>
          <td>2001-10-31</td>
        </tr>
        <tr>
          <th>7</th>
          <td>2001-11-30</td>
          <td>109716.868489</td>
          <td>23206.664911</td>
          <td>86510.203579</td>
          <td>2001-11-30</td>
        </tr>
        <tr>
          <th>8</th>
          <td>2001-12-31</td>
          <td>109494.189038</td>
          <td>23070.799900</td>
          <td>86423.389138</td>
          <td>2001-12-31</td>
        </tr>
        <tr>
          <th>9</th>
          <td>2002-01-31</td>
          <td>109424.930934</td>
          <td>22951.233078</td>
          <td>86473.697856</td>
          <td>2002-01-31</td>
        </tr>
        <tr>
          <th>10</th>
          <td>2002-02-28</td>
          <td>109334.446257</td>
          <td>22861.907749</td>
          <td>86472.538508</td>
          <td>2002-02-28</td>
        </tr>
      </tbody>
    </table>



The data goes all the way back to 2001 and is updated monthly.

Blaze provides us with the first 10 rows of the data for display. Just
to confirm, let’s just count the number of rows in the Blaze expression:

.. code:: ipython2

    adp_empl_sec.count()




.. raw:: html

    176



Let’s go plot it for fun. This data set is definitely small enough to
just put right into a Pandas DataFrame

.. code:: ipython2

    adp_df = odo(adp_empl_sec, pd.DataFrame)
    
    adp_df.plot(x='asof_date', y='total_private')
    plt.xlabel("As Of Date (asof_date)")
    plt.ylabel("Employment Levels")
    plt.title("ADP Employment Level Data")
    plt.legend().set_visible(False)



.. image:: notebook_files/notebook_6_0.png


