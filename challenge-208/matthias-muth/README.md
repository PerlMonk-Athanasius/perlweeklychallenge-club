# Juggling with indexes.
*Challenge 208 solutions in Perl by Matthias Muth*

## Task 1: Minimum Index Sum

> You are given two arrays of strings.<br/>
> Write a script to find out all common strings in the given two arrays with minimum index sum. If no common strings found returns an empty list.

Let's take one step at a time for this one:

'All common strings':<br/>
For checking whether a string is contained in both arrays, we use a typical Perl pattern and create an 'existence' hash from the first array's strings.
Later we can go through the strings from the second array
and check for each one whether it exists in the first array
by simply checking the existence of a hash entry for that string.<br/>
The typical Perl pattern to create that hash looks like this:
```perl
    my %index1 = map { ( $list1[$_] => 1 ) } 0..$#list1;
```
Actually, as we will also need the strings *index* within the first array later, we don't store the typical `1`, but that index value:
```perl
    my %index1 = map { ( $list1[$_] => $_ ) } 0..$#list1;
```
We only need to create this hash for the first array of strings.
For the second one we can loop over the strings, using the second string index as the loop index.

'Minimum index sum':<br/>
So we need the sum of the two indexes into the first and the second array for every string,
and we need to store all those sums to later find their minimum.<br/>
And we need to keep the information about which string generated each sum.<br/>
So why don't we create another hash, this time using the *sum* as the key, and the string as the value?<br/>
For finding the minimum in the end, we then can do `min( keys %strings_by_index_sum )`.<br/>
And the strings to be returned are the `value` of that minimum's hash entry.<br/>
String**s**? Plural?<br/>
Oh, yes, the same index sum can be generated by more that one string (this case exists in the examples!).
So we should not store a string as the value, but an arrayref,
onto which we push whichever string generates that index sum.
Looks like this:
```perl
    my %strings_by_index_sum;
    for ( 0..$#list2 ) {
        if ( exists $index1{ $list2[$_] } ) {
            my $index_sum = $index1{ $list2[$_] } + $_;
            push @{$strings_by_index_sum{$index_sum}}, $list2[$_];
        }
    }
```

Now it's time to return what we found.<br/>
As already explained, we get the minimum of the keys of our special hash, and return the strings in that hash entry.<br/>
To avoid an empty list in the `min(...)` call (which leads to a warning),
we guard this computation by checking whether anything was found at all, returning an empty list if not.<br/>
So:
```perl
    return
        %strings_by_index_sum
        ? sort @{$strings_by_index_sum{ min( keys %strings_by_index_sum ) } }
        : ();
```
Putting everything together:

```perl
sub min_index_sum {
    my @list1 = @{$_[0]};
    my @list2 = @{$_[1]};
    my %index1 = map { ( $list1[$_] => $_ ) } 0..$#list1;
    my %strings_by_index_sum;
    for ( 0..$#list2 ) {
        if ( exists $index1{ $list2[$_] } ) {
            my $index_sum = $index1{ $list2[$_] } + $_;
            push @{$strings_by_index_sum{$index_sum}}, $list2[$_];
        }
    }
    return
        %strings_by_index_sum
        ? sort @{$strings_by_index_sum{ min( keys %strings_by_index_sum ) } }
        : ();
}
```

## Task 2: Duplicate and Missing

>You are given an array of integers in sequence with one missing and one duplicate.<br/>
Write a script to find the duplicate and missing integer in the given array. Return -1 if none found.<br/>
For the sake of this task, let us assume the array contains no more than one duplicate and missing.

In this task, for finding a duplicate value we need to go through the array and compare every value to the previous one.<br/>
This immediately makes me think of using `List::Util`'s `reduce` function,
which already does the job of looping over the array for us,
as well as the job of handing us two values at a time (the `$a`and `$b` special variables) for use in a code block that we supply.

Within that code block, we can check for a duplicate value and set a variable if we found one,
Similarly, we can also check for a missing value between the previous entry and the current one:
```perl
    reduce {
        $dup     = $b     if $b == $a;
        $missing = $b - 1 if $a < $b - 1;
        $b;
    } @_;
```
(We mustn't forget to return `$b` from the code block, it will be the next iteration's `$a`.)

There is a special case where we might 'miss' the missing value:<br/>
When the duplicate values happen to be at the end of the array,
the 'missing' value is the one that is now hidden by the repeated value in the last position.

We can assume that that value is the missing one if we know that there is a duplicate
(and the rules state that there will be *at maximum* one duplicate),
and if we haven't detected a missing value before.<br/>
If we did *not* find a duplicate when we arrive there, we need to return `-1` anyways,
so in that case we don't need to worry about whether we have a missing value, and which one, at all.

To summarize this solution:

```perl
use List::Util qw( reduce );

sub dup_and_missing {
    my ( $dup, $missing );
    reduce {
        $dup     = $b     if $a == $b;
        $missing = $b - 1 if $a < $b - 1;
        $b;
    } @_;
    return
        defined $dup
        ? ( $dup, $missing // ( $_[-1] + 1 ) )
        : -1;
}
```

**Thank you for the challenge!**

