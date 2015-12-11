---
layout: post
title: Perl學習手記(一)
tags: Perl
categories: 编程
---

散列的遍历。


~~~perl

%xmldoc=(

"CTRL" =>

{

"name" => "cs_edit",

"type" => "CS_EDIT",

"C" =>

{

"name" => "edit",

"type" => "0",

 "clsid" => "0x010551e6",

"idx" => "0",

"EN" =>

{

"EVT" =>[ { "name" => "EVT_FE_OK", "type" => "0", "value" => "0x1013", },

 { "name" => "EVT_FE_BACK", "type" => "0", "value" => "0x1013", },],

}

},

 },

 );

 

printharsh(/%xmldoc);

 

 sub printarray

 {

my ($array) = @_;

 foreach ($array)

{

if ($_ =~ /^HASH/)

 {

printharsh($_);

}

elsif ($_ =~ /^ARRAY/)

{

printarray($_);

}

else

{

print "$_ /n";

}

}

}

 

sub printharsh

{

my ($harsh) = @_;

if ($harsh =~ /^HASH/)

 {

while (($key, $value) = each(%$harsh))

{

if ($value =~ /^HASH/)

{

printharsh($value);

 }

elsif ($value =~ /^ARRAY/)

 {

my @array = @{$value};

foreach (@array)

{

 if ($_ =~ /^HASH/)

{

printharsh($_)

}

elsif ($_ =~ /^ARRAY/)

 {

 printarray(@$value);

 }

else

 {

print "$_ /n";

}

 }

}

else

{

 print "$key => $value /n";

}

 }

}

}
~~~