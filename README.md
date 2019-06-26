# gpt-2-keyword-generation

A method for encoding a dataset of text documents into a form that when finetuned with GPT-2, the network will be able to generate text pertaining to the specified keywords (although the encoded text can theoetically work with any type of text-based network, it leverages GPT-2's long horizon and strong context abilities).

The encoding is parallelized using ray in order to massively speed up encoding on large datasets (about 11x speedup using 32 vCPUs/threads vs. single threaded, at 70% CPU utilization)

## Simple Usage

This repo contains a script which attempts to extract the keywords in an unsupervised manner (although you can provide your own keywords if you have them). The methodology is as follows for each text document:

1. Extract the keywords from each document as "keywords" using spaCy, which both tokenizes keywords and tags their parts-of-speech.
	* Only nouns, verbs, adjectives, and adverbs are extracted. Nouns use the raw version of the word (for best user experience when they input them manually) while the other POS use the lemmatized versions (to reduce overfitting but still provide information).
	* Proper nouns, named entities, and compound nouns count as their own keyword.
	* Pronouns and stop words are excluded from keywords.
	* Keywords are deduped.
2. Prepare the keywords in such a way that the document text is generated conditionally on the keywords.
	* Normalize the keywords (replace spaces/punctuation w/ dashes) and dedupe the keywords in a given document. The keywords are *not* case-normalized for best user experience when specifying keywords.
	* Shuffle the order of the keywords to prevent GPT-2 from cheating and learning when the order of the keywords should be written in the document proper.
	* For each set of keywords in a document, create `repeat` random combinations (default: 3) of the keywords
	* Select a random number of *up to* `max_keywords` (default: 3), which are then shuffled, to prevent the neural network from a) learning the number of keywords as a hint to the length of the text and b) the order of the keywords in the resulting text.
3. Write the keywords, then the document for each generated set of keywords.

The default case (passing a CSV of `titles`) generates `keywords`, and outputs a `.txt` of keywords and titles.

## Advanced Usage

This script is also capable of handling additional hierarchal conditions. This script has 4 total possibilities implemented:
`category`, `keywords`, `title`, and `body`.

`category` is the broadest scope of a given text. (e.g. the subreddit of a given post, the speaker of a given phrase if using for chatbots)

`body` is used if there's a large amount of text dependant on `title` (e.g. a blog post).

## Helpful Notes

* The scope of the text document(s) plus the keywords must be within gpt-2's max scope (e.g. should only be a paragraph or two max).
* Manual tagging may work better, and that is an option that can be passed.
* There should be an equal amount of all `category` documents to prevent sampling bias.
* The delimeters are chosen to be single, uncommon ASCII characters that are relatively unlikely to be used anywhere else, such that the network explicitly learns the significance of those characters. (see [Wired](https://www.wired.com/2013/08/the-rarity-of-the-ampersand/) and [Stack Overflow](https://stackoverflow.com/questions/492090/least-used-delimiter-character-in-normal-text-ascii-128) on the character rarity)

## Maintainer/Creator

Max Woolf ([@minimaxir](https://minimaxir.com))

*Max's open-source projects are supported by his [Patreon](https://www.patreon.com/minimaxir). If you found this project helpful, any monetary contributions to the Patreon are appreciated and will be put to good creative use.*

## License

MIT

## Disclaimer

This repo has no affiliation or relationship with OpenAI.