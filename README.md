# NAME

IO::Storm - IO::Storm allows you to write Bolts and Spouts for Storm in Perl.

# VERSION

version 0.17

# SYNOPSIS

    package SplitSentenceBolt;

    use Moo;
    use namespace::clean;

    extends 'Storm::Bolt';

    sub process {
        my ($self, $tuple) = @_;

        my @words = split(' ', $tuple->values->[0]);
        foreach my $word (@words) {

            $self->emit([ $word ]);
        }

    }

    SplitSentenceBolt->new->run;

# DESCRIPTION

IO::Storm allows you to leverage Storm's multilang support to write Bolts and
Spouts in Perl.  As of version 0.02, the API is designed to very closely mirror
that of the [Streamparse Python library](http://streamparse.readthedocs.org/en/latest/api.html).  The exception being that we don't currently support
the `BatchingBolt` class or the `emit_many` methods.

# Bolts

To create a Bolt, you want to extend the `Storm::Bolt` class.

    package SplitSentenceBolt;

    use Moo;
    use namespace::clean;

    extends 'Storm::Bolt';

## Processing tuples

To have your Bolt start processing tuples, you want to override the `process`
method, which takes a `IO::Storm::Tuple` as its only argument.  This method
should do any processing you want to perform on the tuple and then `emit` its
output.

    sub process {
        my ($self, $tuple) = @_;

        my @words = split(' ', $tuple->values->[0]);
        foreach my $word (@words) {

            $self->emit([ $word ]);
        }

    }

To actually start your Bolt, call the `run` method, which will initialize the
bolt and start the event loop.

    SplitSentenceBolt->new->run;

## Automatic reliability

By default, the Bolt will automatically handle acks, anchoring, and
failures.  If you would like to customize the behavior of any of these things,
you will need to set the `auto_anchor`, `auto_anchor`, or `auto_fail`
attributes to 0.  For more information about Storm's guaranteed message
processing, please [see their documentation](https://storm.incubator.apache.org/documentation/Guaranteeing-message-processing.html#what-is-storms-reliability-api).

# Spouts

To create a Spout, you want to extend the `Storm::Spout` class.

    package SentenceSpout;

    use Moo;
    use namespace::clean;

    extends 'Storm::Spout';

## Emitting tuples

To actually emit anything on your Spout, you have to implement the
`next_tuple` method.

    my $sentences = ["a little brown dog",
                     "the man petted the dog",
                     "four score and seven years ago",
                     "an apple a day keeps the doctor away",];
    my $num_sentences = scalar(@$sentences);

    sub next_tuple {
        my ($self) = @_;

        $self->emit( [ $sentences->[ rand($num_sentences) ] ] );

    }

To actually start your Spout, call the `run` method, which will initialize the
Spout and start the event loop.

    SentenceSpout->new->run;

# Methods supported by Spouts and Bolts

## Custom initialization

If you need to have some custom action happen when your component is being
initialized, just override `initialize` method, which receives the Storm
configuration for the component and information about its place in the topology
as its arguments.

    sub initialize {
        my ( $self, $storm_conf, $context ) = @_;
    }

## Logging

Use the `log` method to send messages back to the Storm ShellBolt parent
process which will be added to the general Storm log.

    sub process {
        my ($self, $tuple) = @_;
        ...
        $self->log("Working on $tuple");
        ...
    }

# AUTHORS

- Dan Blanchard <dblanchard@ets.org>
- Cory G Watson <gphat@cpan.org>

# COPYRIGHT AND LICENSE

This software is copyright (c) 2014 by Educational Testing Service.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
