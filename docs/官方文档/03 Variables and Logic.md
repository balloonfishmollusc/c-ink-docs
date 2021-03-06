# Part 3: Variables and Logic

So far we've made conditional text, and conditional choices, using tests based on what content the player has seen so far.

**ink** also supports variables, both temporary and global, storing numerical and content data, or even story flow commands. It is fully-featured in terms of logic, and contains a few additional structures to help keep the often complex logic of a branching story better organised.


## 1) Global Variables

The most powerful kind of variable, and arguably the most useful for a story, is a variable to store some unique property about the state of the game - anything from the amount of money in the protagonist's pocket, to a value representing the protagonist's state of mind.

This kind of variable is called "global" because it can be accessed from anywhere in the story - both set, and read from. (Traditionally, programming tries to avoid this kind of thing, as it allows one part of a program to mess with another, unrelated part. But a story is a story, and stories are all about consequences: what happens in Vegas rarely stays there.)

### Defining Global Variables

Global variables can be defined anywhere, via a `VAR` statement. They should be given an initial value, which defines what type of variable they are - integer, floating point (decimal), content, or a story address.

	VAR knowledge_of_the_cure = false
	VAR players_name = "Emilia"
	VAR number_of_infected_people = 521
	VAR current_epilogue = -> they_all_die_of_the_plague

### Using Global Variables

We can test global variables to control options, and provide conditional text, in a similar way to what we have previously seen.

	=== the_train ===
		The train jolted and rattled. { mood > 0:I was feeling positive enough, however, and did not mind the odd bump|It was more than I could bear}.
		*	{ not knows_about_wager } 'But, Monsieur, why are we travelling?'[] I asked.
		* 	{ knows_about_wager} I contemplated our strange adventure[]. Would it be possible?

#### Advanced: storing diverts as variables

A "divert" statement is actually a type of value in itself, and can be stored, altered, and diverted to.

	VAR 	current_epilogue = -> everybody_dies
	
	=== continue_or_quit ===
	Give up now, or keep trying to save your Kingdom?
	*  [Keep trying!] 	-> more_hopeless_introspection
	*  [Give up] 		-> current_epilogue


#### Advanced: Global variables are externally visible

Global variables can be accessed, and altered, from the runtime as well from the story, so provide a good way to communicate between the wider game and the story.

The **ink** layer is often be a good place to store gameplay-variables; there's no save/load issues to consider, and the story itself can react to the current values.



### Printing variables

The value of a variable can be printed as content using an inline syntax similar to sequences, and conditional text:

	VAR friendly_name_of_player = "Jackie"
	VAR age = 23
	
	My name is Jean Passepartout, but my friend's call me {friendly_name_of_player}. I'm {age} years old.

This can be useful in debugging. For more complex printing based on logic and variables, see the section on functions.

### Evaluating strings

It might be noticed that above we refered to variables as being able to contain "content", rather than "strings". That was deliberate, because a string defined in ink can contain ink - although it will always evaluate to a string. (Yikes!)

	VAR a_colour = ""
	
	~ a_colour = "{~red|blue|green|yellow}"
	
	{a_colour}

... produces one of red, blue, green or yellow.

Note that once a piece of content like this is evaluated, its value is "sticky". (The quantum state collapses.) So the following:

	The goon hits you, and sparks fly before you eyes, {a_colour} and {a_colour}.

... won't produce a very interesting effect. (If you really want this to work, use a text function to print the colour!)

This is also why

	VAR a_colour = "{~red|blue|green|yellow}"

is explicitly disallowed; it would be evaluated on the construction of the story, which probably isn't what you want.


## 2) Logic

Obviously, our global variables are not intended to be constants, so we need a syntax for altering them.

Since by default, any text in an **ink** script is printed out directly to the screen, we use a markup symbol to indicate that a line of content is intended meant to be doing some numerical work, we use the `~` mark.

The following statements all assign values to variables:


	=== set_some_variables ===
		~ knows_about_wager = true
		~ x = (x * x) - (y * y) + c
		~ y = 2 * x * y

and the following will test conditions:

	{ x == 1.2 }
	{ x / 2 > 4 }
	{ y - 1 <= x * x }

### Mathematics

**ink** supports the four basic mathematical operations (`+`, `-`, `*` and `/`), as well as `%` (or `mod`), which returns the remainder after integer division. There's also POW for to-the-power-of:

	{POW(3, 2)} is 9.
	{POW(16, 0.5)} is 4.


If more complex operations are required, one can write functions (using recursion if necessary), or call out to external, game-code functions (for anything more advanced).


#### RANDOM(min, max)

Ink can generate random integers if required using the RANDOM function. RANDOM is authored to be like a dice (yes, pendants, we said *a dice*), so the min and max values are both inclusive.

	~ temp dice_roll = RANDOM(1, 6)
	
	~ temp lazy_grading_for_test_paper = RANDOM(30, 75)
	
	~ temp number_of_heads_the_serpent_has = RANDOM(3, 8)

The random number generator can be seeded for testing purposes, see the section of Game Queries and Functions section above.

#### Advanced: numerical types are implicit

Results of operations - in particular, for division - are typed based on the type of the input. So integer division returns integer, but floating point division returns floating point results.

	~ x = 2 / 3
	~ y = 7 / 3
	~ z = 1.2 / 0.5

assigns `x` to be 0, `y` to be 2 and `z` to be 2.4.

#### Advanced: INT(), FLOOR() and FLOAT()

In cases where you don't want implicit types, or you want to round off a variable, you can cast it directly.

	{INT(3.2)} is 3.
	{FLOOR(4.8)} is 4.
	{INT(-4.8)} is -4.
	{FLOOR(-4.8)} is -5.
	
	{FLOAT(4)} is, um, still 4.



### String queries

Oddly for a text-engine, **ink** doesn't have much in the way of string-handling: it's assumed that any string conversion you need to do will be handled by the game code (and perhaps by external functions.) But we support three basic queries - equality, inequality, and substring (which we call ? for reasons that will become clear in a later chapter).

The following all return true:

	{ "Yes, please." == "Yes, please." }
	{ "No, thank you." != "Yes, please." }
	{ "Yes, please" ? "ease" }


## 3) Conditional blocks (if/else)

We've seen conditionals used to control options and story content; **ink** also provides an equivalent of the normal if/else-if/else structure.

### A simple 'if'

The if syntax takes its cue from the other conditionals used so far, with the `{`...`}` syntax indicating that something is being tested.

	{ x > 0:
		~ y = x - 1
	}

Else conditions can be provided:

	{ x > 0:
		~ y = x - 1
	- else:
		~ y = x + 1
	}

### Extended if/else if/else blocks

The above syntax is actually a specific case of a more general structure, something like a "switch" statement of another language:

	{
		- x > 0:
			~ y = x - 1
		- else:
			~ y = x + 1
	}

And using this form we can include 'else-if' conditions:

	{
		- x == 0:
			~ y = 0
		- x > 0:
			~ y = x - 1
		- else:
			~ y = x + 1
	}

(Note, as with everything else, the white-space is purely for readability and has no syntactic meaning.)

### Switch blocks

And there's also an actual switch statement:

	{ x:
	- 0: 	zero
	- 1: 	one
	- 2: 	two
	- else: lots
	}

#### Example: context-relevant content

Note these tests don't have to be variable-based and can use read-counts, just as other conditionals can, and the following construction is quite frequent, as a way of saying "do some content which is relevant to the current game state":

	=== dream ===
		{
			- visited_snakes && not dream_about_snakes:
				~ fear++
				-> dream_about_snakes
	
			- visited_poland && not dream_about_polish_beer:
				~ fear--
				-> dream_about_polish_beer
	
			- else:
				// breakfast-based dreams have no effect
				-> dream_about_marmalade
		}

The syntax has the advantage of being easy to extend, and prioritise.



### Conditional blocks are not limited to logic

Conditional blocks can be used to control story content as well as logic:

	I stared at Monsieur Fogg.
	{ know_about_wager:
		<> "But surely you are not serious?" I demanded.
	- else:
		<> "But there must be a reason for this trip," I observed.
	}
	He said nothing in reply, merely considering his newspaper with as much thoroughness as entomologist considering his latest pinned addition.

You can even put options inside conditional blocks:

	{ door_open:
		* 	I strode out of the compartment[] and I fancied I heard my master quietly tutting to himself. 			-> go_outside
	- else:
		*	I asked permission to leave[] and Monsieur Fogg looked surprised. 	-> open_door
		* 	I stood and went to open the door[]. Monsieur Fogg seemed untroubled by this small rebellion. -> open_door
	}

...but note that the lack of weave-syntax and nesting in the above example isn't accidental: to avoid confusing the various kinds of nesting at work, you aren't allowed to include gather points inside conditional blocks.

### Multiline blocks

There's one other class of multiline block, which expands on the alternatives system from above. The following are all valid and do what you might expect:

 	// Sequence: go through the alternatives, and stick on last
 	{ stopping:
 		-	I entered the casino.
 		-  I entered the casino again.
 		-  Once more, I went inside.
 	}
 	
 	// Shuffle: show one at random
 	At the table, I drew a card. <>
 	{ shuffle:
 		- 	Ace of Hearts.
 		- 	King of Spades.
 		- 	2 of Diamonds.
 			'You lose this time!' crowed the croupier.
 	}
 	
 	// Cycle: show each in turn, and then cycle
 	{ cycle:
 		- I held my breath.
 		- I waited impatiently.
 		- I paused.
 	}
 	
 	// Once: show each, once, in turn, until all have been shown
 	{ once:
 		- Would my luck hold?
 		- Could I win the hand?
 	}

#### Advanced: modified shuffles

The shuffle block above is really a "shuffled cycle"; in that it'll shuffle the content, play through it, then reshuffle and go again.

There are two other versions of shuffle:

`shuffle once` which will shuffle the content, play through it, and then do nothing.

	{ shuffle once:
	-	The sun was hot.
	- 	It was a hot day.
	}

`shuffle stopping` will shuffle all the content (except the last entry), and once its been played, it'll stick on the last entry.

	{ shuffle stopping:
	- 	A silver BMW roars past.
	-	A bright yellow Mustang takes the turn.
	- 	There are like, cars, here.
	}


## 4) Temporary Variables

### Temporary variables are for scratch calculations

Sometimes, a global variable is unwieldy. **ink** provides temporary variables for quick calculations of things.

	=== near_north_pole ===
		~ temp number_of_warm_things = 0
		{ blanket:
			~ number_of_warm_things++
		}
		{ ear_muffs:
			~ number_of_warm_things++
		}
		{ gloves:
			~ number_of_warm_things++
		}
		{ number_of_warm_things > 2:
			Despite the snow, I felt incorrigibly snug.
		- else:
			That night I was colder than I have ever been.
		}

The value in a temporary variable is thrown away after the story leaves the stitch in which it was defined.

### Knots and stitches can take parameters

A particularly useful form of temporary variable is a parameter. Any knot or stitch can be given a value as a parameter.

	*	[Accuse Hasting]
			-> accuse("Hastings")
	*	[Accuse Mrs Black]
			-> accuse("Claudia")
	*	[Accuse myself]
			-> accuse("myself")
	
	=== accuse(who) ===
		"I accuse {who}!" Poirot declared.
		"Really?" Japp replied. "{who == "myself":You did it?|{who}?}"
		"And why not?" Poirot shot back.


... and you'll need to use parameters if you want to pass a temporary value from one stitch to another!

#### Example: a recursive knot definition

Temporary variables are safe to use in recursion (unlike globals), so the following will work.

	-> add_one_to_one_hundred(0, 1)
	
	=== add_one_to_one_hundred(total, x) ===
		~ total = total + x
		{ x == 100:
			-> finished(total)
		- else:
			-> add_one_to_one_hundred(total, x + 1)
		}
	
	=== finished(total) ===
		"The result is {total}!" you announce.
		Gauss stares at you in horror.
		-> END


(In fact, this kind of definition is useful enough that **ink** provides a special kind of knot, called, imaginatively enough, a `function`, which comes with certain restrictions and can return a value. See the section below.)


#### Advanced: sending divert targets as parameters

Knot/stitch addresses are a type of value, indicated by a `->` character, and can be stored and passed around. The following is therefore legal, and often useful:

	=== sleeping_in_hut ===
		You lie down and close your eyes.
		-> generic_sleep (-> waking_in_the_hut)
	
	===	 generic_sleep (-> waking)
		You sleep perchance to dream etc. etc.
		-> waking
	
	=== waking_in_the_hut
		You get back to your feet, ready to continue your journey.

...but note the `->` in the `generic_sleep` definition: that's the one case in **ink** where a parameter needs to be typed: because it's too easy to otherwise accidentally do the following:

	=== sleeping_in_hut ===
		You lie down and close your eyes.
		-> generic_sleep (waking_in_the_hut)

... which sends the read count of `waking_in_the_hut` into the sleeping knot, and then attempts to divert to it.





## 5) Functions

The use of parameters on knots means they are almost functions in the usual sense, but they lack one key concept - that of the call stack, and the use of return values.

**ink** includes functions: they are knots, with the following limitations and features:

A function:

- cannot contain stitches
- cannot use diverts or offer choices
- can call other functions
- can include printed content
- can return a value of any type
- can recurse safely

(Some of these may seem quite limiting, but for more story-oriented call-stack-style features, see the section on [Tunnels](#1-tunnels).)

Return values are provided via the `~ return` statement.

### Defining and calling functions

To define a function, simply declare a knot to be one:

	=== function say_yes_to_everything ===
		~ return true
	
	=== function lerp(a, b, k) ===
		~ return ((b - a) * k) + a

Functions are called by name, and with brackets, even if they have no parameters:

	~ x = lerp(2, 8, 0.3)
	
	*	{say_yes_to_everything()} 'Yes.'

As in any other language, a function, once done, returns the flow to wherever it was called from - and despite not being allowed to divert the flow, functions can still call other functions.

	=== function say_no_to_nothing ===
		~ return say_yes_to_everything()

### Functions don't have to return anything

A function does not need to have a return value, and can simply do something that is worth packaging up:

	=== function harm(x) ===
		{ stamina < x:
			~ stamina = 0
		- else:
			~ stamina = stamina - x
		}

...though remember a function cannot divert, so while the above prevents a negative Stamina value, it won't kill a player who hits zero.

### Functions can be called inline

Functions can be called on `~` content lines, but can also be called during a piece of content. In this context, the return value, if there is one, is printed (as well as anything else the function wants to print.) If there is no return value, nothing is printed.

Content is, by default, 'glued in', so the following:

	Monsieur Fogg was looking {describe_health(health)}.
	
	=== function describe_health(x) ===
	{
	- x == 100:
		~ return "spritely"
	- x > 75:
		~ return "chipper"
	- x > 45:
		~ return "somewhat flagging"
	- else:
		~ return "despondent"
	}

produces:

	Monsieur Fogg was looking despondent.

#### Examples

For instance, you might include:

	=== function max(a,b) ===
		{ a < b:
			~ return b
		- else:
			~ return a
		}
	
	=== function exp(x, e) ===
		// returns x to the power e where e is an integer
		{ e <= 0:
			~ return 1
		- else:
			~ return x * exp(x, e - 1)
		}

Then:

	The maximum of 2^5 and 3^3 is {max(exp(2,5), exp(3,3))}.

produces:

	The maximum of 2^5 and 3^3 is 32.


#### Example: turning numbers into words

The following example is long, but appears in pretty much every inkle game to date. (Recall that a hyphenated line inside multiline curly braces indicates either "a condition to test" or, if the curly brace began with a variable, "a value to compare against".)

    === function print_num(x) ===
    {
        - x >= 1000:
            {print_num(x / 1000)} thousand { x mod 1000 > 0:{print_num(x mod 1000)}}
        - x >= 100:
            {print_num(x / 100)} hundred { x mod 100 > 0:and {print_num(x mod 100)}}
        - x == 0:
            zero
        - else:
            { x >= 20:
                { x / 10:
                    - 2: twenty
                    - 3: thirty
                    - 4: forty
                    - 5: fifty
                    - 6: sixty
                    - 7: seventy
                    - 8: eighty
                    - 9: ninety
                }
                { x mod 10 > 0:<>-<>}
            }
            { x < 10 || x > 20:
                { x mod 10:
                    - 1: one
                    - 2: two
                    - 3: three
                    - 4: four
                    - 5: five
                    - 6: six
                    - 7: seven
                    - 8: eight
                    - 9: nine
                }
            - else:
                { x:
                    - 10: ten
                    - 11: eleven
                    - 12: twelve
                    - 13: thirteen
                    - 14: fourteen
                    - 15: fifteen
                    - 16: sixteen
                    - 17: seventeen
                    - 18: eighteen
                    - 19: nineteen
                }
            }
    }

which enables us to write things like:

	~ price = 15
	
	I pulled out {print_num(price)} coins from my pocket and slowly counted them.
	"Oh, never mind," the trader replied. "I'll take half." And she took {print_num(price / 2)}, and pushed the rest back over to me.



### Parameters can be passed by reference

Function parameters can also be passed 'by reference', meaning that the function can actually alter the the variable being passed in, instead of creating a temporary variable with that value.

For instance, most **inkle** stories include the following:

	=== function alter(ref x, k) ===
		~ x = x + k

Lines such as:

	~ gold = gold + 7
	~ health = health - 4

then become:

	~ alter(gold, 7)
	~ alter(health, -4)

which are slightly easier to read, and (more usefully) can be done inline for maximum compactness.

	*	I ate a biscuit[] and felt refreshed. {alter(health, 2)}
	* 	I gave a biscuit to Monsieur Fogg[] and he wolfed it down most undecorously. {alter(foggs_health, 1)}
	-	<> Then we continued on our way.

Wrapping up simple operations in function can also provide a simple place to put debugging information, if required.




##  6) Constants


### Global Constants

Interactive stories often rely on state machines, tracking what stage some higher level process has reached. There are lots of ways to do this, but the most conveninent is to use constants.

Sometimes, it's convenient to define constants to be strings, so you can print them out, for gameplay or debugging purposes.

	CONST HASTINGS = "Hastings"
	CONST POIROT = "Poirot"
	CONST JAPP = "Japp"
	
	VAR current_chief_suspect = HASTINGS
	
	=== review_evidence ===
		{ found_japps_bloodied_glove:
			~ current_chief_suspect = POIROT
		}
		Current Suspect: {current_chief_suspect}

Sometimes giving them values is useful:

	CONST PI = 3.14
	CONST VALUE_OF_TEN_POUND_NOTE = 10

And sometimes the numbers are useful in other ways:

	CONST LOBBY = 1
	CONST STAIRCASE = 2
	CONST HALLWAY = 3
	
	CONST HELD_BY_AGENT = -1
	
	VAR secret_agent_location = LOBBY
	VAR suitcase_location = HALLWAY
	
	=== report_progress ===
	{  secret_agent_location == suitcase_location:
		The secret agent grabs the suitcase!
		~ suitcase_location = HELD_BY_AGENT
	
	-  secret_agent_location < suitcase_location:
		The secret agent moves forward.
		~ secret_agent_location++
	}

Constants are simply a way to allow you to give story states easy-to-understand names.

## 7) Advanced: Game-side logic

There are two core ways to provide game hooks in the **ink** engine. External function declarations in ink allow you to directly call C# functions in the game, and variable observers are callbacks that are fired in the game when ink variables are modified. Both of these are described in [Running your ink](RunningYourInk.md).