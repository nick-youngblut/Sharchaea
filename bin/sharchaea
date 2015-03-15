#!/usr/bin/env perl

use Term::Animation 2.0;
use Curses;
use strict;
use warnings; 
use Data::Dumper;
use List::Util qw/min max/;
use Math::Trig;
use POSIX;


#-- main --#
# flagella1
my $flagella_len = 80;
my $nrow = 5;
my $n_strings = 10;
my $lambda = 0.55;

$n_strings--;
my @amp = map{ ($_ - ($n_strings / 2)) / ($n_strings * 1.5) } 0..$n_strings; 
push @amp, reverse(@amp[1..$#amp-1]);

## making arrays of flagella at different amplitudes
my @flagella = map{ make_flagella_str($_, $lambda, $flagella_len, $nrow) } @amp;

## padding flagella frames lacking enough rows
pad_frames(\@flagella);

## making strings from frame-arrays
@flagella = map{ join("\n", @$_) } @flagella;


# flagella2
$flagella_len = 86;
$lambda = 0.56;

## making arrays of flagella at different amplitudes
my @flagella2 = map{ make_flagella_str($_, $lambda, $flagella_len, $nrow) } @amp;

## padding flagella frames lacking enough rows
pad_frames(\@flagella2);

## making strings from frame-arrays
@flagella2 = map{ join("\n", @$_) } @flagella2;
@flagella2 = (@flagella2[2..$#flagella2], @flagella2[0..1]);

#@flagella2 = (@flagella2[11..$#flagella2], @flagella2[0..10]);


# flagella3
$flagella_len = 74;
$lambda = 0.52;

## making arrays of flagella at different amplitudes
my @flagella3 = map{ make_flagella_str($_, $lambda, $flagella_len, $nrow) } @amp;

## padding flagella frames lacking enough rows
pad_frames(\@flagella3);

## making strings from frame-arrays
@flagella3 = map{ join("\n", @$_) } @flagella3;
@flagella3 = (@flagella3[1..$#flagella3], @flagella3[0..0]);
#@flagella3 = (@flagella3[6..$#flagella3], @flagella3[0..5]);




# sharchaea
my $sharchaea_body = make_sharchaea_body();


# initialize anim object
my $anim = Term::Animation->new();
$anim->color(1);

# screen size
my ($width, $height, $assumed_size) = $anim->screen_size();

# set the delay for getch
halfdelay( 1 );
 

## initialize entities 
my $entity_s = Term::Animation::Entity->
  new(
      name          => 'sharchaea_body',
      shape         => $sharchaea_body, 
      position      => [$width, 5, 1],
      callback_args => [-1, 0, 0, 1],
      #callback      => \&updown,
      wrap          => 0,              # turn screen wrap on
      default_color => 'red'		  
     );
$anim->add_entity($entity_s);

my $entity_f1 = Term::Animation::Entity->
  new(
      name          => 'flagella1',
      shape         => \@flagella, 
      position      => [$width+79, 14, 10],
      callback_args => [-1, 0, 0, 1],
      wrap          => 0,              # turn screen wrap on
      default_color => 'red',
      transparent   => " "
     );
$anim->add_entity($entity_f1);

my $entity_f2 = Term::Animation::Entity->
  new(
      name          => 'flagella2',
      shape         => \@flagella2, 
      position      => [$width+79, 14, 10],
      callback_args => [-1, 0, 0, 1],
      wrap          => 0,              # turn screen wrap on
      default_color => 'red',
      transparent   => " "
     );
$anim->add_entity($entity_f2);

my $entity_f3 = Term::Animation::Entity->
  new(
      name          => 'flagella3',
      shape         => \@flagella3, 
      position      => [$width+79, 16, 10],
      callback_args => [-1, 0, 0, 1],
      wrap          => 0,              # turn screen wrap on
      default_color => 'red',
      transparent   => " "
     );
$anim->add_entity($entity_f3);


 
# animation loop
while(1) {
  # run and display a single animation frame
  $anim->animate();
 
  # use getch to control the frame rate, and get input at the
  # same time. (not a good idea if you are expecting much input)
  my $input = getch();
  if($input eq 'q') { last; }
}
 
# cleanly end the animation, to avoid hosing up the user's terminal
$anim->end();



#-- subrountines --#
## callback args
sub updown {
    my ($entity, $anim) = @_;
 
    # get the current position of the entity
    my ($x, $y, $z) = $entity->position();
 
    my @moves = (0,0,0,0,1,
		 0,0,0,0,0,0,0,0,-1,
		 0,0,0,0,0,0,0,0,1,
		 0,0,0,0,0,0,0,0,-1,
		 0,0,0,0);

    # last callback values
    my $cargs = $entity->callback_args();
    my $cnt = $cargs->[0];

    my $move = $moves[$cnt % scalar(@moves)];
    #my $move = 1;

    # new callback
    $cnt += 1;
    $entity->callback_args([$cnt]);
    
			      
    # return the absolute x,y,z coordinates to move to
    return ($x-1, $y+$move, $z);
  }

sub damped_sine_wave{
  my $t = shift || 0;
  my $amp = shift || 1;
  my $lambda = shift || 1;

  return $amp * exp($lambda * $t) * cos(2 * pi * $t)
}

# making a matrix of characters
sub set_char{
  my $y = shift;

  $y = $y - int($y);   # just decimal

  if ($y <= 0.33){
    return '.';
  }
  elsif ($y > 0.33 && $y <= 0.66){
    return '-';
  }
  elsif ($y > 0.66){
    return '`';
  }
  else{
    die "ERROR: value: $y\n";
  }
}

sub mtx2str{
  my $mtx_ref = shift;

  my @str;
  foreach my $row (@$mtx_ref){
    @$row = map{defined $_ ? $_ : ' '} @$row;
    push @str, join("", @$row);    
  }
  return \@str; #join("\n", @str);
}

sub rescale{
  my $vals_ref = shift or die $!;
  # new specified range
  my $c = shift;
  my $d = shift;
  # current range
  my $a = min @$vals_ref;
  my $b = max @$vals_ref;

  return map{ ($_ - $a) * ($d - $c) / ($b - $a) + $c } @$vals_ref;

}

sub make_flagella_str{
  my $amplitude = shift;
  my $lambda = shift || 1;
  my $flagella_len = shift || 80;
  my $nrow = shift || 5;
  
  # making x,y for damp sine wave
  my @x = map{ $_ * $nrow / $flagella_len } 0..$flagella_len-1;
  my @y = map{ damped_sine_wave($_, $amplitude, $lambda) } @x;

  # rescale y values to 0..$rnow-1
  my $min_y = min(@y);
  if ($min_y < 0){
    @y = map{ $_ + abs($min_y) } @y;
  }
  else{
    @y = map{ $_ - $min_y } @y;
  }

  # setting 1st few y values as same integer
  @y[0..2] = (int $y[0]) x 3;

  # making matrix from x-y values; matrix of ascii 
  my $max_y = max @y;
  my @mtx;
  for my $x (0..$flagella_len-1){
    my $y_val = $y[$x];
    my $y_int = $max_y - int($y_val);  # flipping rows in table
    $mtx[$y_int][$x] = set_char($y_val);
  }

  return mtx2str(\@mtx);
}

sub pad_frames{
  my $flagella = shift or die $!;

  my $max_nrow = max(map{ scalar @$_ } @$flagella);

  foreach my $frame (@$flagella){
    my $n_pads = $max_nrow - scalar(@$frame);
    my $n_pads_bottom = ceil($n_pads / 2);
    my $n_pads_top = int($n_pads / 2);
    $n_pads_bottom++ if $n_pads_bottom + $n_pads_top < $n_pads;
    my $ncol = length $frame->[0];

    # bottom padding
    for my $i (0..$n_pads_bottom-1){
      unshift(@$frame, " " x $ncol);
    }
    # top padding
    for my $i (0..$n_pads_top-1){
      push(@$frame, " " x $ncol);
    }    
  }
}

sub print_frames{
  my $frames = shift or die $!;
  
  foreach my $frame (@$frames){
    foreach my $line (@$frame){
      print $line, "\n";
    }
    print "\n";
  }
}

sub make_sharchaea_body{
  my $sharchaea1 = join("\n",
			q{                                              ...............                  },
			q{                                     .......``      .....`                     },
			q{                                ....`            ...`                          },
			q{                             ..`               .`                              },
			q{                          .-`                 :`                               },
			q{                         :                    :                                },
			q{                      ..`.....................:                                },
			q{          ...........`                        ```..............                },
			q{     ....`                                                     `....           },
			q{   .`                                                               `...       },
			q{ .`                                                                     `-.    },
			q{:           @@         \    \    \                                         `.  },
			q{:          @@@          \    \    \                                          `.},
			q{:                        )    )    )                                          :},
			q{:                  ..   /    /    /                                           :},
			q{:        ........:` :  /    /    /                                            :},
			q{`.......`\/ \/ \/  :                                                          :},
			q{  \/ \/          .`                                                         .` },
			q{       /\  /\ .^`                                                      ....``   },
			q{     `:``````^                                                 ......`         },
			q{       `....................................................```                },
			q{                                                                               }
		       );


  my $sharchaea2 = join("\n",
			q{                                              ...............                  },
			q{                                     .......``      .....`                     },
			q{                                ....`            ...`                          },
			q{                             ..`               .`                              },
			q{                          .-`                 :`                               },
			q{                         :                    :                                },
			q{                      ..`.....................:                                },
			q{          ...........`                        ```..............                },
			q{     ....`                                                     `....           },
			q{   .`                                                               `...       },
			q{ .`                                                                     `-.    },
			q{:           @@         \    \    \                                         `.  },
			q{:          @@@          \    \    \                                          `.},
			q{:                        )    )    )                                          :},
			q{:                  ..   /    /    /                                           :},
			q{:        ........:` :  /    /    /                                            :},
			q{`.......`\/ \/ \/  :                                                          :},
			q{  \/ \/       __ ..`                                                        .` },
			q{          __  \.`                                                     ....``   },
			q{         .\.``                                                 ......`         },
			q{        `...................................................```                },
			q{                                                                               }
		       );


  my $sharchaea3 = join("\n",
			q{                                              ...............                  },
			q{                                     .......``      .....`                     },
			q{                                ....`            ...`                          },
			q{                             ..`               .`                              },
			q{                          .-`                 :`                               },
			q{                         :                    :                                },
			q{                      ..`.....................:                                },
			q{          ...........`                        ```..............                },
			q{     ....`                                                     `....           },
			q{   .`                                                               `...       },
			q{ .`                                                                     `-.    },
			q{:           @@         \    \    \                                         `.  },
			q{:          @@@          \    \    \                                          `.},
			q{:                        )    )    )                                          :},
			q{:                  ..   /    /    /                                           :},
			q{:        ........:` :  /    /    /                                            :},
			q{`.......`\/ \/ \/  :                                                          :},
			q{  \/ \/        __ .`                                                        .` },
			q{               \.`                                                    ....``   },
			q{           __ .`                                               ......`         },
			q{           \.`    ..........................................```                },
			q{           `----``                                                             }
		       );


  my $n_frames = 5;

  return [($sharchaea1) x $n_frames, 
	  ($sharchaea3) x $n_frames];
	  #($sharchaea3) x $n_frames, ($sharchaea2) x $n_frames];
}
 

