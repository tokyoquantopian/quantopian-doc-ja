In the previous lesson we created a data pipeline that selects assets to consider for our portfolio, and calculates alpha scores for those assets. The remaining lessons will be conducted in Quantopian's `Interactive Development Environment (IDE) <https://www.quantopian.com/algorithms>`__ where we will build a trading algorithm, attach our data pipeline to it, and use alpha scores for portfolio construction. Then, we will analyze our algorithm's performance under more realistic conditions by simulating it over historical data. This type of historical simulation is commonly known as "backtesting."

Algorithm API & Core Functions
------------------------------

The next step will be to build a basic structure for our trading algorithm using Quantopian's Algorithm API. The Algorithm API provides functions that facilitate order scheduling and execution, and allow us to initialize and manage parameters in our algorithms.
Let's take a look at some of the core functions we will use in our algorithm:


**initialize(context)**

``initialize`` is called exactly once when our algorithm starts running and requires ``context`` as input. Any parameter initialization and one-time startup logic should go here.
``context`` is an augmented Python `dictionary <https://docs.python.org/2/tutorial/datastructures.html#dictionaries>`__ used for maintaining state throughout the simulation process, and can be referenced in different parts of our algorithm. Any variables that we want to persist between function calls should be stored in ``context`` instead of using global variables. Context variables can be accessed and initialized using dot notation (``context.some_attribute``).


**before_trading_start(context, data)**

``before_trading_start`` is called once per day before the market opens and requires ``context`` and ``data`` as input. ``context`` is a reference to the same dictionary from ``initialize``, and data is an object that stores several API functions that allow us to look up current or historical pricing and volume data for any asset.
before_trading_start is also where we get our pipeline's output, and do any pre-processing of the resulting data before using it for portfolio construction. We will cover this in the next lesson.

``before_trading_start`` is also where we get our pipeline's output, and do any pre-processing of the resulting data before using it for portfolio construction. We will cover this in the next lesson.

**schedule_function(func, day_rule, time_rule)**

On Quantopian, algorithms can trade equities between 9:30AM-4PM Eastern Time on regular trading days, following the `NYSE trading calendar <https://www.nyse.com/markets/hours-calendars>`__ . ``schedule_function`` allows us to execute custom functions at specified dates and times. For example, we can schedule a function to rebalance our portfolio at market open on the first day of each week as follows:

.. code:: python3

    schedule_function(
        rebalance,
        date_rule=date_rules.week_start(),
        time_rule=time_rules.market_open()
    )


Scheduling functions should be done in ``initialize``, and custom functions scheduled with this method need to take ``context`` and ``data`` as arguments. For a full list of ``date_rules`` and ``time_rules`` available, check out the `documentation <https://www.quantopian.com/docs/api-reference/algorithm-api-reference#quantopian.algorithm.schedule_function>`__.


Next, let's build a skeleton for our trading algorithm. For now we will have our algorithm keep track of the number of days that have passed in the simulation and log different messages depending on the date and time. In the next couple of lessons we will integrate our data pipeline and add trading logic.
To run this example algorithm, create a copy by clicking the "Clone" button. Once you are in the IDE, run a backtest by clicking "Build Algorithm" (top left) or "Run Full Backtest" (top right).

.. code:: python3

    # Import Algorithm API
    import quantopian.algorithm as algo


    def initialize(context):
        # Initialize algorithm parameters
        context.day_count = 0
        context.daily_message = "Day {}."
        context.weekly_message = "Time to place some trades!"

        # Schedule rebalance function
        algo.schedule_function(
            rebalance,
            date_rule=algo.date_rules.week_start(),
            time_rule=algo.time_rules.market_open()
        )


    def before_trading_start(context, data):
        # Execute any daily actions that need to happen
        # before the start of a trading session
        context.day_count += 1
        log.info(context.daily_message, context.day_count)


    def rebalance(context, data):
        # Execute rebalance logic
        log.info(context.weekly_message)

Now that we have a basic structure for a trading algorithm, let's add the data pipeline we created in the previous lesson to our algorithm.

