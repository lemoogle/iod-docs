This How To will highlight the operations available when doing searches
on IDOL OnDemand. For simplicity and to let you try the queries on IOD,
most of the examples below are assuming that the **english Wikipedia
public dataset** is being queried

Search Capabilities
-----------------


Conceptual Search
====================

The **Query Text Index API** allows you to search for content in the
IDOL OnDemand databases. It leverages the power of IDOL to return
conceptually relevant results without requiring training but simply by
using a statistical understanding of the data and the terms within it.
The **Get Parametric Values**, **Find related concepts** and **Find
Similar Documents** Apis also make use of IDOL's conceptual capabilities

.. http:post:: :: /querytextindex/v1?text=Red+Panda&database\_match=wiki\_eng

.. code-block:: javascript

    {
    "documents":[
        {
          "reference": "http://en.wikipedia.org/wiki/Red Panda Adventures",
          "weight": 84.59,
          "links": [
            "RED",
            "PAND"
          ],
          "index": "wiki_eng",
          "title": "Red Panda Adventures",
          "content": {}
        },
        {
          "reference": "http://en.wikipedia.org/wiki/Red panda",
          "weight": 84.59,
          "links": [
            "RED",
            "PAND"
          ],
          "index": "wiki_eng",
          "title": "Red panda",
          "content": {}
        }
      ]
      }

IDOL indexes assign weights to terms based on their relevance to the
dataset. As a simplified example, if every document in your index is
about *red* things, then the term *red* should not be as relevant in
your query than the term *Panda*. IDOL ranks result based on various
factors like the weights of the terms in the index, frequency within the
returned documents, stemmed proximity. However it does not operate on a
*match all terms* basis and actually operates better with more text
rather than less, documents will be returned based on how closely they
relate to the entire query .

**The Text Parameter**

As we notice in the above query, the *text* parameter is a common
parameter to all the APIs mentioned above and is the source of all the
conceptual term matching and weighting that IDOL OnDemand applies to the
results.

Search Operators
~~~~~~~~~~~~~~~~~~~~~~

**Wildcard search**

.. http:post:: :: /querytextindex/v1?text=Obam\*

the ``*`` operator operates as the usual wildcard operator. In our
example we search for any term that start with Obam. **Tip:** don't run
it on letters such as ``a*`` as the query will take a long time to
return.

**Occurence search**

.. http:post:: :: /querytextindex/v1?text=Gene[3:5]

By specifying a column separated range of numbers within curly brackets,
we are able to specify the number of occurences we are looking for. In
this example, only the documents containing Gene between 3 and 5 times
will be returned.

**Exact Term Match**

.. http:post:: :: /querytextindex/v1?text="lovely"

By putting a term in between quotes, you can ensure that the exact
version of the word will be looked for and not other versions that share
the same stem like *lovely, love, loved, loving* etc.

**Exact Match - Case Sensitivity** > /querytextindex/v1?text="~Apple"

The ``~`` character can be used within quotes to ensure case
sensitivity. In our case we only want to return documents that have
Apple capitalized.

**Phrase search** > /querytextindex/v1?text="Red+Panda"

Quotes enable exact *phrase* search.

Boolean Operators
~~~~~~~~~~~~~~~~~~~~~~


**Boolean Operators**

The text Parameter offers a level of boolean search or bracketed boolean
Search.

.. http:post:: :: /querytextindex/v1?text="Red+Panda" **AND** Elephant

Boolean operator need to be capitalized , we can search for documents
containing both red pandas and elephants.

.. http:post:: :: /querytextindex/v1?text=(Panda **OR** Elephant)\ **AND** Lion

We can use brackets to group together elements and ensure the correct
order in which the operators should be applied. In this casee want any
document that mentions lions AND either pandas or elephants

.. http:post:: :: /querytextindex/v1?text=cat **XOR** dog

XOR stands for exclusive OR and will return documents that contain
either *cat* or *dog* but NOT the two at the same time

.. http:post:: :: /querytextindex/v1?text=cat **NOT** dog
.. http:post:: :: /querytextindex/v1?text=(cat **NOT** dog) **OR** (dog **NOT** cat)

The NOT operator can restrict your searches NOT to include certain
terms. In the example above we show how to return documents about cats
that don't include dog in the result. The second example shows how to
replicate an XOR without the NOT and OR operator

Further information on Boolean Operators is available in the
documentation section of the site : `Boolean and Proximity
Operators <https://www.idolondemand.com/developer/docs/BooleanProximityOperators.html>`__

Proximity and order Operators
~~~~~~~~~~~~~~~~~~~~~~


**Sentence and Paragraph search**

.. http:post:: :: /querytextindex/v1?text=cat SENTENCE dog /querytextindex/v1?text=cat
    PARAGRAPH dog

The *SENTENCE* and *PARAGRAPH* operators allow you to make sure that two
terms you are searching for are either in the same sentence or the same
paragraph.

**Order Operators**

.. http:post:: :: /querytextindex/v1?text=cat BEFORE dog /querytextindex/v1?text=cat
    AFTER dog

The *BEFORE* and *AFTER* operators act like the *AND* operator and
ensures that both terms are in the document result but it also will
check that cat appears *BEFORE* dog or *AFTER* dog in the 2nd case.

**Term Proximity**

.. http:post:: :: /querytextindex/v1?text=(monkey NEAR4 red)

The NEAR\ *N* allows you to specify a maximum word distance between the
terms you are querying for. In our example, the words monkey and red
need to be within 4 words of each other. This allows us to make sure
that the term red is associated closely with the term spider, returning
documents such as *Red-tailed monkey* or *Red-faced spider monkey*

.. http:post:: :: /querytextindex/v1?text=(red DNEAR3 monkey)

DNEAR\ *N* is in fact a directed NEAR\ *N* , it ensures that the terms
appear in that order like the *BEFORE* operator , but also restricts the
number of words that can appear in between.

NEAR\ *N* and DNEAR\ *N* are very useful to associate adjectives with
nouns as doing an exact search for *"Red monkey"* won't return
*Red-tailed Monkey* and an unrestricted search for *red AND monkey* will
return any document that include both terms , i.e. a red berry that is
eaten by monkeys.

Advanced Search operations
~~~~~~~~~~~~~~~~~~~~~~


**Precedence of operators**

Boolean and Proximity operators will be applied in the following order:

::

    First:      NOT
                     NEAR; DNEAR; XNEAR; YNEAR
                     AND; BEFORE; AFTER; WHEN; SENTENCE; PARAGRAPH
    Last:       OR; WNEAR; EOR

Operators that have the same level of precedence have neither left or
right associativity. You should use brackets to bind terms together as
appropriate. Proximity operators must have terms on either side and
cannot be adjacent to brackets.

**Searching specific index fields**

.. http:post:: :: /querytextindex/v1?text=galaxy:DRETITLE /querytextindex/v1?text=("LA
    galaxy"):DRETITLE /querytextindex/v1?text=("LA galaxy"):DRETITLE AND
    Beckham

A column separating your search terms or bracketed search function will
ensure that only the specified *index type* field is used. These field
specific searches can also be used within more complicated boolean
formulae.

**Using multipliers to modify term weights**

.. http:post:: :: /querytextindex/v1?text=Red Panda[\*5]

You can adjust the weight of a specific term by specifying a multipliers
by which the weight of the term in your query will be multiplied. In our
example we want Panda to have 5 times its usual weight. It can be useful
if we get too many results about Red things that aren't really about
Pandas.

**Setting manual term weights at the query level**

.. http:post:: :: /querytextindex/v1?text=Red[10] Panda[20]

It is also possible to set manual weights through the use of square
brackets. In our case we are setting the weight of the term Panda to 20
and the weight of the term Red to 10.

**Applying weights to bracketed expressions**

.. http:post:: :: /querytextindex/v1?text= (Spider Monkey)[\*3] OR Tiger

We can multiply the weights of any bracketed expression to adjust
relevance , in this case we assign a triple weight to spider and monkey
terms.

Facetted Search and Field Matching
====================

Documents generally do not only contain free text, but also custom
fields like category tags, prices, authors, or any other value that may
be relevant to associate with the text. In this section we will have a
look at the different types of values and how they can be used.

**The fieldtext Parameter**

All of the IDOL APIs that include the text parameter for conceptual
search also have a *fieldtext=* parameter. This parameter is where the
user can define rules and restrictions that need to apply on these
custom fields of the documents.

Facets - Parametric Fields
~~~~~~~~~~~~~~~~

Wikipedia data contains some fields of type Parametric, these are fields
which have values that can be listed and counted based on a query using
the **Get Parametric Values API**.

The parametric fields available are *wikipedia\_type*,
*person\_profession* for people, *place\_country\_code* for places,
*company\_exchange* for companies

The **Get Parametric Values API** allows us to retrieve the unique
values that occur in a particular field, which can then be used to
provide faceted search.

For example, with a color parametric field, you can use te API to
retrieve all the color values that occur in a search, and the
corresponding counts. A common use for this information is to provide
filters to a search

.. http:post:: :: /getparametricvalues/v1?index=wiki\_eng&field\_name=wikipedia\_type

.. code-block:: javascript

    {
      "WIKIPEDIA_TYPE": {
        "PERSON": 1126163,
        "PLACE": 515896,
        "MUSICAL ALBUM": 113211,
        "SPECIES": 242170,
        "COMPANY": 77766,
        "FILM": 94852,
        "SONG": 55061,
        "BOOK": 46433,
        "VIDEO GAME": 16559,
        "GEOGRAPHICAL FEATURE": 90180,
        "PLAY": 6670
      }
    }

In this example we use the wikipedia dataset and the WIKIPEDIA\_TYPE to
offer total counts for each WIKIPEDIA\_TYPE Value.

**Facetted Search**

The **Get Parametric Values API** offers many of the search
functionalities of the **Query Text Index API** such as the *text=*
parameter as well as the *fieldtext=* operator, which will discuss
further in this How To

This means that if I want to run the same query as above but only on
documents about cats and dogs, I would simply have to run

.. http:post:: :: /getparametricvalues/v1?index=wiki\_eng&field\_name=wikipedia\_type&**text=**\ cats AND dogs
    .. code-block:: javascript

        {
          "WIKIPEDIA_TYPE": {
            "PERSON": 32738,
            "MUSICAL ALBUM": 4885,
            "BOOK": 3704,
            "FILM": 6012,
            "COMPANY": 3119,
            "SONG": 2079,
            "VIDEO GAME": 1158,
            "PLACE": 6280,
            "GEOGRAPHICAL FEATURE": 1302,
            "SPECIES": 2835,
            "PLAY": 441
          }
        }

These fields can be used in search implementations to provide a list of
all the available values for certain queries so that queries can then be
filtered based on these *facets*

Text match selectors
~~~~~~~~~~~~~~~~~~~~

**Matching a single value**

.. http:post:: :: /querytextindex/v1?text=Painting&**fieldtext=**\ MATCH{PERSON}:WIKIPEDIA\_TYPE

Since we have shown above that PERSON is a parametric entry for the
WIKIPEDIA\_TYPE field, we can run a search for all the documents that
relate to painting that are of type *PERSON*.

.. http:post:: :: /getparametricvalues/v1?index=wiki\_eng&field\_name=**person\_profession**\ &text=painting&\ **fieldtext=**\ MATCH{PERSON}:WIKIPEDIA\_TYPE

The previous query did not give us all the *painters* however, so we can
run another parametric query to find the list of the professions that
get returned from our query of Persons related to painting.

.. code-block:: javascript

    {
      "PERSON_PROFESSION": {
        "PAINTER": 4409,
        "PHOTOGRAPHER": 52,
        ...
       }
    }

It seems Painter is the value that we want

.. http:post:: :: /querytextindex/v1?text=Painting&**fieldtext=**\ MATCH{PERSON}:WIKIPEDIA\_TYPE
    **AND** MATCH{PAINTER}:PERSON\_PROFESSION

While it may be redundant in this case, as the entries with a
PERSON\_PROFESSION should also be of type PERSON, we can use boolean
operators in the fieltext parameter. Our example if finally retrieving
all the Painters related to *painting* .

**Matching multiple values**

.. http:post:: :: /querytextindex/v1?text=Painting&**fieldtext=**\ MATCHALL{PAINTER,SCULPTOR}:PERSON\_PROFESSION

It could be that we want to get all the people who were BOTH painters
and sculptors, the MATCHALL operator ensure that each of the values
specified has a match in the documents returned.

**Matching a value exclusively**

.. http:post:: :: /querytextindex/v1?text=Painting&**fieldtext=**\ MATCHCOVER{PAINTER}:PERSON\_PROFESSION

Should we want the people who were ONLY painters, the MATCHCOVER
specifier will make sure that all values of the PERSON\_PROFESSION field
match *PAINTER*

**Excluding Matches**

.. http:post:: :: /querytextindex/v1?text=Painting&**fieldtext=**\ NOTMATCH{PAINTER}:PERSON\_PROFESSION

NOTMATCH does what it says, the value specified will NOT be present in
any of the specified fields for the results returned.

Numeric Search
====================

When dealing with Numeric type fields, many numeric operations are
possible

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**GREATER{1000000}**:PLACE\_POPULATION

We can look for places with more than a million people.

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**LESS{100000}**:PLACE\_POPULATION

We can look for places with less than 100000 people.

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**EQUAL{1061235}**:PLACE\_POPULATION

We can look for documents with population exactly equal to 1061235.

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**NRANGE{12,26}**:PLACE\_POPULATION

We can look for documents with population between 12 and 26

Geo/Coordinate Search
====================

Numeric fields can define many things, like population or prices,
however, two numeric fields paired together can also indicate
coordinates.

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=\ **DISTCARTESIAN**\ {40,-100,2}:X:Y
.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=\ **DISTCARTESIAN**\ {x,y,radius}:X:Y

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=\ **DISTSPHERICAL**\ {40,-100,2}:LAT:LON
.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=\ **DISTSPHERICAL**\ {lat,lon,radius in KM}:LAT:LON

Places indexed in the wikipedia dataset have LAT and LON fields,
indicating their approximate coordinates. DISTCARTESIAN will treat the
coordinates as it would in a two-dimensional plane with the distance
being the units on that plane. However DISTSPHERICAL will assume it is
provided spherical coordinates and a radius in KM, it is very useful to
find places in the vicinity of another.

Date Search
====================

Date fields allow for useful date filtering on the results.

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**GTNOW{}**:CREATED\_DATE
.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**LTNOW{}**:CREATED\_DATE

*GTNOW* and *LTNOW* let you restrict for articles created in the past ,
or the future !

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**RANGE{-7,0}**:CREATED\_DATE

The RANGE operator lets you specify exact time range for which the
specified DATE field will be checked against. In our example we want
CREATED\_DATE to be within 7 days in the past.

Other allowed date syntaxes are as below:

-  *D+/M+/#YY+* , *HH:NN:SS D+/M+/#YY+* *HH:NN:SS D+/M+/#YY+ #ADBC* Are
   allowed date formats for the operator
-  N : for number of days , as in the example above
-  Ne : for epoch times
-  Ns : for a negative or positive number of seconds from now.

Other Operations
====================



**Boolean operators in fieldtext**

The fieldtext operator supports the three basic boolean operators NOT,
AND and OR.

.. http:post:: :: /querytextindex/v1?text=Painting&fieldtext=MATCH{PERSON}:WIKIPEDIA\_TYPE **AND** MATCH{PAINTER}:PERSON\_PROFESSION

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**GREATER{1000000}**:PLACE\_POPULATION **OR** **DISTCARTESIAN**\ {50,-10,2}:LAT:LON

**Field Existence**

.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**EXISTS{}**:PLACE\_POPULATION
.. http:post:: :: /querytextindex/v1?text=\*&fieldtext=**EMPTY{}**:PLACE\_POPULATION

The EXISTS Operator allows us to ensure that a field is present in the
result. Here we are only returning documents that have the field
PLACE\_POPULATION. The oppositve EMPTY will return results if the field
value is empty or if the field doesn't exist.

Note: NOT+EXISTS{} will return results only if the field doesn't exist (
an empty value counts as EXISTS )
