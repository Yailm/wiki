
floating point arithmetic
------------------------------------

#### some issues:

    a = 18561861982347651    # a big number
    (a-1)/a == 1             # True
    4.2+2.1 == 6.3           # False

#### solution:

    from decimal import Decimal
    a = 18613845618237681
    Decimal(a-1) / a == 1    # False
    Decimal((a-1)/a) == 1    # True, don't apply to result simply
