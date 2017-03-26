### Golf

想了解代码的内容，`perl -MO=Deparse < code.pl`

#### Join lines
```perl
#!perl -p
chop
```
#### Duplicate lines
```perl
#!perl -p
print
```
#### Odd lines
```perl
#!perl -p
<>
```
#### Factorial
```perl
use bigint;print+($_*1)->bfac,$/for<>
```
#### Permission list
```perl
printf"%04o is -$_
",$n++for glob'{-,r}{-,w}{-,x}'x3
```
#### Leap year
```perl
#!perl -p
s/00$|$/$& is a leap year./;3&$`&&s&s&s not&
```
#### Reverse entire input
```perl
print~~reverse<>
```
```perl
print''.reverse<>
```
#### Sum input
```perl
$\+=<>for%!;print
```
    #!perl -p
    $\+=$_}{

```perl
map$\+=$_,<>;print
```
#### Transporse

Reflect the input around a line from top left to bottom right
```perl
#!perl -ln0
print/^./mgwhile s///g
```
#### Rot13

Rotate ascii 13
```perl
#!perl -p
y/A-z/N-ZA-Sn-za-m/
```
```perl
y/@-~/M-ZA-Sn-za-m/,print for<>
```
#### Multiply

Multiply a list of comma-seperated integers
input:`2, 1, 2, 5, 149`
output:`2980`
```perl
y/,/*/,print eval for<>
```
```perl
print eval<>=~y/,/*/r
```
#### Multiply long version
```perl
y/,/*/,print eval,$/for<>
```
#### Drop first line
```perl
<>,print<>
```
#### Premutation

全排列
```perl
/(.).*\1/||print"@{[/./g]}
"for glob"{0,1,2,3,4,5}"x6
```
#### Duplicate characters
```perl
print$\=getc for%!
```
```perl
print/((.))/sgfor<>
```
```perl
#!perl -p
s/./$&$&/gs
```
#### Word frequency count
```perl
$/=$";y/a-zA-Z//cd,$_{+lc}++for<>;print"$_{$_} $_
"x/\D/for sort%_
```
#### Segs
```perl
<>=~/.*(.+?)(?{print"$1
"})^/
```
#### Stratum
```perl
<>=~'.*(.)(??{print$b="$1$b$1",$/})'
```
```perl
$_=<>;print$*=$&.$*.chop,$/while/.$/
```
#### Swab

Swap the adjacent bytes
```perl
$/=\1;print<>.$_ while<>
```
#### ASCII stars
```perl
print$"x abs,'**'x($a-abs).'*
'for-($a=<>-1)..$a
```
#### Apple lines
```perl
print$_=chop.$_,$/for(pple.'*'x25 .a)x(<>+1)
```
#### Number slide
```perl
print$_,--($a||=++$b)?$":$/for 1..<>
```
#### Uniq words
```perl
#!perl -lna
++$#$_||print for@F
```
#### Captical and small letter
```perl
print$a=$_=<>,(join$/,!s/./{\u$&,\l$&}/g,glob)!~$a,$',$`
```
