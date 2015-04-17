#CMU Pronouncing Dictionary

There is no way to know how an English word should be pronounced given only its
spelling. However, in the course of creative language generation, we're often
in need of information about how a particular word would be pronounced, if read
aloud; we might want to use this information for a number of creative and poetic
purposes, such as automated rhyming and assonance, or to generate text that
conforms to a particular meter.

Fortunately, the powers that be (i.e., DARPA and "member companies of the
Carnegie Mellon Sphinx Speech Consortium) have gifted us with the CMU
Pronouncing Dictionary. The CMU Pronouncing Dictionary is a plain-text,
computer-readable database that maps English words to their pronunciations.
It's an incredible boon to poets and researchers alike.

[Visit the home page of the
dictionary](http://www.speech.cs.cmu.edu/cgi-bin/cmudict), or [download the
dictionary
itself](http://svn.code.sf.net/p/cmusphinx/code/trunk/cmudict/cmudict-0.7b).

##Modules and libraries

I've made a Python module for parsing and using the CMU pronouncing dictionary.
You can download it
[here](https://github.com/aparrish/rwet-examples/tree/master/pronouncing) (grab
the `cmudict.py` file and the `cmudict-0.7b` file and put them in the same
directory as the rest of your Python. I'm making a cleaner library soon,
promise!)

For the Javascript folks out there, I made this module that you can install
with `npm` or use in the browser: [pronouncing-js](https://github.com/aparrish/pronouncingjs).

##File format

The dictionary is a plain-text file. Each line of the file has a word and its
pronunciation, separated by two spaces. Here's a sample line:

	CARNEGIE  K AA1 R N EH0 G IY0

This is the entry for the word `CARNEGIE`, which has a pronunciation of 
`K AA1 R N EH0 G IY0` (for more on what the characters in the pronunciation
mean, see below).

Occasionally, one word will have several pronunciations associated with it.
In those cases, the dictionary has an entry for each possible pronunciation,
with a parenthesized number that increments for each subsequent entry:

	ADULT  AH0 D AH1 L T
	ADULT(1)  AE1 D AH0 L T

Additionally, there are some lines (at the beginning of the file) that begin
with a semicolon (`;`). These are comments and should be ignored.

Parsing the dictionary is as easy as this:

* Read each line, discarding if the line is a comment;
* Split the line at the point where the double space occurs;
* Strip off the parenthesized portion rand add an item to a map/dictionary data structure mapping the string to a list of pronunciations for that string.

A dictionary/map data structure works great for storing this information, but
I also like to use a list of tuples, with one tuple for each word/pronunciation
pair. (Lookups are slower with this data structure, but it has the advantage
of retaining the order of the words.)

##Phonetic alphabet

So what are those weird symbols that make up the pronunciations? They're
letters is what's called the "[ARPAbet](http://en.wikipedia.org/wiki/Arpabet),"
which is a way of representing (in ASCII plain text) all of the sounds that
occur in English. [The CMU Dictionary home page has a
table](http://www.speech.cs.cmu.edu/cgi-bin/cmudict) that describes what each
syllable represents.

> Linguistics nerds might say at this point, "Hey you bigshots, there's already
> a standard alphabet for phonetic transcription! It's called IPA and it's
> rad!" This is all true: the IPA does exist, and it is, indeed, rad. I need to
> do some research to be sure, but I imagine that the ARPAbet was designed and
> chosen for this task because it's easier to type and parse than IPA (which,
> in the pre-Unicode days from which the CMU dictionary originates, would
> require a custom character encoding in addition to custom keyboard mappings).
> Having a specialized phonetic alphabet for English is also helpful, because
> it abstracts away some of the trickier aspects of English phonology (such as
> regional variation, post-vowel glides, aspirated consonants, etc.)

The pronunciation information for each word also includes information the
where the stress falls in each word. The stress is indicated by the number
next to each vowel. `1` is primary stress; `2` is secondary stress; and `0` is
unstressed.

##Simple parsing

* [Code example](https://github.com/aparrish/rwet-examples/blob/master/pronouncing/words_beginning_with_sss.py) in Python
* [Code example](https://gist.github.com/aparrish/78c23721aeb0e8784615) in Javascript

##Counting syllables

To count syllables the number of syllables, you need only to count how many
vowels there are. Because all vowels in the dictionary have a number next
to them (for stress), you can simply count how many times those numbers
occur. A simple implementation in Python:

	def syllable_count(phones):
		return sum([phones.count(i) for i in '012'])

In Javascript (using underscore):

	function syllableCount(phones) {
	  return _.reduce(
	    _.map(phones, function(i) { return (i.match(/[012]/g)||[]).length; }),
	    function (a, b) { return a+b; })
	}

##Rhymes

[Code example](https://github.com/aparrish/rwet-examples/blob/master/pronouncing/cmudict.py) in Python. Note: to use this example code, you'll need to download
both the Python file and the CMU pronouncing dictionary itself. [Similar code example](https://github.com/aparrish/pronouncingjs/blob/master/pronouncing.js) in Javascript.

Rhyming isn't as simple as it seems! Many people when asked think that 
two words rhyme if they both have the same final syllable. This isn't true:
think "lessen" and "strengthen"---same last syllable, but hardly rhymes. A
better way to think about rhymes: two words rhyme if they have matching
"rhyming parts," where the "rhyming part" is defined as "everything from the
stressed syllable nearest the end of the word up to the end of the word."

Here's an implementation in Python of finding the "rhyming part" of a word,
based on its CMU pronunciation:

	def rhyming_part(phones):
		idx = 0
		phones_list = phones.split()
		for i in reversed(range(0, len(phones_list))):
			if phones_list[i][-1] in ('1', '2'):
				idx = i
				break
		return ' '.join(phones_list[idx:])

##Shortcomings

The CMU dictionary is amazing, but it isn't perfect! Here are some shortcomings
to watch out for:

* The pronunciations are sometimes inaccurate (I've found this to be true particularly with stress assignment)
* While multiple pronunciations are provided, no information is given about the contexts in which these variations occur: whether one of the alternate pronunciations is a regional variation, or whether the pronunciations are different based on how the word is being used (as, e.g., a noun or a verb)
* Stress is provided, but syllable boundaries are not.
* Even though the dictionary has over 100,000 entries, it's still woefully incomplete!

