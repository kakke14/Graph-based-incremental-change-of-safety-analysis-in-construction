# Graph-based-incremental-change-of-safety-analysis-in-construction
This repository consist of files that are used to create and query the knowledge base in ##


## Description of content
  * **mapping.pl** consist of predicates that allow the user to map from local IDs (integer-based) to IFC Guid (String-based)
  * **parent_relation.pl** consists of predicates that capture the relations (i.e., edges) between elements (nodes)
  * **schedule.pl** consist of the predicates that defines relations between BIM objects and the State nodes
  * **products.pl** consist of the predicates that describes the spatial artefact and bim object nodes
  * **graph.html** contains a browserbased visualization of the resulting graph network

## How to get started
  1. Install a prolog environment such as [SWI-prolog](https://www.swi-prolog.org)
  2. run swipl *.pl in this folder, which loads all the shared instance files

## Query examples
=============================================

QUERY: When did X first appear? When did X stop changing?

(note - "setof" sorts the list of results)

?-

    product(movement_space, XLocal),
    setof(T, P^(
    relation(parent_additive, P, XLocal),
    installed_at(P, T)),
    [XExistsAt|_]),

    setof(T, P^Rel^(
    relation(Rel, P, XLocal),
    (Rel = parent_additive ; Rel = parent_subtractive),
    installed_at(P, T)),
    ParentTimes),
    last(ParentTimes, XStoppedChangingAt).


ANSWER:

    XLocal = 634,
    XExistsAt = XStoppedChangingAt, XStoppedChangingAt = 1,
    ParentTimes = [1] ;
    XLocal = 635,
    XExistsAt = 1,
    ParentTimes = [1, 2],
    XStoppedChangingAt = 2 ;


=============================================

QUERY: Which BIM model products P contribute to the existence and shape of X? When?

?-

    product(movement_space, XLocal),
    relation(Rel, P, XLocal),
    (Rel = parent_additive ; Rel = parent_subtractive),
    installed_at(P, When),
    product(Type, P), mapping_ifc_to_local(IFC_GUID, P).

ANSWER:

    XLocal = 634,
    Rel = parent_additive,
    P = 50,
    When = 1,
    Type = bim_object,
    IFC_GUID = '01U2Ox69TF78CjGAzXHAWU'

    ...

=============================================

QUERY: Did X result from a merge/split between two other artefacts, and when?

(note - these two queries are just examples of what can be expressed, they don't capture split/merge in all situations)

%% case: split occurs when two artefacts share a subtractive parent

?-

    product(fall_space, A),
    product(fall_space, B),
    A < B,
    relation(parent_subtractive, PSubtract, A),
    relation(parent_subtractive, PSubtract, B),
    installed_at(PSubtract, SplitT).

ANSWER:

    A = 648,
    B = 649,
    PSubtract = 104,
    SplitT = 2 

    ...

=============================================

QUERY: How many products and spatial artefacts exist at time T? Which ones?

?-

    (T=3 ; T=8),
    setof(P,
    TP^(installed_at(P, TP), TP =< T),
    ExistsAtT),
    length(ExistsAtT, CountExistsAt).

ANSWER:

    T = 3,
    ExistsAtT = [2, 3, 4, 5, 6, 7, 8, 9, 10|...],
    CountExistsAt = 106 ;
    T = 8,
    ExistsAtT = [2, 3, 4, 5, 6, 7, 8, 9, 10|...],
    CountExistsAt = 435.

