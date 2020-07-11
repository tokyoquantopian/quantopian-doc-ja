EventVestor: Impairments and Charges
====================================

In this notebook, we’ll take a look at EventVestor’s *Impairments and
Charges* dataset, available on the `Quantopian
Store <https://www.quantopian.com/store>`__. This dataset spans January
01, 2007 through the current day, and documents goodwill impairments and
other one time charges reported by companies.

Blaze
~~~~~

Before we dig into the data, we want to tell you about how you generally
access Quantopian Store data sets. These datasets are available through
an API service known as `Blaze <http://blaze.pydata.org>`__. Blaze
provides the Quantopian user with a convenient interface to access very
large datasets.

Blaze provides an important function for accessing these datasets. Some
of these sets are many millions of records. Bringing that data directly
into Quantopian Research directly just is not viable. So Blaze allows us
to provide a simple querying interface and shift the burden over to the
server side.

It is common to use Blaze to reduce your dataset in size, convert it
over to Pandas and then to use Pandas for further computation,
manipulation and visualization.

Helpful links: \* `Query building for
Blaze <http://blaze.pydata.org/en/latest/queries.html>`__ \*
`Pandas-to-Blaze
dictionary <http://blaze.pydata.org/en/latest/rosetta-pandas.html>`__ \*
`SQL-to-Blaze
dictionary <http://blaze.pydata.org/en/latest/rosetta-sql.html>`__.

| Once you’ve limited the size of your Blaze object, you can convert it
  to a Pandas DataFrames using: > ``from odo import odo``
| > ``odo(expr, pandas.DataFrame)``

Free samples and limits
~~~~~~~~~~~~~~~~~~~~~~~

One other key caveat: we limit the number of results returned from any
given expression to 10,000 to protect against runaway memory usage. To
be clear, you have access to all the data server side. We are limiting
the size of the responses back from Blaze.

There is a *free* version of this dataset as well as a paid one. The
free one includes about three years of historical data, though not up to
the current day.

With preamble in place, let’s get started:

.. code:: ipython2

    # import the dataset
    from quantopian.interactive.data.eventvestor import impairments_and_charges
    # or if you want to import the free dataset, use:
    # from quantopian.interactive.data.eventvestor import impairments_and_charges_free
    
    # import data operations
    from odo import odo
    # import other libraries we will use
    import pandas as pd

.. code:: ipython2

    # Let's use blaze to understand the data a bit using Blaze dshape()
    impairments_and_charges.dshape




.. parsed-literal::

    dshape("""var * {
      event_id: ?float64,
      asof_date: datetime,
      trade_date: ?datetime,
      symbol: ?string,
      event_type: ?string,
      event_headline: ?string,
      charge_amount: ?float64,
      amount_units: ?string,
      event_rating: ?float64,
      timestamp: datetime,
      sid: ?int64
      }""")



.. code:: ipython2

    # And how many rows are there?
    # N.B. we're using a Blaze function to do this, not len()
    impairments_and_charges.count()




.. raw:: html

    3991



.. code:: ipython2

    # Let's see what the data looks like. We'll grab the first three rows.
    impairments_and_charges[:3]




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>asof_date</th>
          <th>trade_date</th>
          <th>symbol</th>
          <th>event_type</th>
          <th>event_headline</th>
          <th>charge_amount</th>
          <th>amount_units</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>131321</td>
          <td>2007-01-05</td>
          <td>2007-01-08</td>
          <td>GT</td>
          <td>Impairments/Charges</td>
          <td>Goodyear To Record $155M To $160M Charges in 1...</td>
          <td>160</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-06</td>
          <td>3384</td>
        </tr>
        <tr>
          <th>1</th>
          <td>110962</td>
          <td>2007-01-08</td>
          <td>2007-01-09</td>
          <td>MO</td>
          <td>Impairments/Charges</td>
          <td>Altria Group Subsidiary To Record $245M Asset ...</td>
          <td>245</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-09</td>
          <td>4954</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1182869</td>
          <td>2007-01-16</td>
          <td>2007-01-16</td>
          <td>FRX</td>
          <td>Impairments/Charges</td>
          <td>Forest Labs to Record $494M Charge in 4Q 07</td>
          <td>494</td>
          <td>$M</td>
          <td>1</td>
          <td>2007-01-17</td>
          <td>3014</td>
        </tr>
      </tbody>
    </table>



Let’s go over the columns: - **event_id**: the unique identifier for
this event. - **asof_date**: EventVestor’s timestamp of event capture. -
**trade_date**: for event announcements made before trading ends,
trade_date is the same as event_date. For announcements issued after
market close, trade_date is next market open day. - **symbol**: stock
ticker symbol of the affected company. - **event_type**: this should
always be *Impairments/Charges*. - **event_headline**: a brief
description of the event - **charge_amount**: amount charged in
``amount_units`` - **amount_units**: units of the amount charged. Most
commonly millions of dollars. - **event_rating**: this is always 1. The
meaning of this is uncertain. - **timestamp**: this is our timestamp on
when we registered the data. - **sid**: the equity’s unique identifier.
Use this instead of the symbol.

We’ve done much of the data processing for you. Fields like
``timestamp`` and ``sid`` are standardized across all our Store
Datasets, so the datasets are easy to combine. We have standardized the
``sid`` across all our equity databases.

We can select columns and rows with ease. Below, we’ll fetch all 2012
charges greater than $200M.

.. code:: ipython2

    twohundreds = impairments_and_charges[('2011-12-31' < impairments_and_charges['asof_date']) & 
                                            (impairments_and_charges['asof_date'] <'2013-01-01') & 
                                            (impairments_and_charges.charge_amount > 200)&
                                            (impairments_and_charges.amount_units == "$M")]
    # When displaying a Blaze Data Object, the printout is automatically truncated to ten rows.
    twohundreds.sort('asof_date')




.. raw:: html

    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>event_id</th>
          <th>asof_date</th>
          <th>trade_date</th>
          <th>symbol</th>
          <th>event_type</th>
          <th>event_headline</th>
          <th>charge_amount</th>
          <th>amount_units</th>
          <th>event_rating</th>
          <th>timestamp</th>
          <th>sid</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>1382496</td>
          <td>2012-01-11</td>
          <td>2012-01-12</td>
          <td>XL</td>
          <td>Impairments/Charges</td>
          <td>XL Group to Record Upto $220M Charges</td>
          <td>220</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-12</td>
          <td>8340</td>
        </tr>
        <tr>
          <th>1</th>
          <td>1382455</td>
          <td>2012-01-11</td>
          <td>2012-01-12</td>
          <td>RF</td>
          <td>Impairments/Charges</td>
          <td>Regions Financial to Record Upto $745M Impairm...</td>
          <td>745</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-12</td>
          <td>34913</td>
        </tr>
        <tr>
          <th>2</th>
          <td>1383159</td>
          <td>2012-01-13</td>
          <td>2012-01-16</td>
          <td>ADM</td>
          <td>Impairments/Charges</td>
          <td>Archer Daniels to Record Upto $360M Charge in ...</td>
          <td>360</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-14</td>
          <td>128</td>
        </tr>
        <tr>
          <th>3</th>
          <td>1383004</td>
          <td>2012-01-13</td>
          <td>2012-01-13</td>
          <td>NVS</td>
          <td>Impairments/Charges</td>
          <td>Novartis to Record $1.22B Charges</td>
          <td>1220</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-14</td>
          <td>21536</td>
        </tr>
        <tr>
          <th>4</th>
          <td>1383880</td>
          <td>2012-01-18</td>
          <td>2012-01-18</td>
          <td>HES</td>
          <td>Impairments/Charges</td>
          <td>Hess Corp to Record $525M Charge in 4Q 11 on R...</td>
          <td>525</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-19</td>
          <td>216</td>
        </tr>
        <tr>
          <th>5</th>
          <td>1384387</td>
          <td>2012-01-19</td>
          <td>2012-01-19</td>
          <td>ECL</td>
          <td>Impairments/Charges</td>
          <td>Ecolab to Record $480M Charges by FY 13</td>
          <td>480</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-20</td>
          <td>2427</td>
        </tr>
        <tr>
          <th>6</th>
          <td>1385496</td>
          <td>2012-01-23</td>
          <td>2012-01-24</td>
          <td>MUR</td>
          <td>Impairments/Charges</td>
          <td>Murphy Oil Unit to Record $370M Asset Impairme...</td>
          <td>370</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-24</td>
          <td>5126</td>
        </tr>
        <tr>
          <th>7</th>
          <td>1386032</td>
          <td>2012-01-24</td>
          <td>2012-01-25</td>
          <td>BBOX</td>
          <td>Impairments/Charges</td>
          <td>Black Box to Record $320M Charges in 3Q 12</td>
          <td>320</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-25</td>
          <td>11732</td>
        </tr>
        <tr>
          <th>8</th>
          <td>1385962</td>
          <td>2012-01-24</td>
          <td>2012-01-25</td>
          <td>RE</td>
          <td>Impairments/Charges</td>
          <td>Everest Re Group to Record $245M Catastrophe L...</td>
          <td>245</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-25</td>
          <td>13720</td>
        </tr>
        <tr>
          <th>9</th>
          <td>1388133</td>
          <td>2012-01-30</td>
          <td>2012-01-30</td>
          <td>X</td>
          <td>Impairments/Charges</td>
          <td>United States Steel to Record Upto $450M Charg...</td>
          <td>450</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-01-31</td>
          <td>8329</td>
        </tr>
        <tr>
          <th>10</th>
          <td>1388719</td>
          <td>2012-01-31</td>
          <td>2012-01-31</td>
          <td>X</td>
          <td>Impairments/Charges</td>
          <td>United States Steel to Record Up to $450M Char...</td>
          <td>450</td>
          <td>$M</td>
          <td>1</td>
          <td>2012-02-01</td>
          <td>8329</td>
        </tr>
      </tbody>
    </table>



Now suppose we want a DataFrame of the Blaze Data Object above, and we
only want the sid, charge_amount and the asof_date.

.. code:: ipython2

    df = odo(twohundreds, pd.DataFrame)
    df = df[['sid', 'asof_date','charge_amount']].dropna()
    # When printing a pandas DataFrame, the head 30 and tail 30 rows are displayed. The middle is truncated.
    df




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>sid</th>
          <th>asof_date</th>
          <th>charge_amount</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>34913</td>
          <td>2012-01-11</td>
          <td>745.0</td>
        </tr>
        <tr>
          <th>1</th>
          <td>8340</td>
          <td>2012-01-11</td>
          <td>220.0</td>
        </tr>
        <tr>
          <th>2</th>
          <td>128</td>
          <td>2012-01-13</td>
          <td>360.0</td>
        </tr>
        <tr>
          <th>3</th>
          <td>21536</td>
          <td>2012-01-13</td>
          <td>1220.0</td>
        </tr>
        <tr>
          <th>4</th>
          <td>216</td>
          <td>2012-01-18</td>
          <td>525.0</td>
        </tr>
        <tr>
          <th>5</th>
          <td>2427</td>
          <td>2012-01-19</td>
          <td>480.0</td>
        </tr>
        <tr>
          <th>6</th>
          <td>5126</td>
          <td>2012-01-23</td>
          <td>370.0</td>
        </tr>
        <tr>
          <th>7</th>
          <td>11732</td>
          <td>2012-01-24</td>
          <td>320.0</td>
        </tr>
        <tr>
          <th>8</th>
          <td>13720</td>
          <td>2012-01-24</td>
          <td>245.0</td>
        </tr>
        <tr>
          <th>9</th>
          <td>8329</td>
          <td>2012-01-30</td>
          <td>450.0</td>
        </tr>
        <tr>
          <th>10</th>
          <td>8329</td>
          <td>2012-01-31</td>
          <td>450.0</td>
        </tr>
        <tr>
          <th>11</th>
          <td>351</td>
          <td>2012-03-05</td>
          <td>703.0</td>
        </tr>
        <tr>
          <th>12</th>
          <td>7334</td>
          <td>2012-03-13</td>
          <td>293.0</td>
        </tr>
        <tr>
          <th>13</th>
          <td>1335</td>
          <td>2012-03-24</td>
          <td>700.0</td>
        </tr>
        <tr>
          <th>14</th>
          <td>2263</td>
          <td>2012-04-02</td>
          <td>350.0</td>
        </tr>
        <tr>
          <th>15</th>
          <td>6116</td>
          <td>2012-04-05</td>
          <td>372.0</td>
        </tr>
        <tr>
          <th>16</th>
          <td>23112</td>
          <td>2012-04-10</td>
          <td>400.0</td>
        </tr>
        <tr>
          <th>17</th>
          <td>32902</td>
          <td>2012-04-17</td>
          <td>370.0</td>
        </tr>
        <tr>
          <th>18</th>
          <td>24838</td>
          <td>2012-04-19</td>
          <td>260.0</td>
        </tr>
        <tr>
          <th>19</th>
          <td>2351</td>
          <td>2012-04-30</td>
          <td>420.0</td>
        </tr>
        <tr>
          <th>20</th>
          <td>24838</td>
          <td>2012-05-17</td>
          <td>280.0</td>
        </tr>
        <tr>
          <th>21</th>
          <td>754</td>
          <td>2012-05-22</td>
          <td>350.0</td>
        </tr>
        <tr>
          <th>22</th>
          <td>3735</td>
          <td>2012-05-23</td>
          <td>1700.0</td>
        </tr>
        <tr>
          <th>23</th>
          <td>14388</td>
          <td>2012-06-05</td>
          <td>425.0</td>
        </tr>
        <tr>
          <th>24</th>
          <td>4151</td>
          <td>2012-06-08</td>
          <td>600.0</td>
        </tr>
        <tr>
          <th>25</th>
          <td>11673</td>
          <td>2012-06-14</td>
          <td>1000.0</td>
        </tr>
        <tr>
          <th>26</th>
          <td>88</td>
          <td>2012-06-21</td>
          <td>439.0</td>
        </tr>
        <tr>
          <th>27</th>
          <td>26204</td>
          <td>2012-06-25</td>
          <td>272.0</td>
        </tr>
        <tr>
          <th>29</th>
          <td>5061</td>
          <td>2012-07-02</td>
          <td>6200.0</td>
        </tr>
        <tr>
          <th>30</th>
          <td>903</td>
          <td>2012-07-06</td>
          <td>210.0</td>
        </tr>
        <tr>
          <th>...</th>
          <td>...</td>
          <td>...</td>
          <td>...</td>
        </tr>
        <tr>
          <th>61</th>
          <td>5520</td>
          <td>2012-10-26</td>
          <td>275.0</td>
        </tr>
        <tr>
          <th>62</th>
          <td>166</td>
          <td>2012-11-01</td>
          <td>2000.0</td>
        </tr>
        <tr>
          <th>63</th>
          <td>42173</td>
          <td>2012-11-01</td>
          <td>250.0</td>
        </tr>
        <tr>
          <th>65</th>
          <td>26169</td>
          <td>2012-11-15</td>
          <td>400.0</td>
        </tr>
        <tr>
          <th>66</th>
          <td>7671</td>
          <td>2012-11-15</td>
          <td>325.0</td>
        </tr>
        <tr>
          <th>67</th>
          <td>24833</td>
          <td>2012-11-17</td>
          <td>400.0</td>
        </tr>
        <tr>
          <th>68</th>
          <td>161</td>
          <td>2012-11-20</td>
          <td>290.0</td>
        </tr>
        <tr>
          <th>69</th>
          <td>23998</td>
          <td>2012-11-26</td>
          <td>400.0</td>
        </tr>
        <tr>
          <th>71</th>
          <td>24838</td>
          <td>2012-11-28</td>
          <td>1075.0</td>
        </tr>
        <tr>
          <th>72</th>
          <td>5092</td>
          <td>2012-11-30</td>
          <td>267.5</td>
        </tr>
        <tr>
          <th>73</th>
          <td>5862</td>
          <td>2012-12-04</td>
          <td>300.0</td>
        </tr>
        <tr>
          <th>74</th>
          <td>1335</td>
          <td>2012-12-05</td>
          <td>1000.0</td>
        </tr>
        <tr>
          <th>75</th>
          <td>7041</td>
          <td>2012-12-05</td>
          <td>650.0</td>
        </tr>
        <tr>
          <th>76</th>
          <td>239</td>
          <td>2012-12-07</td>
          <td>2000.0</td>
        </tr>
        <tr>
          <th>77</th>
          <td>8580</td>
          <td>2012-12-10</td>
          <td>380.0</td>
        </tr>
        <tr>
          <th>78</th>
          <td>1274</td>
          <td>2012-12-11</td>
          <td>880.0</td>
        </tr>
        <tr>
          <th>79</th>
          <td>14064</td>
          <td>2012-12-11</td>
          <td>370.0</td>
        </tr>
        <tr>
          <th>80</th>
          <td>8340</td>
          <td>2012-12-12</td>
          <td>350.0</td>
        </tr>
        <tr>
          <th>81</th>
          <td>4488</td>
          <td>2012-12-13</td>
          <td>750.0</td>
        </tr>
        <tr>
          <th>82</th>
          <td>25305</td>
          <td>2012-12-17</td>
          <td>300.0</td>
        </tr>
        <tr>
          <th>83</th>
          <td>25955</td>
          <td>2012-12-18</td>
          <td>220.0</td>
        </tr>
        <tr>
          <th>84</th>
          <td>8369</td>
          <td>2012-12-18</td>
          <td>288.0</td>
        </tr>
        <tr>
          <th>85</th>
          <td>21462</td>
          <td>2012-12-19</td>
          <td>240.0</td>
        </tr>
        <tr>
          <th>86</th>
          <td>40430</td>
          <td>2012-12-19</td>
          <td>400.0</td>
        </tr>
        <tr>
          <th>87</th>
          <td>10025</td>
          <td>2012-12-19</td>
          <td>240.0</td>
        </tr>
        <tr>
          <th>88</th>
          <td>24783</td>
          <td>2012-12-20</td>
          <td>2000.0</td>
        </tr>
        <tr>
          <th>89</th>
          <td>13720</td>
          <td>2012-12-20</td>
          <td>220.0</td>
        </tr>
        <tr>
          <th>90</th>
          <td>17395</td>
          <td>2012-12-21</td>
          <td>4300.0</td>
        </tr>
        <tr>
          <th>91</th>
          <td>34334</td>
          <td>2012-12-21</td>
          <td>333.1</td>
        </tr>
        <tr>
          <th>92</th>
          <td>7543</td>
          <td>2012-12-26</td>
          <td>1100.0</td>
        </tr>
      </tbody>
    </table>
    <p>89 rows × 3 columns</p>
    </div>


