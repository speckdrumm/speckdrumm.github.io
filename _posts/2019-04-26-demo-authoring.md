---
title: "Demo authoring: now and then"
---

Hi I'm [Paralax](https://twitter.com/paralax_sd). I've been doing stuff for the
[demoscene](https://en.wikipedia.org/wiki/Demoscene) since around 2001. Recently 
I've been involved in a demo project that (surprisingly enough) won us the 
PC Demo competition at [Revision 2019](https://www.pouet.net/prod.php?which=81078):
[sp04 - Airplane? by Spacepigs](https://www.pouet.net/prod.php?which=81078).
I didn't see it coming but in any case: I'm grateful and more than hyped that 
this has happened. Hats off to all competitors of this party, so many releases 
were simply stunning and I had an awesome time!

I've mostly recovered from the party and so I thought it would be as good a moment
as any to write something about the portion of the project I've been directly 
responsible for: the tool support.

I'd like to structure this article into three parts. First I'd like to describe
the principal challenges I see in demo making, how these challenges have 
influenced the set of tools I've written over the years and conclude by describing
the latest tool I've made, the one used for our Easter release: the speckdrumm bart tool.


Table of Contents:
1. [Demo making Challenges](#demo-making-challenges)
2. My past approaches to demo making
	1. [The naive approach](#the-naive-approach)
	2. [Demo Scripting](#demo-scripting)
	3. [Hennecke.Net](#henneckenet)
3. [Bart Tool](#bart-tool)

## Demo making challenges

Let's condense demomaking down to the bare essentials and go from there. I think any demo
can be described as a piece of software that modulates a set of parameters over time. These
parameters are tied to some kind of artistic vision and drive whatever specific technologies
are employed for realizing said vision. It's the job of a democoder to author all required
base technologies and come up with specific code to realize the desired output.

Given this premise it seems to me that one goal for efficient demo making must be to support
an authoring workflow where outcomes of parameter changes are instantly visible. Authors
spending less time waiting can get more done and will end up with more refined results.
I see my strenghts in providing support for collaborators and
working with large code bases made to realize this support. This approach has informed my past
strategies to demo making. I'd like to go back to the beginning and review how this has turned
out until now.

## The naive approach

Back in 2001, when I first started thinking about working on a release my first approach
was of course the most straightforward: create a new C++ project, initialize DirectX,
put in some kind of visuals, play with numbers in the code and evolve from there.
Obviously this had some major drawbacks. Every time any number is changed the executable has to
be recompiled. Afterwards the entire demo has to be reloaded. Most time is spent waiting
and little time is spent on tweaking. Clearly a more efficient way of authoring content was needed.

## Demo Scripting

My first approach to solving this problem was pretty much copied from the 
[Farbrausch](http://www.farbrausch.de/) release [Fr09 - Goldrausch](https://www.pouet.net/prod.php?which=2903).
This release came with a simple script that loaded all necessary resources, 
arranged cameras, animated parameters etc. Inspired by this approach I made up a 
similar system that supported very specific text commands - a kind of demo [DSL](https://en.wikipedia.org/wiki/Domain-specific_language)
if you will.

Input text was tokenized, converted into a very simple byte code representation 
and executed prior to demo execution. I've also added an in-demo text editor that allowed
me to tweak the script while the demo was already running. Doing this essentially cut the
reloading penalty I've had to deal with earlier. My first demo I worked on already comes with 
this kind of system: [Speckulation](https://www.pouet.net/prod.php?which=7006) by Speckdrumm,
released at [Underground Conference 2002](https://www.pouet.net/party.php?which=46&when=2002).
In a way it's a funny coincidence that 17 year later we'd win the PC demo competition with an 
invitation for the latest revision of this very demo party ;).

We worked with this scripting system for a while but by 2003 it became evident that
this approach was holding us back. For once, this way of working was too coder centric and
that means that our artists didn't want to author demos using this system. They were
accustomed to tools like 3D Studio Max and Photoshop and wanted something similar.

## Hennecke.Net

We needed a real demo tool. After releasing [Speckdrumm vs. Ferdi Tayfur](https://www.pouet.net/prod.php?which=10733)
I set out to do just that. I came up with a tool called 'Hennecke.Net', half-seriously named after
the East German miner [Adolf Hennecke](https://en.wikipedia.org/wiki/Adolf_Hennecke), famous for 
overfullfilling his work quota by 387%.

Hennecke.Net is made up of two parts: there's the tool user interface that allows an artist to
create content and the engine executable that does the actual rendering. The engine is automatically 
started by the demotool and made to render into a window provided by Hennecke.
Only the engine executable will be part of any release. The demo tool and the engine executable
communicate via TCP/IP using a simple custom text based protocol.

The engine executable is set up to provide an array of 'entity types' to the demo tool. These
can be instantiated inside the demo tool and will run any kind of code inside the player process.
Entities can also  have properties that can be animated by binding them to controllers and 
possess optional input and output pins in order to share data with other entities.

![Early Hennecke shot](/assets/hennecke01.png)
*A very early version of Hennecke.Net, Nov. 2003*

The resulting DAGs made up of connected entities can then be arranged on a timeline. The tool
UI also comes with a dedicated curve editor. This editor sports certain keyframe editing 
capabilities that are dependent on the type of parameter that's animated. Animated floating
point numbers are visualized as one dimensional curves, colors are split into their four
components, enums are blocks on a grid and so on.

The demotool is also able to resynthesize the commands necessary to restore the demo state.
This is needed to export the demo into a standalone executable. An exported demo is nothing
else but the player executable merged with an archive containing all necessary assets.
This asset is expected to contain a specific text file containing the commands to load the demo.
A player executable that's not started with special commandline arguments will first try to 
find this script and launch itself that way.

![Curve Editor](/assets/hennecke02.jpg)
*A surviving shot of the curve editor, October 2008*

Later on we also had support for multiple timelines which allowed us to nest timelines and 
use render to texture in order to embed composited output in parts of another scene.
We used this functionality to render the fake TV spots onto the TV screen of our 2008 winnig
entry [Zapdrumm](https://www.pouet.net/prod.php?which=52359).

![The Nonstop Schlachthaus Experience](/assets/hennecke03.jpg)
*A final shot of the tool showing [The Nonstop Schlachthaus Experience](https://www.pouet.net/prod.php?which=16354), June 2005*

In total we used Hennecke for ten PC demos. By 2009 however it became clear that the way the
tool worked was no longer up for the task. For once the underlying architecture of the tool
wasn't in great shape. It was hard to extend the demotool because parts of the logic and 
the UI were intimately coupled. It's been six year since I started this project and by then
I knew how to do better. 

Another problem that became evident was that it was really hard to
modify specific parts of a scene. We had entities that could load scene files that were 
exported into our own scene format. These entities provided access to cameras and lights but
It was hard to attach new nodes to the scene graph. Furthermore, while the tool realized a
pretty usable WYSIWYG experience, any new entity property that was needed meant going back 
into Visual Studio, extend and test the new entity behavior and share a new version of the 
player. 

Wouldn't it be great if entity authoring could be done right in the environment? 
Before starting a new demo tool we wanted to work on a game entry for which we also needed
some kind of tooling. This new project for a simple barber game was fittingly called 'Bart Tool'
(German for beard). We never realized any part of the game but instead this was the stepping
stone for the demo tool we're still using today.

# Bart Tool

yada!
