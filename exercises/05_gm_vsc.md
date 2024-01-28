1) Show that P is the weakest failure detector for Group Membership.
Note: the failure detector D is weakest for solving some problem A (e.g.,
Consensus or NBAC) if D provides the smallest amount of information about
failures that allows to solve A.
Hint: Reduce Group Membership to Perfect Failure Detector and vice versa.


In order to show that P is the weakest failure detector for Group Membership, we
need to that that:
1. P can be used to implement Group Membership.
2. Group Membership can be used to implement P.

For (2), assume that all processes run a GM algorithm. We can implement a
perfect failure detector as follows:
Whenever a new view is installed, all processes that are freshly removed from the view are added to the
detected set.
This approach satisfies both Strong Completeness and Strong Accuracy, directly from the corresponding
properties of GM.


