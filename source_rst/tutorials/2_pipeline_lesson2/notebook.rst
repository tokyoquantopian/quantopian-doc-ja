##Creating a Pipeline In this lesson, we will take a look at creating an
empty pipeline. First, let’s import the Pipeline class:

.. code:: ipython2

    from quantopian.pipeline import Pipeline

In a new cell, let’s define a function to create our pipeline. Wrapping
our pipeline creation in a function sets up a structure for more complex
pipelines that we will see later on. For now, this function simply
returns an empty pipeline:

.. code:: ipython2

    def make_pipeline():
        return Pipeline()

In a new cell, let’s instantiate our pipeline by running
``make_pipeline()``:

.. code:: ipython2

    my_pipe = make_pipeline()

###Running a Pipeline

Now that we have a reference to an empty Pipeline, ``my_pipe`` let’s run
it to see what it looks like. Before running our pipeline, we first need
to import ``run_pipeline``, a research-only function that allows us to
run a pipeline over a specified time period.

.. code:: ipython2

    from quantopian.research import run_pipeline

Let’s run our pipeline for one day (2015-05-05) with ``run_pipeline``
and display it. Note that the 2nd and 3rd arguments are the start and
end dates of the simulation, respectively.

.. code:: ipython2

    result = run_pipeline(my_pipe, '2015-05-05', '2015-05-05')

A call to ``run_pipeline`` returns a `pandas
DataFrame <http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html>`__
indexed by date and securities. Let’s see what the empty pipeline looks
like:

.. code:: ipython2

    result




.. raw:: html

    <div style="max-height:1000px;max-width:1500px;overflow:auto;">
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th rowspan="61" valign="top">2015-05-05 00:00:00+00:00</th>
          <th>Equity(2 [AA])</th>
        </tr>
        <tr>
          <th>Equity(21 [AAME])</th>
        </tr>
        <tr>
          <th>Equity(24 [AAPL])</th>
        </tr>
        <tr>
          <th>Equity(25 [AA_PR])</th>
        </tr>
        <tr>
          <th>Equity(31 [ABAX])</th>
        </tr>
        <tr>
          <th>Equity(39 [DDC])</th>
        </tr>
        <tr>
          <th>Equity(41 [ARCB])</th>
        </tr>
        <tr>
          <th>Equity(52 [ABM])</th>
        </tr>
        <tr>
          <th>Equity(53 [ABMD])</th>
        </tr>
        <tr>
          <th>Equity(62 [ABT])</th>
        </tr>
        <tr>
          <th>Equity(64 [ABX])</th>
        </tr>
        <tr>
          <th>Equity(66 [AB])</th>
        </tr>
        <tr>
          <th>Equity(67 [ADSK])</th>
        </tr>
        <tr>
          <th>Equity(69 [ACAT])</th>
        </tr>
        <tr>
          <th>Equity(70 [VBF])</th>
        </tr>
        <tr>
          <th>Equity(76 [TAP])</th>
        </tr>
        <tr>
          <th>Equity(84 [ACET])</th>
        </tr>
        <tr>
          <th>Equity(86 [ACG])</th>
        </tr>
        <tr>
          <th>Equity(88 [ACI])</th>
        </tr>
        <tr>
          <th>Equity(100 [IEP])</th>
        </tr>
        <tr>
          <th>Equity(106 [ACU])</th>
        </tr>
        <tr>
          <th>Equity(110 [ACXM])</th>
        </tr>
        <tr>
          <th>Equity(112 [ACY])</th>
        </tr>
        <tr>
          <th>Equity(114 [ADBE])</th>
        </tr>
        <tr>
          <th>Equity(117 [AEY])</th>
        </tr>
        <tr>
          <th>Equity(122 [ADI])</th>
        </tr>
        <tr>
          <th>Equity(128 [ADM])</th>
        </tr>
        <tr>
          <th>Equity(134 [SXCL])</th>
        </tr>
        <tr>
          <th>Equity(149 [ADX])</th>
        </tr>
        <tr>
          <th>Equity(153 [AE])</th>
        </tr>
        <tr>
          <th>...</th>
        </tr>
        <tr>
          <th>Equity(48961 [NYMT_O])</th>
        </tr>
        <tr>
          <th>Equity(48962 [CSAL])</th>
        </tr>
        <tr>
          <th>Equity(48963 [PAK])</th>
        </tr>
        <tr>
          <th>Equity(48969 [NSA])</th>
        </tr>
        <tr>
          <th>Equity(48971 [BSM])</th>
        </tr>
        <tr>
          <th>Equity(48972 [EVA])</th>
        </tr>
        <tr>
          <th>Equity(48981 [APIC])</th>
        </tr>
        <tr>
          <th>Equity(48989 [UK])</th>
        </tr>
        <tr>
          <th>Equity(48990 [ACWF])</th>
        </tr>
        <tr>
          <th>Equity(48991 [ISCF])</th>
        </tr>
        <tr>
          <th>Equity(48992 [INTF])</th>
        </tr>
        <tr>
          <th>Equity(48993 [JETS])</th>
        </tr>
        <tr>
          <th>Equity(48994 [ACTX])</th>
        </tr>
        <tr>
          <th>Equity(48995 [LRGF])</th>
        </tr>
        <tr>
          <th>Equity(48996 [SMLF])</th>
        </tr>
        <tr>
          <th>Equity(48997 [VKTX])</th>
        </tr>
        <tr>
          <th>Equity(48998 [OPGN])</th>
        </tr>
        <tr>
          <th>Equity(48999 [AAPC])</th>
        </tr>
        <tr>
          <th>Equity(49000 [BPMC])</th>
        </tr>
        <tr>
          <th>Equity(49001 [CLCD])</th>
        </tr>
        <tr>
          <th>Equity(49004 [TNP_PRD])</th>
        </tr>
        <tr>
          <th>Equity(49005 [ARWA_U])</th>
        </tr>
        <tr>
          <th>Equity(49006 [BVXV])</th>
        </tr>
        <tr>
          <th>Equity(49007 [BVXV_W])</th>
        </tr>
        <tr>
          <th>Equity(49008 [OPGN_W])</th>
        </tr>
        <tr>
          <th>Equity(49009 [PRKU])</th>
        </tr>
        <tr>
          <th>Equity(49010 [TBRA])</th>
        </tr>
        <tr>
          <th>Equity(49131 [OESX])</th>
        </tr>
        <tr>
          <th>Equity(49259 [ITUS])</th>
        </tr>
        <tr>
          <th>Equity(49523 [TLGT])</th>
        </tr>
      </tbody>
    </table>
    <p>8236 rows × 0 columns</p>
    </div>



The output of an empty pipeline is a DataFrame with no columns. In this
example, our pipeline has an index made up of all 8000+ securities
(truncated in the display) for May 5th, 2015, but doesn’t have any
columns.

In the following lessons, we’ll take a look at how to add columns to our
pipeline output, and how to filter down to a subset of securities.
