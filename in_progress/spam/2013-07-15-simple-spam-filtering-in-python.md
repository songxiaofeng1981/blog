title: Simple Spam Filtering in Python
date: 2013-07-15 09:48
categories: python tutorial

Recently, one of my tutoring clients asked to do a project in machine learning. 
Machine learning is a topic I would normally stay away from with a client; even 
if we arrive at a completed, working program, understanding *why* it's
working in the first place requires background knowledge. Luckily, we were able
to choose a project of appropriate size, difficulty, and prerequisite knowledge:

a naive Bayesian spam filter.

*Note: Those familiar with the math behind a naive Bayes classifier may complain that the probability calculations are performed incorrectly, and they would be right. My goal here is to present the general idea as a way to learn about Python without getting bogged down in the math.*

Our mission is to write a script capable of classifying an email as either "ham" or "spam".
So, how do we get a stupid Python script to detect spam email? 

It takes two steps.

#### The Learn Phase

The first step is called "supervised learning". In it, we feed the program email for 
which we already know the classification (i.e. "Here's a spam email I got
before. Here's one that's not spam ("ham"). Here's another spam email." etc.). The
program will take note of various properties of the email (which we'll discuss
in a bit). It records data for every email we feed to it, building up a sort
of knowledge-base of spam/ham email properties.

In the second phase, the program will use this refer back to this data
to determine the *most likely* classification of an unknown email. That is, it
uses what it has "learned" on a new, previously unseen email message.

To begin, we'll record a very simple set of properties: the frequency of words that appear in
the body of the email. Later, we'll make it a bit more robust by considering
properties like the subject line and sender. At a high level, we'll be using two identical dictionary-like structures (one for ham and one for spam). The spam data structure will contain every word we've seen in spam email and the number of times we've seen it (i.e. `{'vacation': 87, 'viagra': 19, ...}`). The same goes for the ham data structure.

## Classification

The classification of an unknown email is then relatively
straightforward to determine: 

* Keep a "spam score" and a "ham score", both starting at `0`
* For each word in the email, if it appears in the spam dictionary, add the
number of times that word has been seen to the spam score
* Do the same for the ham score
* Divide by number of messages seen in learning to get the "relative scores"
* Choose the largest score as the most likely classification

We calculate "relative" scores to even the playing field between our two data sets. 
For example, we may have seen the word "vacation" 8 times in ham
email and only 3 times in spam email. This suggests that an email with the word
"vaction" is likely to be ham. However, if we fed our program 200 ham emails and only
10 spam emails during the learn phase, those numbers take on quite a different meaning.
It becomes *much* more likely that an email containing "vacation" is spam, 
since 30% of the spam we saw had this word while only 4% of the ham email did. 

To account for this, we'll give each word a relative score
equal to the frequency in each class of email divided by the number of messages
seen in the learn phase for that class. In our example, those values would be `.3`
for spam and `.04` for ham.

## Finally, Some Code

Before we start writing any code, let's break down each task we know the program
will need to perform:

* During "learning"
    * Parse an email
    * Record salient properties of the parsed message
    * Add those properties to a persistent data store
* During "classification"
    * Parse an email
    * Load classification data from the data store
    * Compare the unkown email's properties to those of already-classified email
    * Decide the most likely classification

That seems like a reasonably comprehensive list. If our program can perform the
last step, it's essentially a spam filter.

Let's begin with the functionality required during the learn phase. Though the
program could be written as a series of function calls,
creating a class responsible for managing our interaction with email messages
"feels" right (especially if this were to become an API). Let's start with a 
generic skeleton for the script. 

Helpfully, I have a simple shell script to generate this skeleton for a new 
Python file. I recommend this practice if you avoid writing scripts the correct way
because writing the boiler-plate code "takes too long".

    #!py
    """Simple naive-bayes spam filter"""

    import sys

    def main():
        """Main entry point when called as script"""
        pass

    if __name__ == '__main__':
        sys.exit(main())

Of course, we'll likely need to alter this skeleton code, but it's a good start.
Next let's write an outline for the class. The way I typically write *any* large
portion of a program is to write the definition *and docstrings* of functions
and classes I need. Writing the docstring before you implement what it documents
is a gut-check to make sure you fully understand what that piece of code will do. If you
can't write the docstring (or use the word "and" fifty times), it's an early sign 
that you may need to rethink the structure of your program. Of course, we're not 
handcuffed to the choices we make now, but I find it to be a helpful way to 
organize my thoughts.

Here is the class definition skeleton:

    #!py
    class EmailClassifier(object):
        """Manage learning from and classification of email messages"""

        def __init__(self):
            pass

        def parse_message(self, message):
            """Return a list containing every word found in message"""
            pass

        def process_classified_message(self, message, classification):
            """Update our internal data store to reflect the information obtained
            from message with given classification.""" 
            pass

        def classify_message(self, message):
            """Return the most probable classification for the given message."""
            pass

        def save(self):
            """Persist the clasification data to disk."""
            pass

        def load(self):
            """Load previously saved clasification data to disk."""
            pass
