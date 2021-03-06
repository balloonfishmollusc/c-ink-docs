# Part 1: The Basics

## 1) Content

### The simplest ink script

The most basic ink script is just text in a .ink file.

	Hello, world!

On running, this will output the content, and then stop.

Text on separate lines produces new paragraphs. The script:

	Hello, world!
	Hello?
	Hello, are you there?

produces output that looks the same.


### Comments

By default, all text in your file will appear in the output content, unless specially marked up.

The simplest mark-up is a comment. **ink** supports two kinds of comment. There's the kind used for someone reading the code, which the compiler ignores:

	"What do you make of this?" she asked.
	
	// Something unprintable...
	
	"I couldn't possibly comment," I replied.
	
	/*
		... or an unlimited block of text
	*/

and there's the kind used for reminding the author what they need to do, that the compiler prints out during compilation:


	TODO: Write this section properly!

### Tags

Text content from the game will appear 'as is' when the engine runs. However, it can sometimes be useful to mark up a line of content with extra information to tell the game what to do with that content.

**ink** provides a simple system for tagging lines of content, with hashtags.

	A line of normal game-text. # colour it blue

These don't show up in the main text flow, but can be read off by the game and used as you see fit. See [Running Your Ink](RunningYourInk.md#marking-up-your-ink-content-with-tags) for more information.


## 2) Choices

Input is offered to the player via text choices. A text choice is indicated by an `*` character.

If no other flow instructions are given, once made, the choice will flow into the next line of text.

	Hello world!
	*	Hello back!
		Nice to hear from you!

This produces the following game:

	Hello world
	1: Hello back!
	
	> 1
	Hello back!
	Nice to hear from you.

By default, the text of a choice appears again, in the output.

### Suppressing choice text

Some games separate the text of a choice from its outcome. In **ink**, if the choice text is given in square brackets, the text of the choice will not be printed into response.

	Hello world!
	*	[Hello back!]
		Nice to hear from you!

produces

	Hello world
	1: Hello back!
	
	> 1
	Nice to hear from you.

#### Advanced: mixing choice and output text

The square brackets in fact divide up the option content. What's before is printed in both choice and output; what's inside only in choice; and what's after, only in output. Effectively, they provide alternative ways for a line to end.

	Hello world!
	*	Hello [back!] right back to you!
		Nice to hear from you!

produces:

	Hello world
	1: Hello back!
	> 1
	Hello right back to you!
	Nice to hear from you.

This is most useful when writing dialogue choices:

	"What's that?" my master asked.
	*	"I am somewhat tired[."]," I repeated.
		"Really," he responded. "How deleterious."

produces:

	"What's that?" my master asked.
	1. "I am somewhat tired."
	> 1
	"I am somewhat tired," I repeated.
	"Really," he responded. "How deleterious."

### Multiple Choices

To make choices really choices, we need to provide alternatives. We can do this simply by listing them:

	"What's that?" my master asked.
	*	"I am somewhat tired[."]," I repeated.
		"Really," he responded. "How deleterious."
	*	"Nothing, Monsieur!"[] I replied.
		"Very good, then."
	*  "I said, this journey is appalling[."] and I want no more of it."
		"Ah," he replied, not unkindly. "I see you are feeling frustrated. Tomorrow, things will improve."

This produces the following game:

	"What's that?" my master asked.
	
	1: "I am somewhat tired."
	2: "Nothing, Monsieur!"
	3: "I said, this journey is appalling."
	
	> 3
	"I said, this journey is appalling and I want no more of it."
	"Ah," he replied, not unkindly. "I see you are feeling frustrated. Tomorrow, things will improve."

The above syntax is enough to write a single set of choices. In a real game, we'll want to move the flow from one point to another based on what the player chooses. To do that, we need to introduce a bit more structure.

## 3) Knots

### Pieces of content are called knots

To allow the game to branch we need to mark up sections of content with names (as an old-fashioned gamebook does with its 'Paragraph 18', and the like.)

These sections are called "knots" and they're the fundamental structural unit of ink content.

### Writing a knot

The start of a knot is indicated by two or more equals signs, as follows.

	=== top_knot ===

(The equals signs on the end are optional; and the name needs to be a single word with no spaces.)

The start of a knot is a header; the content that follows will be inside that knot.

	=== back_in_london ===
	
	We arrived into London at 9.45pm exactly.

#### Advanced: a knottier "hello world"

When you start an ink file, content outside of knots will be run automatically. But knots won't. So if you start using knots to hold your content, you'll need to tell the game where to go. We do this with a divert arrow `->`, which is covered properly in the next section.

The simplest knotty script is:

	-> top_knot
	
	=== top_knot ===
	Hello world!

However, **ink** doesn't like loose ends, and produces a warning on compilation and/or run-time when it thinks this has happened. The script above produces this on compilation:

	WARNING: Apparent loose end exists where the flow runs out. Do you need a '-> END' statement, choice or divert? on line 3 of tests/test.ink

and this on running:

	Runtime error in tests/test.ink line 3: ran out of content. Do you need a '-> DONE' or '-> END'?

The following plays and compiles without error:

	=== top_knot ===
	Hello world!
	-> END

`-> END` is a marker for both the writer and the compiler; it means "the story flow should now stop".

## 4) Diverts

### Knots divert to knots

You can tell the story to move from one knot to another using `->`, a "divert arrow". Diverts happen immediately without any user input.

	=== back_in_london ===
	
	We arrived into London at 9.45pm exactly.
	-> hurry_home
	
	=== hurry_home ===
	We hurried home to Savile Row as fast as we could.

#### Diverts are invisible

Diverts are intended to be seamless and can even happen mid-sentence:

	=== hurry_home ===
	We hurried home to Savile Row -> as_fast_as_we_could
	
	=== as_fast_as_we_could ===
	as fast as we could.

produces the same line as above:

	We hurried home to Savile Row as fast as we could.

#### Glue

The default behaviour inserts line-breaks before every new line of content. In some cases, however, content must insist on not having a line-break, and it can do so using `<>`, or "glue".

	=== hurry_home ===
	We hurried home <>
	-> to_savile_row
	
	=== to_savile_row ===
	to Savile Row
	-> as_fast_as_we_could
	
	=== as_fast_as_we_could ===
	<> as fast as we could.

also produces:

	We hurried home to Savile Row as fast as we could.

You can't use too much glue: multiple glues next to each other have no additional effect. (And there's no way to "negate" a glue; once a line is sticky, it'll stick.)


## 5) Branching The Flow

### Basic branching

Combining knots, options and diverts gives us the basic structure of a choose-your-own game.

	=== paragraph_1 ===
	You stand by the wall of Analand, sword in hand.
	* [Open the gate] -> paragraph_2
	* [Smash down the gate] -> paragraph_3
	* [Turn back and go home] -> paragraph_4
	
	=== paragraph_2 ===
	You open the gate, and step out onto the path.
	
	...

### Branching and joining

Using diverts, the writer can branch the flow, and join it back up again, without showing the player that the flow has rejoined.

	=== back_in_london ===
	
	We arrived into London at 9.45pm exactly.
	
	*	"There is not a moment to lose!"[] I declared.
		-> hurry_outside
	
	*	"Monsieur, let us savour this moment!"[] I declared.
		My master clouted me firmly around the head and dragged me out of the door.
		-> dragged_outside
	
	*	[We hurried home] -> hurry_outside


	=== hurry_outside ===
	We hurried home to Savile Row -> as_fast_as_we_could


	=== dragged_outside ===
	He insisted that we hurried home to Savile Row
	-> as_fast_as_we_could


	=== as_fast_as_we_could ===
	<> as fast as we could.


### The story flow

Knots and diverts combine to create the basic story flow of the game. This flow is "flat" - there's no call-stack, and diverts aren't "returned" from.

In most ink scripts, the story flow starts at the top, bounces around in a spaghetti-like mess, and eventually, hopefully, reaches a `-> END`.

The very loose structure means writers can get on and write, branching and rejoining without worrying about the structure that they're creating as they go. There's no boiler-plate to creating new branches or diversions, and no need to track any state.

#### Advanced: Loops

You absolutely can use diverts to create looped content, and **ink** has several features to exploit this, including ways to make the content vary itself, and ways to control how often options can be chosen.

See the sections on Varying Text and [Conditional Choices](#conditional-choices) for more information.

Oh, and the following is legal and not a great idea:

	=== round ===
	and
	-> round

## 6) Includes and Stitches

### Knots can be subdivided

As stories get longer, they become more confusing to keep organised without some additional structure.

Knots can include sub-sections called "stitches". These are marked using a single equals sign.

	=== the_orient_express ===
	= in_first_class
		...
	= in_third_class
		...
	= in_the_guards_van
		...
	= missed_the_train
		...

One could use a knot for a scene, for instance, and stitches for the events within the scene.

### Stitches have unique names

A stitch can be diverted to using its "address".

	*	[Travel in third class]
		-> the_orient_express.in_third_class
	
	*	[Travel in the guard's van]
		-> the_orient_express.in_the_guards_van

### The first stitch is the default

Diverting to a knot which contains stitches will divert to the first stitch in the knot. So:

	*	[Travel in first class]
		"First class, Monsieur. Where else?"
		-> the_orient_express

is the same as:

	*	[Travel in first class]
		"First class, Monsieur. Where else?"
		-> the_orient_express.in_first_class

(...unless we move the order of the stitches around inside the knot!)

You can also include content at the top of a knot outside of any stitch. However, you need to remember to divert out of it - the engine *won't* automatically enter the first stitch once it's worked its way through the header content.

	=== the_orient_express ===
	
	We boarded the train, but where?
	*	[First class] -> in_first_class
	*	[Second class] -> in_second_class
	
	= in_first_class
		...
	= in_second_class
		...


### Local diverts

From inside a knot, you don't need to use the full address for a stitch.

	-> the_orient_express
	
	=== the_orient_express ===
	= in_first_class
		I settled my master.
		*	[Move to third class]
			-> in_third_class
	
	= in_third_class
		I put myself in third.

This means stitches and knots can't share names, but several knots can contain the same stitch name. (So both the Orient Express and the SS Mongolia can have first class.)

The compiler will warn you if ambiguous names are used.

### Script files can be combined

You can also split your content across multiple files, using an include statement.

	INCLUDE newspaper.ink
	INCLUDE cities/vienna.ink
	INCLUDE journeys/orient_express.ink

Include statements should always go at the top of a file, and not inside knots.

There are no rules about what file a knot must be in to be diverted to. (In other words, separating files has no effect on the game's namespacing).


## 7) Varying Choices

### Choices can only be used once

By default, every choice in the game can only be chosen once. If you don't have loops in your story, you'll never notice this behaviour. But if you do use loops, you'll quickly notice your options disappearing...

	=== find_help ===
	
		You search desperately for a friendly face in the crowd.
		*	The woman in the hat[?] pushes you roughly aside. -> find_help
		*	The man with the briefcase[?] looks disgusted as you stumble past him. -> find_help

produces:

	You search desperately for a friendly face in the crowd.
	
	1: The woman in the hat?
	2: The man with the briefcase?
	
	> 1
	The woman in the hat pushes you roughly aside.
	You search desperately for a friendly face in the crowd.
	
	1: The man with the briefcase?
	
	>

... and on the next loop you'll have no options left.

#### Fallback choices

The above example stops where it does, because the next choice ends up in an "out of content" run-time error.

	> 1
	The man with the briefcase looks disgusted as you stumble past him.
	You search desperately for a friendly face in the crowd.
	
	Runtime error in tests/test.ink line 6: ran out of content. Do you need a '-> DONE' or '-> END'?

We can resolve this with a 'fallback choice'. Fallback choices are never displayed to the player, but are 'chosen' by the game if no other options exist.

A fallback choice is simply a "choice without choice text":

	*	-> out_of_options

And, in a slight abuse of syntax, we can make a default choice with content in it, using an "choice then arrow":

	* 	->
		Mulder never could explain how he got out of that burning box car. -> season_2

#### Example of a fallback choice

Adding this into the previous example gives us:

	=== find_help ===
	
		You search desperately for a friendly face in the crowd.
		*	The woman in the hat[?] pushes you roughly aside. -> find_help
		*	The man with the briefcase[?] looks disgusted as you stumble past him. -> find_help
		*	->
			But it is too late: you collapse onto the station platform. This is the end.
			-> END

and produces:

	You search desperately for a friendly face in the crowd.
	
	1: The woman in the hat?
	2: The man with the briefcase?
	
	> 1
	The woman in the hat pushes you roughly aside.
	You search desperately for a friendly face in the crowd.
	
	1: The man with the briefcase?
	
	> 1
	The man with the briefcase looks disgusted as you stumble past him.
	You search desperately for a friendly face in the crowd.
	But it is too late: you collapse onto the station platform. This is the end.


### Sticky choices

The 'once-only' behaviour is not always what we want, of course, so we have a second kind of choice: the "sticky" choice. A sticky choice is simply one that doesn't get used up, and is marked by a `+` bullet.

	=== homers_couch ===
		+	[Eat another donut]
			You eat another donut. -> homers_couch
		*	[Get off the couch]
			You struggle up off the couch to go and compose epic poetry.
			-> END

Fallback choices can be sticky too.

	=== conversation_loop
		*	[Talk about the weather] -> chat_weather
		*	[Talk about the children] -> chat_children
		+	-> sit_in_silence_again

### Conditional Choices

You can also turn choices on and off by hand. **ink** has quite a lot of logic available, but the simplest tests is "has the player seen a particular piece of content".

Every knot/stitch in the game has a unique address (so it can be diverted to), and we use the same address to test if that piece of content has been seen.

	*	{ not visit_paris } 	[Go to Paris] -> visit_paris
	+ 	{ visit_paris 	 } 		[Return to Paris] -> visit_paris
	
	*	{ visit_paris.met_estelle } [ Telephone Mme Estelle ] -> phone_estelle

Note that the test `knot_name` is true if *any* stitch inside that knot has been seen.

Note also that conditionals don't override the once-only behaviour of options, so you'll still need sticky options for repeatable choices.

#### Advanced: multiple conditions

You can use several logical tests on an option; if you do, *all* the tests must all be passed for the option to appear.

	*	{ not visit_paris } 	[Go to Paris] -> visit_paris
	+ 	{ visit_paris } { not bored_of_paris }
		[Return to Paris] -> visit_paris

#### Logical operators: AND and OR

The above "multiple conditions" are really just conditions with an the usual programming AND operator. Ink supports `and` (also written as `&&`) and `or` (also written as `||`) in the usual way, as well as brackets.

	*	{ not (visit_paris or visit_rome) && (visit_london || visit_new_york) } [ Wait. Go where? I'm confused. ] -> visit_someplace

For non-programmers `X and Y` means both X and Y must be true. `X or Y` means either or both. We don't have a `xor`.

You can also use the standard `!` for `not`, though it'll sometimes confuse the compiler which thinks `{!text}` is a once-only list. We recommend using `not` because negated boolean tests are never that exciting.

#### Advanced: knot/stitch labels are actually read counts

The test:

	*	{seen_clue} [Accuse Mr Jefferson]

is actually testing an *integer* and not a true/false flag. A knot or stitch used this way is actually an integer variable containing the number of times the content at the address has been seen by the player.

If it's non-zero, it'll return true in a test like the one above, but you can also be more specific as well:

	* {seen_clue > 3} [Flat-out arrest Mr Jefferson]


#### Advanced: more logic

**ink** supports a lot more logic and conditionality than covered here - see the section on [variables and logic](#part-3-variables-and-logic).


## 8) Variable Text

### Text can vary

So far, all the content we've seen has been static, fixed pieces of text. But content can also vary at the moment of being printed.

### Sequences, cycles and other alternatives

The simplest variations of text are provided by alternatives, which are selected from depending on some kind of rule. **ink** supports several types. Alternatives are written inside `{`...`}` curly brackets, with elements separated by `|` symbols (vertical divider lines).

These are only useful if a piece of content is visited more than once!

#### Types of alternatives

**Sequences** (the default):

A sequence (or a "stopping block") is a set of alternatives that tracks how many times its been seen, and each time, shows the next element along. When it runs out of new content it continues the show the final element.

	The radio hissed into life. {"Three!"|"Two!"|"One!"|There was the white noise racket of an explosion.|But it was just static.}
	
	{I bought a coffee with my five-pound note.|I bought a second coffee for my friend.|I didn't have enough money to buy any more coffee.}

**Cycles** (marked with a `&`):

Cycles are like sequences, but they loop their content.

	It was {&Monday|Tuesday|Wednesday|Thursday|Friday|Saturday|Sunday} today.


**Once-only** (marked with a `!`):

Once-only alternatives are like sequences, but when they run out of new content to display, they display nothing. (You can think of a once-only alternative as a sequence with a blank last entry.)

	He told me a joke. {!I laughed politely.|I smiled.|I grimaced.|I promised myself to not react again.}

**Shuffles** (marked with a `~`):

Shuffles produce randomised output.

	I tossed the coin. {~Heads|Tails}.

#### Features of Alternatives

Alternatives can contain blank elements.

	I took a step forward. {!||||Then the lights went out. -> eek}

Alternatives can be nested.

	The Ratbear {&{wastes no time and |}swipes|scratches} {&at you|into your {&leg|arm|cheek}}.

Alternatives can include divert statements.

	I {waited.|waited some more.|snoozed.|woke up and waited more.|gave up and left. -> leave_post_office}

They can also be used inside choice text:

	+ 	"Hello, {&Master|Monsieur Fogg|you|brown-eyes}!"[] I declared.

(...with one caveat; you can't start an option's text with a `{`, as it'll look like a conditional.)

(...but the caveat has a caveat, if you escape a whitespace `\ ` before your `{` ink will recognise it as text.)

	+\	{&They headed towards the Sandlands|They set off for the desert|The party followed the old road South}

#### Examples

Alternatives can be used inside loops to create the appearance of intelligent, state-tracking gameplay without particular effort.

Here's a one-knot version of whack-a-mole. Note we use once-only options, and a fallback, to ensure the mole doesn't move around, and the game will always end.

	=== whack_a_mole ===
		{I heft the hammer.|{~Missed!|Nothing!|No good. Where is he?|Ah-ha! Got him! -> END}}
		The {&mole|{&nasty|blasted|foul} {&creature|rodent}} is {in here somewhere|hiding somewhere|still at large|laughing at me|still unwhacked|doomed}. <>
		{!I'll show him!|But this time he won't escape!}
		* 	[{&Hit|Smash|Try} top-left] 	-> whack_a_mole
		*  [{&Whallop|Splat|Whack} top-right] -> whack_a_mole
		*  [{&Blast|Hammer} middle] -> whack_a_mole
		*  [{&Clobber|Bosh} bottom-left] 	-> whack_a_mole
		*  [{&Nail|Thump} bottom-right] 	-> whack_a_mole
		*   ->
	    	    Then you collapse from hunger. The mole has defeated you!
	            -> END


produces the following 'game':

	I heft the hammer.
	The mole is in here somewhere. I'll show him!
	
	1: Hit top-left
	2: Whallop top-right
	3: Blast middle
	4: Clobber bottom-left
	5: Nail bottom-right
	
	> 1
	Missed!
	The nasty creature is hiding somewhere. But this time he won't escape!
	
	1: Splat top-right
	2: Hammer middle
	3: Bosh bottom-left
	4: Thump bottom-right
	
	> 4
	Nothing!
	The mole is still at large.
	1: Whack top-right
	2: Blast middle
	3: Clobber bottom-left
	
	> 2
	Where is he?
	The blasted rodent is laughing at me.
	1: Whallop top-right
	2: Bosh bottom-left
	
	> 1
	Ah-ha! Got him!


And here's a bit of lifestyle advice. Note the sticky choice - the lure of the television will never fade:

	=== turn_on_television ===
	I turned on the television {for the first time|for the second time|again|once more}, but there was {nothing good on, so I turned it off again|still nothing worth watching|even less to hold my interest than before|nothing but rubbish|a program about sharks and I don't like sharks|nothing on}.
	+	[Try it again]	 		-> turn_on_television
	*	[Go outside instead]	-> go_outside_instead
	
	=== go_outside_instead ===
	-> END



#### Sneak Preview: Multiline alternatives

**ink** has another format for making alternatives of varying content blocks, too. See the section on [multiline blocks](#multiline-blocks) for details.



### Conditional Text

Text can also vary depending on logical tests, just as options can.

	{met_blofeld: "I saw him. Only for a moment." }

and

	"His real name was {met_blofeld.learned_his_name: Franz|a secret}."

These can appear as separate lines, or within a section of content. They can even be nested, so:

	{met_blofeld: "I saw him. Only for a moment. His real name was {met_blofeld.learned_his_name: Franz|kept a secret}." | "I missed him. Was he particularly evil?" }

can produce either:

	"I saw him. Only for a moment. His real name was Franz."

or:

	"I saw him. Only for a moment. His real name was kept a secret."

or:

	"I missed him. Was he particularly evil?"

## 9) Game Queries and Functions

**ink** provides a few useful 'game level' queries about game state, for use in conditional logic. They're not quite parts of the language, but they're always available, and they can't be edited by the author. In a sense, they're the "standard library functions" of the language.

The convention is to name these in capital letters.

### CHOICE_COUNT()

`CHOICE_COUNT` returns the number of options created so far in the current chunk. So for instance.

	*	{false} Option A
	* 	{true} Option B
	*  {CHOICE_COUNT() == 1} Option C

produces two options, B and C. This can be useful for controlling how many options a player gets on a turn.

### TURNS()

This returns the number of game turns since the game began.

### TURNS_SINCE(-> knot)

`TURNS_SINCE` returns the number of moves (formally, player inputs) since a particular knot/stitch was last visited.

A value of 0 means "was seen as part of the current chunk". A value of -1 means "has never been seen". Any other positive value means it has been seen that many turns ago.

	*	{TURNS_SINCE(-> sleeping.intro) > 10} You are feeling tired... -> sleeping
	* 	{TURNS_SINCE(-> laugh) == 0}  You try to stop laughing.

Note that the parameter passed to `TURNS_SINCE` is a "divert target", not simply the knot address itself (because the knot address is a number - the read count - not a location in the story...)

TODO: (requirement of passing `-c` to the compiler)

#### Sneak preview: using TURNS_SINCE in a function

The `TURNS_SINCE(->x) == 0` test is so useful it's often worth wrapping it up as an ink function.

	=== function came_from(-> x)
		~ return TURNS_SINCE(x) == 0

The section on [functions](#5-functions) outlines the syntax here a bit more clearly but the above allows you to say things like:

	* {came_from(->  nice_welcome)} 'I'm happy to be here!'
	* {came_from(->  nasty_welcome)} 'Let's keep this quick.'

... and have the game react to content the player saw *just now*.

### SEED_RANDOM()

For testing purposes, it's often useful to fix the random number generator so ink will produce the same outcomes every time you play. You can do this by "seeding" the random number system.

	~ SEED_RANDOM(235)

The number you pass to the seed function is arbitrary, but providing different seeds will result in different sequences of outcomes.

#### Advanced: more queries

You can make your own external functions, though the syntax is a bit different: see the section on [functions](#5-functions) below.