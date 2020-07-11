Lesson 2: Futures in Research
-----------------------------

Futures Data
~~~~~~~~~~~~

Quantopian has open, high, low, close, and volume (OHLCV) data for 72 US
futures from the beginning of 2002 to the current date. This dataset
contains both day and minute frequency data for 24 hours x 5 days a
week, and is collected from electronic trade data.

The list of US futures currently available on Quantopian can be found at
the end of this notebook.

Future Object
~~~~~~~~~~~~~

On Quantopian, a ``Future`` is an instance of a specific futures
contract denoted by a base symbol + a code for month/year of delivery.
For example, ``CLF16`` is a contract for crude oil with delivery date in
January (``F``) 2016 (``16``). Here is a reference for delivery months
and their corresponding codes:

==== ==============
Code Delivery Month
==== ==============
F    January
G    February
H    March
J    April
K    May
M    June
N    July
Q    August
U    September
V    October
X    November
Z    December
==== ==============

Let’s start by looking at a particular contract of the Light Sweet Crude
Oil future (``CL``). In Research, a reference to a futures contract is
obtained via the symbols function. Run the following code in a new cell
to output the ``Future`` object corresponding to ``CLF16``.

.. code:: ipython2

    clf16 = symbols('CLF16')
    clf16




.. parsed-literal::

    Future(1058201601 [CLF16])



Here is a brief description of some of the properies of the ``Future``
object:

-  **``root_symbol``**: The root symbol of the underlying asset. For
   example, ``CL`` corresponds to crude oil.
-  **``start_date``**: The date the contract becomes available on
   Quantopian. Note that the price of a contract might be ``NaN`` near
   the ``start_date``, as it may not be actively traded until it gets
   closer to its delivery date.
-  **``end_date``**: The last date the contract can be traded or closed
   before delivery.
-  **``notice_date``**: The date in which the exchange can start
   assigning delivery to accounts holding long positions on the
   contract.
-  **``auto_close_date``**: This is two days prior to either
   ``notice_date`` or ``end_date``, whichever is earlier. In
   backtesting, positions in contracts will be automatically closed out
   on their ``auto_close_date``.
-  **``tick_size``**: The price of a future can only change in
   increments of its ``tick_size``. For example, ``CL`` changes in
   increments of $0.01.
-  **``multiplier``**: The number of units per contract. A contract for
   ``CL`` corresponds to 1000 barrels of oil.

In the following lesson, we’ll take a look at how to get pricing and
volume data for a particular futures contract.

List of Available Futures
~~~~~~~~~~~~~~~~~~~~~~~~~

These are the 72 US futures that are currently available on Quantopian.

====== ==============================================
Symbol Future
====== ==============================================
BD     Big Dow
BO     Soybean Oil
CM     Corn E-Mini
CN     Corn
DJ     DJIA Futures
ET     Ethanol
FF     30-Day Federal Funds
FI     5-Year Deliverable Interest Rate Swap Futures
FS     5-Year Interest Rate Swap Futures
FV     5-Year T-Note
MB     Municipal Bonds
MS     Soybeans E-Mini
MW     Wheat E-Mini
OA     Oats
RR     Rough Rice
SM     Soybean Meal
SY     Soybeans
TN     10-Year Deliverable Interest Rate Swap Futures
TS     10-Year Interest Rate Swap Futures
TU     2-Year T-Note
TY     10-Year T-Note
UB     Ultra Tbond
US     30-Year T-Bond
WC     Wheat
YM     Dow Jones E-mini
VX     VIX Futures
AD     Australian Dollar
AI     Bloomberg Commodity Index Futures
BP     British Pound
CD     Canadian Dollar
EC     Euro FX
ED     Eurodollar
EE     Euro FX E-mini
ES     S&P 500 E-Mini
EU     E-micro EUR/USD Futures
FC     Feeder Cattle
JE     Japanese Yen E-mini
JY     Japanese Yen
LB     Lumber
LC     Live Cattle
LH     Lean Hogs
MD     S&P 400 MidCap Futures
ME     Mexican Peso
MI     S&P 400 MidCap E-Mini
ND     NASDAQ 100 Futures
NK     Nikkei 225 Futures
NQ     NASDAQ 100 E-Mini
NZ     New Zealand Dollar
SF     Swiss Franc
SP     S&P 500 Futures
TB     TBills
GC     Gold
HG     Copper High Grade
SV     Silver
CL     Light Sweet Crude Oil
HO     NY Harbor ULSD Futures
HU     Unleaded Gasoline
NG     Natural Gas
PA     Palladium
PL     Platinum
PB     Pork Bellies
QG     Natural Gas E-mini
QM     Crude Oil E-Mini
XB     RBOB Gasoline Futures
EI     MSCI Emerging Markets Mini
EL     Eurodollar NYSE LIFFE
MG     MSCI EAFE Mini
XG     Gold mini-sized
YS     Silver mini-sized
RM     Russell 1000 Mini
SB     Sugar #11
ER     Russell 2000 Mini
====== ==============================================
