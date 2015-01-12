---
layout: post
title: Quick graphs with Perl / GD::Graph
date: '2009-07-08T19:05:00-04:00'
tags:
- Code
- perl
- gd
- graph
- mysql
tumblr_url: http://www.hisham.cc/post/30203540969/quick-graphs-with-perl-gd-graph
---
So I had to quickly whip up some graphs today at work based on values coming in from one of our database tables. Nothing better then Perl and GD (GD::Graph) for a quick and effective solution.

use strict;
use DBI;
use GD::Graph::bars;
use GD::Graph::Data;

my $db_host = ''XXXXXX'';
my $db_name = ''XXXXXX'';   
my $db_user = ''XXXXXX'';
my $db_pass = ''XXXXXX'';
my $query   = "select month, undelivered from XXXXX where XXXXX";

# create labels and values for x-axis

my ($labels, $values) = get_data_from_sql($db_host, $db_name, $db_user, $db_pass, $query);

# graph and save the output

graph($labels, $values, "Month", "Undelivered", "Undelivered Messages by Month", "undelivered.png");


sub get_data_from_sql($$$$$)
{
  my ($db_host, $db_name, $db_user, $db_pass, $query) = @_;

  my $dbh = DBI->connect("dbi:mysql:$db_name:$db_host", $db_user, $db_pass)
    or die "Couldn''t connect to database: " . DBI->errstr;

  my $sth = $dbh->prepare($query)
    or die "Couldn''t prepare statement: " . $dbh->errstr;

  $sth->execute();
  my @row= undef;

  my @labels = ();
  my @values = ();

  while (@row = $sth->fetchrow_array())
  {
    push @labels, $row[0];
    push @values, $row[1];
  }

  $dbh->disconnect;

  return (\@labels, \@values);
}

sub graph($$$$$$)
{
  my ($labels, $values, $x_label, $y_label, $title, $out_file) = @_;

  my $data = GD::Graph::Data->new([$labels, $values,])
    or die GD::Graph::Data->error;

  my $my_graph = GD::Graph::bars->new();

  $my_graph->set(
  x_label => $x_label,
  y_label => $y_label,
  title   => $title,
  bar_spacing => 8,
  shadow_depth => 4,
  shadowclr => ''dred'',
  transparent => 0,
  )
  or warn $my_graph->error;

  $my_graph->plot($data) or die $my_graph->error;
  my $gd = $my_graph->plot($data) or die $my_graph->error;
  open(IMG, ">$out_file") or die $!;
  binmode IMG;
  print IMG $gd->png;
}


If you do not want your data coming from an SQL statement simply fill in $labels and $values with anything you want, like:

$labels = [''Monday'', ''Tuesday'', ''Wednesday''];
$data = [45, 66, 89];


Pretty straight forward and handy. the graph looks like this.
