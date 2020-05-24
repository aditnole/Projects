
.. code:: ipython3

    %logstop
    %logstart -rtq ~/.logs/dw.py append
    %matplotlib inline
    import matplotlib
    import seaborn as sns
    sns.set()
    matplotlib.rcParams['figure.dpi'] = 144

.. code:: ipython3

    from static_grader import grader

DW Miniproject
==============

Introduction
------------

The objective of this miniproject is to exercise your ability to wrangle
tabular data set and aggregate large data sets into meaningful summary
statistics. We’ll work with the same medical data used in the ``pw``
miniproject but leverage the power of Pandas to more efficiently
represent and act on our data.

Downloading the data
--------------------

We first need to download the data we’ll be using from Amazon S3:

.. code:: ipython3

    !mkdir dw-data
    !wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/201701scripts_sample.csv.gz -nc -P ./dw-data/
    !wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/201606scripts_sample.csv.gz -nc -P ./dw-data/
    !wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/practices.csv.gz -nc -P ./dw-data/
    !wget http://dataincubator-wqu.s3.amazonaws.com/dwdata/chem.csv.gz -nc -P ./dw-data/


.. parsed-literal::

    mkdir: cannot create directory ‘dw-data’: File exists
    File ‘./dw-data/201701scripts_sample.csv.gz’ already there; not retrieving.
    
    File ‘./dw-data/201606scripts_sample.csv.gz’ already there; not retrieving.
    
    File ‘./dw-data/practices.csv.gz’ already there; not retrieving.
    
    File ‘./dw-data/chem.csv.gz’ already there; not retrieving.
    


Loading the data
----------------

Similar to the ``PW`` miniproject, the first step is to read in the
data. The data files are stored as compressed CSV files. You can load
the data into a Pandas DataFrame by making use of the ``gzip`` package
to decompress the files and Panda’s ``read_csv`` methods to parse the
data into a DataFrame. You may want to check the Pandas documentation
for parsing
`CSV <http://pandas.pydata.org/pandas-docs/stable/generated/pandas.read_csv.html>`__
files for reference.

For a description of the data set please, refer to the `PW
miniproject <./pw.ipynb>`__. **Note that all questions make use of the
2017 data only, except for Question 5 which makes use of both the 2017
and 2016 data.**

.. code:: ipython3

    import pandas as pd
    import numpy as np
    import gzip
    import os
    os.listdir()




.. parsed-literal::

    ['.ipynb_checkpoints',
     'in.ipynb',
     'dw-data',
     'dw.ipynb',
     'pw.ipynb',
     'pw-data']



.. code:: ipython3

    import gzip
    with gzip.open('./dw-data/201701scripts_sample.csv.gz', 'rb') as f:
        file_content = f.read()
        


.. code:: ipython3

    # load the 2017 data
    
    scripts = pd.read_csv('./dw-data/201701scripts_sample.csv.gz', compression='gzip')
    scripts.head()

.. code:: ipython3

    col_names=[ 'code', 'name', 'addr_1', 'addr_2', 'borough', 'village', 'post_code']
    practices = pd.read_csv('./dw-data/practices.csv.gz', compression='gzip')
    practices.columns = col_names
    practices.head()

.. code:: ipython3

    chem = pd.read_csv('./dw-data/chem.csv.gz', compression='gzip')
    chem.head()


Now that we’ve loaded in the data, let’s first replicate our results
from the ``PW`` miniproject. Note that we are now working with a larger
data set so the answers will be different than in the ``PW`` miniproject
even if the analysis is the same.

Question 1: summary_statistics
------------------------------

In the ``PW`` miniproject we first calculated the total, mean, standard
deviation, and quartile statistics of the ``'items'``,
``'quantity'``\ ’, ``'nic'``, and ``'act_cost'`` fields. To do this we
had to write some functions to calculate the statistics and apply the
functions to our data structure. The DataFrame has a ``describe`` method
that will calculate most (not all) of these things for us.

Submit the summary statistics to the grader as a list of tuples:
[(‘act_cost’, (total, mean, std, q25, median, q75)), …]

.. code:: ipython3

    clist = ['items', 'quantity', 'nic', 'act_cost']
    scripts[clist[1]].describe()
    scripts['items']
    
    data = scripts['items']
    ctotal = sum(data)
    print(ctotal)



.. parsed-literal::

    8888304


.. code:: ipython3

    scripts[clist[1]].describe()




.. parsed-literal::

    count    973193.000000
    mean        741.329835
    std        3665.426958
    min           0.000000
    25%          28.000000
    50%         100.000000
    75%         350.000000
    max      577720.000000
    Name: quantity, dtype: float64



.. code:: ipython3

    scripts['items'].describe()
    
    newlist= []
    summary_stats = []
    for item in clist:
        data = scripts[item]
        ctotal = sum(data)
        #ctotal = scripts[item].describe()[0]
        cmean = scripts[item].describe()[1]
        cstd = scripts[item].describe()[2]
        cq25 = scripts[item].describe()[4]
        cmedian = scripts[item].describe()[5]
        cq75 = scripts[item].describe()[6]
        innertupple = tuple([ctotal, cmean, cstd, cq25, cmedian, cq75 ])
        newlist = tuple([item, innertupple ])
        summary_stats.append(newlist)
        
    summary_stats
    #for item im scripts.




.. parsed-literal::

    [('items', (8888304, 9.133135976111625, 29.204198282803603, 1.0, 2.0, 6.0)),
     ('quantity',
      (721457006, 741.3298348837282, 3665.426958467915, 28.0, 100.0, 350.0)),
     ('nic',
      (71100424.84000827, 73.05891517920908, 188.070256906825, 7.8, 22.64, 65.0)),
     ('act_cost',
      (66164096.11999956,
       67.98661326170655,
       174.40170332301963,
       7.33,
       21.22,
       60.67))]



.. code:: ipython3

    #summary_stats = [('items', (0,) * 6), ('quantity', (0,) * 6), ('nic', (0,) * 6), ('act_cost', (0,) * 6)]

.. code:: ipython3

    grader.score.dw__summary_statistics(summary_stats)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


Question 2: most_common_item
----------------------------

We can also easily compute summary statistics on groups within the data.
In the ``pw`` miniproject we had to explicitly construct the groups
based on the values of a particular field. Pandas will handle that for
us via the ``groupby`` method. This process is `detailed in the Pandas
documentation <https://pandas.pydata.org/pandas-docs/stable/groupby.html>`__.

Use ``groupby`` to calculate the total number of items dispensed for
each ``'bnf_name'``. Find the item with the highest total and return the
result as ``[(bnf_name, total)]``.

.. code:: ipython3

    by_bnfname = scripts.groupby('bnf_name')['items'].sum()
    by_bnfname.head()
    by_bnfname.nlargest(1)
    most = []
    for k,v in by_bnfname.nlargest(1).items():
        most.append(tuple([k,v]))
        
    most
    #a = tuple(by_bnfname.nlargest(1)[0], by_bnfname.nlargest(1)[1])
    #can also use: In [64]: res = g.apply(lambda x: x.order(ascending=False).head(3))
    #by_bnfname.sort('items').head()




.. parsed-literal::

    [('Omeprazole_Cap E/C 20mg', 218583)]



.. code:: ipython3

    most_common_item = most

.. code:: ipython3

    grader.score.dw__most_common_item(most_common_item)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


Question 3: items_by_region
---------------------------

Now let’s find the most common item by post code. The post code
information is in the ``practices`` DataFrame, and we’ll need to
``merge`` it into the ``scripts`` DataFrame. Pandas provides `extensive
documentation <https://pandas.pydata.org/pandas-docs/stable/merging.html>`__
with diagrammed examples on different methods and approaches for joining
data. The ``merge`` method is only one of many possible options.

Return your results as a list of tuples
``(post code, item name, amount dispensed as % of total)``. Sort your
results ascending alphabetically by post code and take only results from
the first 100 post codes.

**NOTE:** Some practices have multiple postal codes associated with
them. Use the alphabetically first postal code. Note some postal codes
may have multiple ``'bnf_name'`` with the same prescription rate for the
maximum. In this case, take the alphabetically first ``'bnf_name'`` (as
in the PW miniproject).

scripts.rename(columns = {‘practice’:‘code’}, inplace = True) merged_df
= scripts.merge(practices, on=[‘code’]) #merged_df.head() filtered =
merged_df.groupby([‘post_code’, ‘bnf_name’])[‘items’]

filter2 = pd.DataFrame({‘items’ : merged_df.groupby( [‘post_code’,
‘bnf_name’] ).size()}).reset_index() filter3 = filter2.sort_values(by
=[‘post_code’, ‘bnf_name’]) filter3

#def postalsum: postallist = filter3[‘post_code’].unique()

print(scripts.head()) filter3
print(scripts.loc[scripts[‘bnf_name’].values == ‘Bisacodyl_Tab E/C
5mg’])

.. code:: ipython3

    asrt = practices.sort_values('post_code').drop_duplicates('code', keep = 'first')
    
    amerge = scripts.merge(asrt, left_on = 'practice', right_on = 'code')
    
    sumitemsbypost = amerge.groupby(['post_code','bnf_name'])[['items']].sum()
    sumitemsbypost.reset_index(inplace=True)
    #sumitemsbypost.head()
    itemsmax = sumitemsbypost.groupby('post_code')[['items']].max()
    #itemsmax.head()
    #above code will find the value of item which is max
    itemsmax.reset_index(inplace=True)
    totalitems = itemsmax.merge(sumitemsbypost, on = ['post_code', 'items'], how= 'left')
    #the above code is matching the bnfname corresponding max item
    totalitems.head()
    





.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>post_code</th>
          <th>items</th>
          <th>bnf_name</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>B11 4BW</td>
          <td>706</td>
          <td>Salbutamol_Inha 100mcg (200 D) CFF</td>
        </tr>
        <tr>
          <th>1</th>
          <td>B12 9LP</td>
          <td>425</td>
          <td>Paracet_Tab 500mg</td>
        </tr>
        <tr>
          <th>2</th>
          <td>B18 7AL</td>
          <td>556</td>
          <td>Salbutamol_Inha 100mcg (200 D) CFF</td>
        </tr>
        <tr>
          <th>3</th>
          <td>B21 9RY</td>
          <td>1033</td>
          <td>Metformin HCl_Tab 500mg</td>
        </tr>
        <tr>
          <th>4</th>
          <td>B23 6DJ</td>
          <td>599</td>
          <td>Lansoprazole_Cap 30mg (E/C Gran)</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    # sorting and finding ratios
    # below code will keep the first item in sorted bnf max table
    sortedtotal = totalitems.sort_values('bnf_name').drop_duplicates(['post_code', 'items'], keep = 'first')
    sortedtotal.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>post_code</th>
          <th>items</th>
          <th>bnf_name</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>73</th>
          <td>E15 4ES</td>
          <td>451</td>
          <td>Amlodipine_Tab 10mg</td>
        </tr>
        <tr>
          <th>66</th>
          <td>DN16 2AB</td>
          <td>1072</td>
          <td>Amlodipine_Tab 5mg</td>
        </tr>
        <tr>
          <th>255</th>
          <td>WS9 8AJ</td>
          <td>396</td>
          <td>Amlodipine_Tab 5mg</td>
        </tr>
        <tr>
          <th>142</th>
          <td>NG2 7SD</td>
          <td>605</td>
          <td>Amlodipine_Tab 5mg</td>
        </tr>
        <tr>
          <th>97</th>
          <td>KT16 8HZ</td>
          <td>401</td>
          <td>Amlodipine_Tab 5mg</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    sortedtotal.sort_values('post_code', inplace = True)
    sortedtotal.head()
    # above code will again sort acc to postcode
    
    postaltotal = sumitemsbypost.groupby('post_code')[['items']].sum().reset_index()
    # this will find the sum of total items from a particular post
    
    q3df = postaltotal.merge(sortedtotal, on ='post_code')
    q3df.head()
    #sortedtotal.shape
    #postaltotal.shape
    q3df['ratio'] = q3df['items_y'].astype(float)/q3df['items_x'].astype(float)
    #q3df['ratio'].head()
    filterdf= q3df[['post_code', 'bnf_name', 'ratio']]
    filterdf = filterdf.head(100)
    vlist = filterdf.values.tolist()
    finallist = []
    for item in vlist:
        finallist.append(tuple(item))
        
    
    
    


.. code:: ipython3

    items_by_region = finallist

.. code:: ipython3

    grader.score.dw__items_by_region(items_by_region)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


Question 4: script_anomalies
----------------------------

Drug abuse is a source of human and monetary costs in health care. A
first step in identifying practitioners that enable drug abuse is to
look for practices where commonly abused drugs are prescribed unusually
often. Let’s try to find practices that prescribe an unusually high
amount of opioids. The opioids we’ll look for are given in the list
below.

.. code:: ipython3

    opioids = ['morphine', 'oxycodone', 'methadone', 'fentanyl', 'pethidine', 'buprenorphine', 'propoxyphene', 'codeine']

These are generic names for drugs, not brand names. Generic drug names
can be found using the ``'bnf_code'`` field in ``scripts`` along with
the ``chem`` table.. Use the list of opioids provided above along with
these fields to make a new field in the ``scripts`` data that flags
whether the row corresponds with a opioid prescription.

The Python join() method is a string method, and takes a list of things
to join with the string. A simpler example might help explain:

         “,”.join([“a”, “b”, “c”]) ‘a,b,c’

.. code:: ipython3

    opioidslist = '|'.join(opioids)
    opioidslist
    chem.head()
    nscript = scripts
    opioidbnfcode = chem.loc[chem['NAME'].str.contains(opioidslist, case=False)]['CHEM SUB'].tolist()
    
    nscript['opioid_prescription'] = nscript['bnf_code'].isin(opioidbnfcode)
    nscript.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>practice</th>
          <th>bnf_code</th>
          <th>bnf_name</th>
          <th>items</th>
          <th>nic</th>
          <th>act_cost</th>
          <th>quantity</th>
          <th>opioid_prescription</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>N85639</td>
          <td>0106020C0</td>
          <td>Bisacodyl_Tab E/C 5mg</td>
          <td>1</td>
          <td>0.39</td>
          <td>0.47</td>
          <td>12</td>
          <td>False</td>
        </tr>
        <tr>
          <th>1</th>
          <td>N85639</td>
          <td>0106040M0</td>
          <td>Movicol Plain_Paed Pdr Sach 6.9g</td>
          <td>1</td>
          <td>4.38</td>
          <td>4.07</td>
          <td>30</td>
          <td>False</td>
        </tr>
        <tr>
          <th>2</th>
          <td>N85639</td>
          <td>0301011R0</td>
          <td>Salbutamol_Inha 100mcg (200 D) CFF</td>
          <td>1</td>
          <td>1.50</td>
          <td>1.40</td>
          <td>1</td>
          <td>False</td>
        </tr>
        <tr>
          <th>3</th>
          <td>N85639</td>
          <td>0304010G0</td>
          <td>Chlorphenamine Mal_Oral Soln 2mg/5ml</td>
          <td>1</td>
          <td>2.62</td>
          <td>2.44</td>
          <td>150</td>
          <td>False</td>
        </tr>
        <tr>
          <th>4</th>
          <td>N85639</td>
          <td>0401020K0</td>
          <td>Diazepam_Tab 2mg</td>
          <td>1</td>
          <td>0.16</td>
          <td>0.26</td>
          <td>6</td>
          <td>False</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    practicecode = practices.loc[0]
    practicecode
    truevalues = nscript[nscript['opioid_prescription']==True]
    
    #somevalue = nscript[nscript['practice']== 'N85639']


Now for each practice calculate the proportion of its prescriptions
containing opioids.

**Hint:** Consider the following list: ``[0, 1, 1, 0, 0, 0]``. What
proportion of the entries are 1s? What is the mean value?

.. code:: ipython3

    prct = practices[['code', 'name']]
    prct.columns = ['practice', 'name']
    prct.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>practice</th>
          <th>name</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>A81002</td>
          <td>QUEENS PARK MEDICAL CENTRE</td>
        </tr>
        <tr>
          <th>1</th>
          <td>A81003</td>
          <td>VICTORIA MEDICAL PRACTICE</td>
        </tr>
        <tr>
          <th>2</th>
          <td>A81004</td>
          <td>WOODLANDS ROAD SURGERY</td>
        </tr>
        <tr>
          <th>3</th>
          <td>A81005</td>
          <td>SPRINGWOOD SURGERY</td>
        </tr>
        <tr>
          <th>4</th>
          <td>A81006</td>
          <td>TENNANT STREET MEDICAL PRACTICE</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    opioids_per_practice = nscript.groupby('practice')['opioid_prescription'].mean().rename('fraction')
    opioids_per_practice.head()
    finalmean = nscript['opioid_prescription'].mean()
    finalstd = nscript['opioid_prescription'].std()
    print(finalstd)
    relative_opioids_per_practice = (opioids_per_practice - finalmean).rename('relative')
    relative_opioids_per_practice
    opioid = nscript.groupby('practice')['opioid_prescription'].sum().rename('opioid')
    #opioid.head()


.. parsed-literal::

    0.18579817605238425


How do these proportions compare to the overall opioid prescription
rate? Subtract off the proportion of all prescriptions that are opioids
from each practice’s proportion.

relative_opioids_per_practice =

Now that we know the difference between each practice’s opioid
prescription rate and the overall rate, we can identify which practices
prescribe opioids at above average or below average rates. However, are
the differences from the overall rate important or just random
deviations? In other words, are the differences from the overall rate
big or small?

To answer this question we have to quantify the difference we would
typically expect between a given practice’s opioid prescription rate and
the overall rate. This quantity is called the **standard error**, and is
related to the **standard deviation**, :math:`\sigma`. The standard
error in this case is

.. math::  \frac{\sigma}{\sqrt{n}} 

where :math:`n` is the number of prescriptions each practice made.
Calculate the standard error for each practice. Then divide
``relative_opioids_per_practice`` by the standard errors. We’ll call the
final result ``opioid_scores``.

.. code:: ipython3

    countpresc = nscript.groupby('practice')['bnf_code'].count().rename('countpresc')
    countpresc.head()
    standard_error_per_practice = (finalstd/(countpresc**0.5)).rename('standard error')
    print(standard_error_per_practice.head())
    opioid_scores = (relative_opioids_per_practice/standard_error_per_practice).rename('opioid_scores')
    opioid_scores.head()


.. parsed-literal::

    practice
    A81005    0.004786
    A81007    0.004873
    A81011    0.004692
    A81012    0.005091
    A81017    0.004007
    Name: standard error, dtype: float64




.. parsed-literal::

    practice
    A81005   -0.548306
    A81007    1.544557
    A81011    2.291795
    A81012    1.373060
    A81017    0.583168
    Name: opioid_scores, dtype: float64



The quantity we have calculated in ``opioid_scores`` is called a
**z-score**:

.. math::  \frac{\bar{X} - \mu}{\sqrt{\sigma^2/n}} 

Here :math:`\bar{X}` corresponds with the proportion for each practice,
:math:`\mu` corresponds with the proportion across all practices,
:math:`\sigma^2` corresponds with the variance of the proportion across
all practices, and :math:`n` is the number of prescriptions made by each
practice. Notice :math:`\bar{X}` and :math:`n` will be different for
each practice, while :math:`\mu` and :math:`\sigma` are determined
across all prescriptions, and so are the same for every z-score. The
z-score is a useful statistical tool used for hypothesis testing,
finding outliers, and comparing data about different types of objects or
events.

Now that we’ve calculated this statistic, take the 100 practices with
the largest z-score. Return your result as a list of tuples in the form
``(practice_name, z-score, number_of_scripts)``. Sort your tuples by
z-score in descending order. Note that some practice codes will
correspond with multiple names. In this case, use the first match when
sorting names alphabetically.

.. code:: ipython3

    combined = pd.concat([opioid, countpresc, opioids_per_practice, relative_opioids_per_practice, standard_error_per_practice,opioid_scores], axis=1)
    combined = combined.reset_index()
    combined.head()
    prct.reset_index().head()
    combinedfinal = combined.merge(prct, on='practice', how='left')
    
    
    combinedfinal.sort_values('opioid_scores', ascending = False , inplace=True)
    combinedfinal = combinedfinal.drop_duplicates('name')
    combinedfinal.head()
    result = combinedfinal[['name','opioid_scores','countpresc']]
    result.head()
    
    result = result.head(100)
    v2list = result.values.tolist()
    final3list = []
    for item in v2list:
        final3list.append(tuple(item))
    
        


.. code:: ipython3

    #unique_practices = 
    anomalies = final3list
    anomalies





.. parsed-literal::

    [('NATIONAL ENHANCED SERVICE', 11.695817862936027, 7),
     ('OUTREACH SERVICE NH / RH', 7.339043019238823, 2),
     ('BRISDOC HEALTHCARE SERVICES OOH', 6.1505817490838295, 60),
     ('H&R P C SPECIAL SCHEME', 5.123032414033079, 36),
     ('HMR BARDOC OOH', 4.958866438487605, 321),
     ('INTEGRATED CARE 24 LTD (CWSX OOH)', 4.8888781604828235, 426),
     ('DARWEN HEALTHCARE', 4.8391589686363385, 1917),
     ('THE LIMES MEDICAL PRACTICE', 4.546841872334426, 1321),
     ('IC24 LTD (BRIGHTON & HOVE OOH)', 4.335047010605197, 357),
     ('OLDHAM 7 DAY ACCESS HUB2 OOH', 4.31178403661019, 56),
     ('IC24 LTD (NORFOLK & WISBECH OOH)', 4.2575005924727645, 489),
     ('ROSSENDALE MIU & OOH', 4.256827446322491, 18),
     ('BURY WALK-IN CENTRE', 4.150589122881536, 138),
     ('IC24 LTD (HORSHAM & MID SUSSEX OOH)', 3.7816207038443523, 215),
     ('LCW HOUNSLOW CCG OOH', 3.582848546206269, 69),
     ('WEEKEND WORKING EASINGTON NORTH', 3.565938771129764, 278),
     ('COMPASS ENFIELD', 3.5587202651067935, 7),
     ('BASSETLAW DRUG & ALCOHOL SERVICE', 3.5332641025115423, 2),
     ('THE PARK SURGERY', 3.511114736389202, 969),
     ('CHAPEL STREET SURGERY', 3.4907459502707447, 1504),
     ('BEECHWOOD MEDICAL PRACTICE', 3.4747953064338977, 1552),
     ('BASSETLAW HOSPICE OF THE GOOD SHEPHERD', 3.454423428527398, 46),
     ('CARDEN SURGERY', 3.4503113243413193, 1375),
     ('GP IN A&E (WIC)', 3.3959177486386096, 87),
     ('NORTHAMPTONSHIRE OUT OF HOURS SERVICE', 3.39355927807245, 382),
     ('NORTHAMPTONSHIRE OOH SERVICE', 3.39355927807245, 382),
     ('WORDEN MEDICAL CENTRE', 3.341328344384246, 1898),
     ('BURY OOH', 3.321529559970347, 292),
     ('IC24 SOUTHEND/CP&R CCG OOH', 3.2211153199464, 336),
     ('SOUTH ESSEX OOH', 3.2211153199464, 336),
     ('IC24 LTD (CRAWLEY OOH)', 3.1942640976202856, 173),
     ('EASTBOURNE  HAILSHAM & SEAFORD OOH', 2.9896925335154507, 255),
     ('THE ROSEBERRY PRACTICE', 2.9803589486570257, 1450),
     ('SHAFTESBURY MEDICAL CTR.', 2.904593003935284, 2126),
     ('THE RICHMOND HILL PRACTICE', 2.8601891978583245, 1729),
     ('DGS OOH (INTEGRATED CARE 24 LTD)', 2.795651191074179, 325),
     ('THE LAKESIDE PRACTICE', 2.7801243484267157, 1554),
     ('DEWSBURY MSK SERVICE', 2.7736443521856917, 3),
     ('HALLIWELL SURGERY 1', 2.6902312553464682, 1124),
     ('THE ROYTON & CROMPTON FAMILY PRACTICE', 2.6674099502870625, 2054),
     ('PRACTICE 3  MEDICAL CENTRE  BRIDLINGTON', 2.6526642039852844, 1936),
     ('LCWUCC - IUC NCL', 2.5927932420214392, 443),
     ('DR MOKASHI', 2.496887689055938, 1341),
     ('THE CREST FAMILY PRACTICE', 2.494358442548829, 1294),
     ('PERKINS PRACTICE', 2.479571342887134, 899),
     ('DARLASTON HEALTH CENTRE', 2.4456345065842653, 996),
     ("DR MA SIMS' PRACTICE", 2.4372766106456107, 1857),
     ('1/LOWER BROUGHTON MEDICAL PRACTICE', 2.435636753514273, 1044),
     ('CORNWALLIS PLAZA SURGERY', 2.424202462055629, 2326),
     ('FAIRMORE MEDICAL PRACTICE', 2.382408598971051, 1288),
     ('LAWLEY MEDICAL PRACTICE', 2.373577973078906, 1409),
     ("PEEL GP'S", 2.3535943384931546, 1557),
     ('PEEL GPS DR JACKSON', 2.3535943384931546, 1557),
     ('PEEL GPS', 2.3535943384931546, 1557),
     ('EASTLANDS MEDICAL CENTRE', 2.325396683274206, 1369),
     ('MAGHULL HEALTH CENTRE (DR SAPRE)', 2.3194074094375097, 1322),
     ('INTEGRATED CARE 24 LIMITED OOH', 2.3106497615538943, 511),
     ('WILLOW BANK SURGERY', 2.295642132037915, 1934),
     ('IC24 LTD (HWLH OOH)', 2.2949004338925127, 216),
     ('CHADWICK PRACTICE', 2.2917945205697308, 1568),
     ('C&WPT OOH SERVICE', 2.270852401273787, 341),
     ('THE MERRYWOOD PRACTICE', 2.270550955639659, 1234),
     ('LEYLAND SURGERY', 2.251774286539937, 1213),
     ('WALKDEN MEDICAL PRACTICE', 2.2415736140706994, 1577),
     ('RADCLIFFE MEDICAL PRACTICE', 2.239607033459532, 1553),
     ('RIBBLESDALE GP-DR SUBBIAH', 2.209745891261483, 1100),
     ('TOWNSIDE SURGERY', 2.209745891261483, 1100),
     ('THE MAZHARI & KHAN PRACTICE', 2.2006627122644895, 912),
     ('ST ANDREWS - BRANSHOLME', 2.1967025909056024, 1634),
     ('JAMES ALEXANDER FAMILY PRACTICE', 2.1967025909056024, 1634),
     ('B&H INTERMEDIATE CARE SERVICE', 2.1862131918267544, 456),
     ('HARDEN SURGERY', 2.155495816284959, 1228),
     ('THE GATEWAY MEDICAL PRACTICE', 2.154423496545371, 1349),
     ('QUAYSIDE MEDICAL PRACTICE', 2.1535448371359025, 1422),
     ('HASTINGS AND ROTHER OOH', 2.076490970994684, 291),
     ('FISHPONDS FAMILY PRACTICE', 2.0743592136603786, 1632),
     ('DRS CLOAK  CHOI AND MILLIGAN', 2.0511156727220916, 1661),
     ("ST MARY'S SURGERY", 2.041026500575296, 981),
     ('THE THORNTON PRACTICE', 1.9970130183083867, 1970),
     ('IC24 LTD (EAST SURREY OOH)', 1.9530599800408153, 194),
     ('CORNISHWAY GROUP PRACTICE', 1.9529030654485127, 1506),
     ('PREMIER HEALTH TEAM', 1.9342677920567377, 876),
     ('CONCORD MEDICAL PRACTICE', 1.9339605744520654, 1386),
     ('COULBY MEDICAL PRACTICE', 1.9268085445885355, 1585),
     ('PARKSIDE MEDICAL CENTRE', 1.9235518278308144, 1314),
     ('THE DISCOVERY PRACTICE', 1.9047661015943373, 1268),
     ('ANCORA MEDICAL PRACTICE', 1.8854159223309173, 2347),
     ('DARWEN HEALTHLINK', 1.8822754121466374, 2019),
     ('JALAL PRACTICE', 1.8661104279692897, 885),
     ('ROOLEY LANE MED. CENTRE', 1.8635817801511316, 1522),
     ('GREAT HOMER STREET MEDICAL CENTRE', 1.813002119840964, 1062),
     ('NORTHPOINT', 1.8024326949572382, 1137),
     ('THE CRESCENT SURGERY', 1.7989165864147443, 1734),
     ('BROOKVALE PRACTICE', 1.7979440966591638, 1484),
     ('LEIGH NHS WIC', 1.7760099128664422, 162),
     ('DR SP SINGH AND PARTNERS', 1.7720416435781805, 1840),
     ('BELGRAVE SURGERY', 1.7643723369327995, 1291),
     ('THE PARKS MEDICAL PRACTICE', 1.7589687048174791, 1466),
     ('THE ROSS PRACTICE', 1.7231557395207378, 1648),
     ('THOMAS WALKER', 1.69692378556541, 1377)]



.. code:: ipython3

    results.head()


::


    ---------------------------------------------------------------------------

    NameError                                 Traceback (most recent call last)

    <ipython-input-30-0f9ba5b4964d> in <module>()
    ----> 1 results.head()
    

    NameError: name 'results' is not defined


.. code:: ipython3

    grader.score.dw__script_anomalies(anomalies)

Question 5: script_growth
-------------------------

Another way to identify anomalies is by comparing current data to
historical data. In the case of identifying sites of drug abuse, we
might compare a practice’s current rate of opioid prescription to their
rate 5 or 10 years ago. Unless the nature of the practice has changed,
the profile of drugs they prescribe should be relatively stable. We
might also want to identify trends through time for business reasons,
identifying drugs that are gaining market share. That’s what we’ll do in
this question.

We’ll load in beneficiary data from 6 months earlier, June 2016, and
calculate the percent growth in prescription rate from June 2016 to
January 2017 for each ``bnf_name``. We’ll return the 50 items with
largest growth and the 50 items with the largest shrinkage
(i.e. negative percent growth) as a list of tuples sorted by growth rate
in descending order in the format
``(script_name, growth_rate, raw_2016_count)``. You’ll notice that many
of the 50 fastest growing items have low counts of prescriptions in
2016. Filter out any items that were prescribed less than 50 times.

.. code:: ipython3

    scripts16 = pd.read_csv('./dw-data/201606scripts_sample.csv.gz')

.. code:: ipython3

    growthrate = (scripts.bnf_name.value_counts() - scripts16.bnf_name.value_counts())/scripts16.bnf_name.value_counts()
    resp = pd.DataFrame(dict(growthrate = growthrate, item16 = scripts16.bnf_name.value_counts())).reset_index()
    resp.fillna(0, inplace=True)
    resp = resp[resp['item16'] >= 50]
    resp.sort_values(by='growthrate', ascending = False, inplace=True)
    script_growth = list(pd.concat([resp.head(50), resp.tail(50)]).itertuples(index=False, name=None))


.. code:: ipython3

    grader.score.dw__script_growth(script_growth)

Question 6: rare_scripts
------------------------

Does a practice’s prescription costs originate from routine care or from
reliance on rarely prescribed treatments? Commonplace treatments can
carry lower costs than rare treatments because of efficiencies in
large-scale production. While some specialist practices can’t help but
avoid prescribing rare medicines because there are no alternatives, some
practices may be prescribing a unnecessary amount of brand-name products
when generics are available. Let’s identify practices whose costs
disproportionately originate from rarely prescribed items.

First we have to identify which ``'bnf_code'`` are rare. To do this,
find the probability :math:`p` of a prescription having a particular
``'bnf_code'`` if the ``'bnf_code'`` was randomly chosen from the unique
options in the beneficiary data. We will call a ``'bnf_code'`` rare if
it is prescribed at a rate less than :math:`0.1p`.

.. code:: ipython3

    unique_values = practices.sort_values(['code', 'post_code']).drop_duplicates(subset= 'code')
    unique_values.head()
    script_by_bnf_code = scripts.groupby(['bnf_code'])['bnf_code'].count().reset_index(name='bnf_code_count')
    script_by_bnf_code['p'] = script_by_bnf_code['bnf_code_count'] / len(scripts)

.. code:: ipython3

    p = 1 / len(script_by_bnf_code)
    script_by_bnf_code['rare'] = np.where(script_by_bnf_code['p'] < 0.1 * p, True, False)
    #rates = ...
    #rare_codes = 
    scripts_rare = pd.merge(scripts, script_by_bnf_code, how = 'inner', on = 'bnf_code')
    scripts_rare.head()
    #script_by_bnf_code.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>practice</th>
          <th>bnf_code</th>
          <th>bnf_name</th>
          <th>items</th>
          <th>nic</th>
          <th>act_cost</th>
          <th>quantity</th>
          <th>opioid_prescription</th>
          <th>bnf_code_count</th>
          <th>p</th>
          <th>rare</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>N85639</td>
          <td>0106020C0</td>
          <td>Bisacodyl_Tab E/C 5mg</td>
          <td>1</td>
          <td>0.39</td>
          <td>0.47</td>
          <td>12</td>
          <td>False</td>
          <td>1035</td>
          <td>0.001064</td>
          <td>False</td>
        </tr>
        <tr>
          <th>1</th>
          <td>N81013</td>
          <td>0106020C0</td>
          <td>Bisacodyl_Tab E/C 5mg</td>
          <td>19</td>
          <td>28.10</td>
          <td>28.14</td>
          <td>860</td>
          <td>False</td>
          <td>1035</td>
          <td>0.001064</td>
          <td>False</td>
        </tr>
        <tr>
          <th>2</th>
          <td>N81029</td>
          <td>0106020C0</td>
          <td>Bisacodyl_Tab E/C 5mg</td>
          <td>50</td>
          <td>67.73</td>
          <td>67.51</td>
          <td>2074</td>
          <td>False</td>
          <td>1035</td>
          <td>0.001064</td>
          <td>False</td>
        </tr>
        <tr>
          <th>3</th>
          <td>N81029</td>
          <td>0106020C0</td>
          <td>Bisacodyl_Suppos 10mg</td>
          <td>1</td>
          <td>1.77</td>
          <td>1.75</td>
          <td>6</td>
          <td>False</td>
          <td>1035</td>
          <td>0.001064</td>
          <td>False</td>
        </tr>
        <tr>
          <th>4</th>
          <td>N81029</td>
          <td>0106020C0</td>
          <td>Dulcolax_Tab 5mg</td>
          <td>1</td>
          <td>3.78</td>
          <td>3.61</td>
          <td>56</td>
          <td>False</td>
          <td>1035</td>
          <td>0.001064</td>
          <td>False</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    costofprac = scripts_rare.groupby('practice')['act_cost'].sum().reset_index(name = 'tpraccost')
    rarecost = scripts_rare[scripts_rare['rare'] ==True].groupby('practice')['act_cost'].sum().reset_index(name = 'rare_cost')
    script_cost_by_practice = pd.merge(costofprac, rarecost, on = 'practice', how='left')
    script_cost_by_practice.fillna(0, inplace=True)
    script_cost_by_practice['rare_cost_prop'] = (script_cost_by_practice['rare_cost'] / script_cost_by_practice['tpraccost'])
    script_cost_by_practice.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>practice</th>
          <th>tpraccost</th>
          <th>rare_cost</th>
          <th>rare_cost_prop</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>A81005</td>
          <td>103840.82</td>
          <td>1247.83</td>
          <td>0.012017</td>
        </tr>
        <tr>
          <th>1</th>
          <td>A81007</td>
          <td>113482.49</td>
          <td>951.06</td>
          <td>0.008381</td>
        </tr>
        <tr>
          <th>2</th>
          <td>A81011</td>
          <td>159507.03</td>
          <td>816.02</td>
          <td>0.005116</td>
        </tr>
        <tr>
          <th>3</th>
          <td>A81012</td>
          <td>83296.81</td>
          <td>1145.11</td>
          <td>0.013747</td>
        </tr>
        <tr>
          <th>4</th>
          <td>A81017</td>
          <td>232656.17</td>
          <td>1712.15</td>
          <td>0.007359</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    total_cost = scripts_rare['act_cost'].sum()
    total_rare_cost = scripts_rare[scripts_rare['rare'] == True]['act_cost'].sum()
    
    overall_rare_cost = total_rare_cost / total_cost
    script_cost_by_practice['relative_rare_cost_prop'] = script_cost_by_practice['rare_cost_prop'] - overall_rare_cost

.. code:: ipython3

    script_cost_by_practice['standard_errors'] = script_cost_by_practice['relative_rare_cost_prop'].std()
    script_cost_by_practice.head()




.. raw:: html

    <div>
    <style scoped>
        .dataframe tbody tr th:only-of-type {
            vertical-align: middle;
        }
    
        .dataframe tbody tr th {
            vertical-align: top;
        }
    
        .dataframe thead th {
            text-align: right;
        }
    </style>
    <table border="1" class="dataframe">
      <thead>
        <tr style="text-align: right;">
          <th></th>
          <th>practice</th>
          <th>tpraccost</th>
          <th>rare_cost</th>
          <th>rare_cost_prop</th>
          <th>relative_rare_cost_prop</th>
          <th>standard_errors</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th>0</th>
          <td>A81005</td>
          <td>103840.82</td>
          <td>1247.83</td>
          <td>0.012017</td>
          <td>-0.003946</td>
          <td>0.060509</td>
        </tr>
        <tr>
          <th>1</th>
          <td>A81007</td>
          <td>113482.49</td>
          <td>951.06</td>
          <td>0.008381</td>
          <td>-0.007582</td>
          <td>0.060509</td>
        </tr>
        <tr>
          <th>2</th>
          <td>A81011</td>
          <td>159507.03</td>
          <td>816.02</td>
          <td>0.005116</td>
          <td>-0.010847</td>
          <td>0.060509</td>
        </tr>
        <tr>
          <th>3</th>
          <td>A81012</td>
          <td>83296.81</td>
          <td>1145.11</td>
          <td>0.013747</td>
          <td>-0.002216</td>
          <td>0.060509</td>
        </tr>
        <tr>
          <th>4</th>
          <td>A81017</td>
          <td>232656.17</td>
          <td>1712.15</td>
          <td>0.007359</td>
          <td>-0.008604</td>
          <td>0.060509</td>
        </tr>
      </tbody>
    </table>
    </div>



.. code:: ipython3

    script_cost_by_practice['z_scores'] = script_cost_by_practice['relative_rare_cost_prop'] / script_cost_by_practice['standard_errors']
    print(script_cost_by_practice.head())
    print(script_cost_by_practice.shape)
    practices_unique = practices.sort_values('name').groupby('code', sort=False).first()
    practices_unique.reset_index(inplace=True)
    merge_script_practice_z_scores = script_cost_by_practice.merge(
        practices_unique[['name', 'code']], how = 'inner', left_on = 'practice', right_on = 'code', sort = False)
    
    print(merge_script_practice_z_scores.shape)
    merge_script_practice_z_scores = merge_script_practice_z_scores.sort_values('name', ascending = True)
    merge_script_practice_z_scores.drop_duplicates(subset = ['code', 'name'], inplace = True)
    
    rare_scripts = []
    for index, row in merge_script_practice_z_scores.iterrows():
        rare_scripts.append(
            (row['practice'], 
             row['name'],
             row['z_scores']))
    
    rare_scripts = sorted(rare_scripts, key=lambda x: x[2], reverse = True)[:100]


.. parsed-literal::

      practice  tpraccost  rare_cost  rare_cost_prop  relative_rare_cost_prop  \
    0   A81005  103840.82    1247.83        0.012017                -0.003946   
    1   A81007  113482.49     951.06        0.008381                -0.007582   
    2   A81011  159507.03     816.02        0.005116                -0.010847   
    3   A81012   83296.81    1145.11        0.013747                -0.002216   
    4   A81017  232656.17    1712.15        0.007359                -0.008604   
    
       standard_errors  z_scores  
    0         0.060509 -0.065216  
    1         0.060509 -0.125308  
    2         0.060509 -0.179263  
    3         0.060509 -0.036615  
    4         0.060509 -0.142190  
    (856, 7)
    (856, 9)


Now for each practice, calculate the proportion of costs that originate
from prescription of rare treatments (i.e. rare ``'bnf_code'``). Use the
``'act_cost'`` field for this calculation.

.. code:: ipython3

    rare_cost_prop = ...

Now we will calculate a z-score for each practice based on this
proportion. First take the difference of ``rare_cost_prop`` and the
proportion of costs originating from rare treatments across all
practices.

.. code:: ipython3

    relative_rare_cost_prop = ...

Now we will estimate the standard errors (i.e. the denominator of the
z-score) by simply taking the standard deviation of this difference.

standard_errors = …

Finally compute the z-scores. Return the practices with the top 100
z-scores in the form ``(post_code, practice_name, z-score)``. Note that
some practice codes will correspond with multiple names. In this case,
use the first match when sorting names alphabetically.

rare_scores = …

rare_scripts = [(“Y03472”, “CONSULTANT DIABETES TEAM”, 16.2626871247)]
\* 100

.. code:: ipython3

    grader.score.dw__rare_scripts(rare_scripts)


.. parsed-literal::

    ==================
    Your score:  1.0
    ==================


*Copyright © 2020 The Data Incubator. All rights reserved.*
