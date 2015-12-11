---
layout: post
title: Perl學習手記(三)
tags: Perl
categories: 编程
---

学习Perl过程中，昨天自行实现了XML文件的读取，将其内容放入到一散列中。
但至于其内容是否正确与否却不得而知，于是需要添加一些测试代码来验证。
以前写C/C++代码时，常用XPATH来搜索XML文件中的节点，这需要XML引擎来处理。

但现在读取XML文件采用的是“土方法”（自行实现的），所以这引擎也得自己来写喽了。
 XPATH引擎，我这其实就一函数，给定XPATH字串和存有XML文件内容的散列（当然是用MyXMLSimple方法读取进来的），
就可以搜索到你想找的XML节点，当然返回值也是个节点。

XPATH是很强大的，哥哥这里只是实现了一个基本的功能：查找节点。
我这里支持的XPATH路径只有两种：全局搜索的路径，即以双斜杠开头的；或是从根开始搜索的，即以间斜杠或直接以节点TAG开头的。
节点可以指定属性的值。支持的XPATH样板：

//TAG[@name="value" or @name="value"]/TAG[@name="value" and @name="value"]  #全局递归搜索

/TAG/TAG/TAG[@name=”value"]  #从根开始搜索

TAG/TAG[@name="value"]           #从根开始

其中的TAG表示节点的节点名字，name为节点的某一属性名字，value为某一属性的值，这些字串暂时不支持通配符查找。
程序中用来一些正则表达式，那个也刚学不太懂，所以一些正则表达式可能写得比较乱或是根本就是错误的，请原谅小弟的无知。
下面是Perl源程序。

~~~perl
####perl pragram###################################################
###############################my xml simple protected sub##############
#get node tag of a nodestring like <%TAG% ... >
#in param: the nodestring
#return value: the tag : %TAG%
sub GetNodeTag
{
  my ($NodeString) = @_;
  $NodeString =~ m/</s*//(/w+)/s*>/ or $NodeString =~ m/</s*(/w+).*?>/;
  return $1;
}

#get the attributes of a node that description by a string like 
#in param1: the nodestring
#in param2: a array to store the names and values of the attributes
#in the array: name1, value1, name2, value2, name3, value3,...
sub GetAtribList
{
  my ($nodestring, $atribarray) = @_;
  @$atribarray = $nodestring =~ m/(/w+)/s*=/s*/"(/w*?)/"/g;
}

#push the attributes of a node into a hash that reprent the node
#in param1: the nodestring
#in param2: the hash that will reprent the node
sub pushattributes
{
  my ($nodestring, $hash) = @_;
  my @tsarray = ();
  GetAtribList($nodestring, /@tsarray);
  if (@tsarray)
  {
 for (my $j = 0; $j < @tsarray;)
 {
   my $key = @tsarray[$j];
   my $value = @tsarray[$j+1];
   $$hash{$key} = $value;
   $j += 2;
 }
  }
}

#push a couple of key and value to a hash
#in param1: the key
#in param2: the value
#in param3: the hash
sub pushhash
{
  my ($key, $value, $hash) = @_;
  if ($hash =~ /^HASH/)
  {
 if (exists
hash{$key})  {    $ekvalue =
hash{$key};
   if ($ekvalue =~ /^ARRAY/)
   {
     my @array=@{$ekvalue};
     my $index = @array;
  
hash{$key}[$index] = $value;    }    else    {       my @tparray = ();       push (@tparray, $ekvalue);     push (@tparray, $value);
hash{$key}=[@tparray];
   }
 }
 else
 {
   $$hash{$key} = $value;
 }
  }
}

#generate a xml dom hash by a nodearray that contained the nodestring of the xml file
#in param1: the nodearray that contained all the nodestring in the xml file <...>
#in param2: the begin pos to generate the xml dom hash in the nodearray
#in param3: the end pos to generate the xml dom hash in the nodearray
#in param4: the hash
sub GenerateXMLDom
{
   my ($nodearray, $begin, $length, $root)=@_;
   for (my $i=$begin; $i<$begin + $length; )
   {
   $temp = $$nodearray[$i];
   my ($nodetag) = GetNodeTag($temp);
   if ($temp =~ m////s*>/) #single node
   {
   my %tshash1 = ();
   pushattributes($temp, /%tshash1);
   pushhash($nodetag, /%tshash1, $root);
   $i++;
   next;
   }
   elsif ($temp =~ m/</s*///) #end of complex node
   {
   $i++;
   next;
   }
   else #complex node
   {
   my %tshash2 = ();
   pushattributes($temp, /%tshash2);

   my $headnum = 1;
   my $endnum = 0;
   my $start = $i;
   while ($endnum != $headnum && $i < $begin + $length-1)
   {
   $i++;
   my $nextnode = $$nodearray[$i];
   if ($nextnode =~ m////s*>/) #single node
   {
      $headnum++;
      $endnum++;
   }
   elsif ($nextnode =~ m/</s*///) #end of complex node
   {
      $endnum++;
   }
   else
   {
      $headnum++;
   }
   }

   $i++;
   my $len = $i - $start;
   GenerateXMLDom(/@nodearray, $start+1, $len-2, /%tshash2);
   pushhash($nodetag, /%tshash2, $root);
   next;
   }
   }
}
###############################end of my xml simple protected sub##############

################################my xml simple public sub#####################
#my xml simple, to read a xml file into a hash
#in param: the xml file to read
#return value: the hash that contained the xml dom content
sub MyXMLSimple
{
  my ($xmlfile) = @_;
  open (INFILE, "$xmlfile") or die "can't open $xmlfile for reading $!";
  while ($line=)
  {
  chomp $line;
  $filecontent.=$line;
  }
  close INFILE;

  #delete xml comment
  $filecontent =~ s///g;

  #cut file context by <...>
  @nodearray = $filecontent =~ m/<.*?>/g;
  $length = @nodearray;

  %rootnode = ();  #the root node of the xml file

  #construct the xml dom struct
  GenerateXMLDom(/@nodearray, 0, $length, /%rootnode);

  return %rootnode;
}

########################end of my xml simple#############################

###########################xpath protected sub###########################

#use to search the node by attributes, the node tag has already matched
#in param1:search xpath suchas TAGNAME[@attributename="attributevalue" and @attributename="attributevalue" or @attributename="attributevalue" ...]
#in param2:a hash with the key of TAGNAME
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub SearchNodeByAttribute
{
 my ($TagString, $hash, $result) = @_;

 #get the attribute name list
 #the element in the array is such as : name, =|!=, value, name, =|!=, value,...
 my @attributes = $TagString =~ m//@(/w+)/s*(!?=)/s*[/"|'](.*?)[/"|']/g;
 
 #get the logical relations between attributes if existed more then one attributes
 my $index = index($TagString, "[");
 my $endex = index($TagString, "]");
 my $attrstr;
 if ($index != -1 && $endex != -1)
 {
   my $len = $endex - $index - 1;
   $attrstr = substr($TagString, $index+1, $len);
 }

 #the logical relations list
 #a sample : and,or,and,or
 my (@atrirelation) = $attrstr =~ m//@/w+/s*!?=/s*[/"|'].*?[/"|']/s*(and|or)/s*/g;

 #if do not give the attributes, then found the node
 if (@attributes <= 0)
 {
  push (@$result, $hash);
  return 1;
 }

 #compare the existed attributes with the given attributes and store the result to a array
 my @tparray = ();
 for (my $i=0; $i < @attributes;)
 {
  my $attname = $attributes[$i];
  my $oper = $attributes[$i+1];
  my $attvalue = $attributes[$i+2];
  my $exvalue = $$hash{$attname};
  my $b = 0;
  if ($oper =~ //s*=/s*/)
  {
   if ($exvalue =~ m/$attvalue/)
   {
      $b = 1;
   }
  }
  else
  {
   if ($exvalue =~ m/$attvalue/)
   {
    $b = 0;
   }
   else
   {
    $b = 1;
   }
  }

  push(@tparray, $b);
  $i += 3;
  }

  #use the logical relations beweent attributes to combine the compare results
  my $tpret = $tparray[0];
  for (my $j = 0; $j < @atrirelation; $j++)
  {
  if ($atrirelation[$j] =~ //s*and/s*/)
  {
   $tpret = $tpret && $tparray[$i+1];
  }
  else
  {
   $tpret = $tpret || $tparray[$i+1];
  }
  }

  #if the combined result is 1, then found it
  if ($tpret)
  {
  push(@$result, $hash);
  }

  return $tpret;
}

#use to search a node by the give tag and resultes from its parent node hash
#in param1:search xpath suchas TAGNAME[@attributename="attributevalue" and @attributename="attributevalue" or @attributename="attributevalue" ...]
#in param2:a hash that is the parent node
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub SearchNode
{
 my ($TagString, $hash, $result) = @_;
 if (length($TagString) == 0)
 {
  return 0;
 }

 #get the node tag
 $TagString =~ m/(/w+)/;
 my $Tag = $1;
 if ($hash =~ /^HASH/)  # must be a hash
 {
   if (exists
hash{$Tag}) # in the hash existed a key value match the node tag, means there is existed a child node in the hash that has the tag of $Tag    {   my $value =
hash{$Tag}; #get the existed child node hash or array
  if ($value =~ /^ARRAY/)  #there is existed more then on hash node that has the node tag of $Tag
  {
   my @values = @{$value};
   my $ret = 0;
   for (my $k=0; $k < @values; $k++)
   {
     #match each node in the array to find the node
     my ($b) = SearchNodeByAttribute($TagString, $values[$i], $result);
     if ($b)
     {
    $ret = 1;
     }
   }

   return $ret;
  }
  else
  {
   #compare the given attributes to find the node
   return SearchNodeByAttribute($TagString, $value, $result);
  }
   }
   else
   {
    return 0;
   }
 }
 else
 {
    return 0;
 }
}

#the $hash as the root hash, and the $xpath was givened from the root, then to search the node by given xpath
#in param2:a hash that is the parent node
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub SearchFromRoot
{
 my ($xpath, $hash, $result) = @_;
 if ($xpath =~ m/^///)
 {
  $xpath = substr($xpath, 1);
 }

 if (length($xpath) == 0)
 {
  return 0;
 }

 #get the first node tag and attributes such as TAGNAME[@attributename="attributevalue" and ... or ...]
 my ($tag) = $xpath =~ m/(/s*/w+/s*(/[(/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*(and|or)/s*)*/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*/])?/s*)/;
 #remove the first node tag then for the second recure search
 $xpath =~ s//s*/w+/s*(/[(/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*(and|or)/s*)*/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*/])?/s*//;
 
 my @nodelist = ();
 my $ret = 0;
 #first find the hash node that match the first node tag and attributes that givened
 $ret = SearchNode($tag, $hash, /@nodelist);
 
 #if do not found, then return 0
 if ($ret == 0)
 {
     return 0;
 }
 else
 {
  #found the first node tag and attributes node
  if (length($xpath) == 0)
  {
      #recure search end, then store the results
      for (my $i=0; $i < @nodelist; $i++)
      {
       push(@$result, $nodelist[$i]);
      }
      return 1;
  }
  else
  {
      my $ret = 0;
      for (my $i=0; $i < @nodelist; $i++)
      {
       #then search the second node tag and attributes
       my ($b) = GetNodeByPath($xpath, $nodelist[$i], $result);
       if ($b)
       {
      $ret = 1;
       }
      }

      return $ret;
  }
 }
}

#used in recure search xpath sub.
#call the recure search sub in each element in the array
#in param1:a xpath
#in param2:a hash that is the root node
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub RecureSearchArrayNode
{
 my ($xpath, $value, $result) = @_;

 if ($value =~ /^HASH/)
 {
  return RecureSearchNode($xpath, $value, $result);
 }
 elsif ($value =~ /^ARRAY/)
 {
   my @tparray = @{$value};
   my $ret = 0;
   for (my $k=0; $k < @tparray; $k++)
   {
     my ($b) = RecureSearchArrayNode($xpath, $tparray[$k], $result);
     if ($b)
     {
    $ret = 1;
     }
   }
   return $ret;
 }
 else
 {
  return 0;
 }
}

#used for the recure search when the xpath was givened like : //TAG/TAG[...]/...
#thought that the xpath is givened from the root node hash, and each hash node in the xml hash dom can be the root node, then do the recure search
#in param1:a xpath
#in param2:a hash that is the root node
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub RecureSearchNode
{
 my ($xpath, $hash, $result) = @_;

 my $ret = 0;
 #first search the node from the given hash node
 $ret = SearchFromRoot($xpath, $hash, $result);

 #then recure the each element in the current hash node
 while (($key, $value) = each(%$hash))
 {
  if ($value =~ /^HASH/)
  {
   my ($b) = RecureSearchNode($xpath, $value, $result);
   if ($b)
   {
     $ret = 1;
   }
  }
  elsif ($value =~ /^ARRAY/)
  {
   my @tparray = @{$value};
   my $tpret = 0;
   for (my $k=0; $k < @tparray; $k++)
   {
     if ($tparray[$k] =~ /^HASH/)
     {
      my ($b) = RecureSearchNode($xpath, $tparray[$k], $result);
      if ($b)
      {
       $tpret = 1;
      }
     }
     elsif ($tparray[$k] =~ /^ARRAY/)
     {
      my ($b) = RecureSearchArrayNode($xpath, $tparray[$k], $result);
      if ($b)
      {
     $tpret = 1;
      }
     }
     else
     {
      next;
     }
   }

   if ($tpret)
   {
      $ret = 1;
   }
  }
  else
  {
   next;
  }
 }

 return $ret;
}

#used search hash node by a string that flow xpath syntax, the xpath is a single path
#in param1:a xpath
#in param2:a hash that is the root node of a xml dom struct
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub SearchPathNode
{
 my ($xpath, $hash, $result) = @_;

 if (length($xpath) != 0)
 {
  if ($xpath =~ m/^/////) #recure search
  {
   $xpath = substr($xpath, 2);
   return RecureSearchNode($xpath, $hash, $result);
  }
  else  #search from root
  {
   return SearchFromRoot($xpath, $hash, $result);
  }
 }
 else
 {
  return 0;
 }
}
###########################end of xpath protected sub###########################

########################################public xpath sub########################
#used search hash node by a string that flow xpath syntax, the xpath is a complex path that combine by more then one xpath
#in param1:a xpath
#in param2:a hash that is the root node of a xml dom struct
#in param3:a array to store the find node which was reprented by a hash
#return value: 1 means has found the node match the attributes or 0 means not found
sub GetNodeByPath
{
 my ($xpathlist, $hash, $result) = @_;

 if ($xpathlist =~ m/(((////|//)?/s*/w+/s*(/[(/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*(and|or)/s*)*/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*/])?/s*///s*)*(////|//)?/s*/w+/s*(/[(/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*(and|or)/s*)*/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*/])?/s*/|/s*)*((////|//)?/s*/w+/s*(/[(/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*(and|or)/s*)*/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*/])?/s*///s*)*(////|//)?/s*/w+/s*(/[(/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*(and|or)/s*)*/s*/@/w+/s*=/s*[/"|']/w+[/"|']/s*/])?/s*/)
 {
  my @xpatharray = split(//|/,$xpathlist);
  $ret = 0;
  foreach my $xpath (@xpatharray)
  {
   if (length($xpath) == 0)
   {
    next;
   }

   my ($b) = SearchPathNode($xpath, $hash, $result);
   if ($b)
   {
     $ret = 1;
   }
  }

  return $ret;
 }
 else
 {
  print "the xpath $xpathlist does not match the syntax of xpath!/n";
  return;
 }
}

###########################end of my xpath#########################################

#test code
(%xmldoc) = MyXMLSimple("test.xml");

#$xpath = "////CTRL[/@name=/"cs_edit/" or /@value=/"ce_edit/"]//C//EN//EVT[/@name='EVT_CON_FE_OK' or /@value=/"0x1013/"]";
$xpath = "////EN//EVT";
@nodelist = ();
$ret = GetNodeByPath($xpath, /%xmldoc, /@nodelist);

if ($ret)
{
 foreach my $value (@nodelist)
 {
   $a = $$value{"name"};
   print "$a /n";
 }
}
#end test
~~~