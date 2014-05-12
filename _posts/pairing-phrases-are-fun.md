# Pairing Phrase generation

We wanted people to pair with something more human than a random sequence of characters.

Requirements:

* 5,000,000 unique pairing phrases of the form `adjective` + `noun`
* no more than 1/1,000 should be offensive
* no more than 1/100 should be weird

We tried various things, like assembling word lists and running natural language processing (NLP) on a large corpus of text.

Ultimately, we retrieved the bigrams from Google, pulling phrases with at least 50,000 uses in the last 20 years.

From there we pruned words taken from [Luis von Ahn](http://www.cs.cmu.edu/~biglou/)'s [bad word list](http://www.cs.cmu.edu/~biglou/resources/bad-words.txt). Some adjectives that got cut:

* `angry`
* `anonymous`
* `awful`
* `bad`
* `dead`
* `homeless`
* `holy`
* `naked`
* `religious`
* `wet`

And some nouns that were stricken from the record:

* `ball`
* `church`
* `drug`
* `fraud`
* `hole`
* `period`
* `pot`
* `slave`
* `weapon`

There are some more obscure pairings that lead to potentially offensive phrases like `stiff bone` (okay, we should have seen that one coming, but still!).

As the language evolves and slang appears, adjectives like `basic` are [redefined](http://www.urbandictionary.com/define.php?term=basic), so we occasionally have to update the list.

So, if you ever get an offensive or weird pairing phrase, rejoice in knowing that you are special.
