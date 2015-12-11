---
layout: post
title: Perl學習手記(二)
tags: Perl
categories: 编程
---

MyXMLSimple-讀取xml文件到一散列中。


~~~perl
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
			$$hash{$key} = $value; $j += 2;
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
		if (exists $$hash{$key})
		{
			$ekvalue = $$hash{$key};
			if ($ekvalue =~ /^ARRAY/)
			{
				my @array=@{$ekvalue};
				my $index = @array;
				$$hash{$key}[$index] = $value;
			}
			else
			{
				my @tparray = ();
				push (@tparray, $ekvalue);
				push (@tparray, $value);
				$$hash{$key}=[@tparray];
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
			pushhash($nodetag, /%tshash1, $root); $i++; next;
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
			pushhash($nodetag, /%tshash2, $root); next;
		}
	}
}

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
	#delete xml comment $filecontent =~ s///g;
	#cut file context by <...>
	@nodearray = $filecontent =~ m/<.*?>/g;
	$length = @nodearray;
	%rootnode = (); #the root node of the xml file
	#construct the xml dom struct
	GenerateXMLDom(/@nodearray, 0, $length, /%rootnode);
	return %rootnode;
}

#test code
(%xmldoc) = MyXMLSimple("test.xml");
~~~