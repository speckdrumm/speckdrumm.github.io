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

During the time when we still thought about doing a game entry, I worked on a [Visual Studio](https://visualstudio.microsoft.com/)
extension for my master's degree in software engineering. With time I familiarized myself
with the APIs Visual Studio provides and learned how the architecture fit together. 
I became more and more convinced that it would be fun to duplicate this general approach
and end up with an extensible shell that could be used for many different scenarios.
Going this route was probably the boldest violation of the [KISS](https://en.wikipedia.org/wiki/KISS_principle) 
principle I ever dared to engage in ;). It obviously wasn't an efficient way to continue making demos.
It didn't matter to me though since this was a hobby project and I did this just for fun and
because I could.

## Bart Basics

Bart by itself is an application that - very much like Visual Studio - comes as a generic extensible
shell. It is again written in C# and the core UI is based on Windows Forms.
By default it can manage solutions and projects that may be part of a solution, display properties,
show tabbed log output and provide base services such as the ability to open a project item using an
editor.

When Bart starts it searches the file system for .dll files in its assemblies subdirectory and
uses reflection to test if any such assembly provides a so called 'package' implementation.
Packages are extension points into the application and are able to add various kinds of functionality
to the demo tool.

A package may contain the following components:
* Properties - Serializable values that may influence the behavior of the package. Accessible to the user
via the options menu and comparable to Visual Studio's Tools->Options dialog.
* Editor Factories - A set of factories that provide new editors to the tool.
* Project Factories - A set of factories that provide new projects to the tool. Any project may influence
the set of available project items, tool windows and come with extra context information.
* Project Item Factories - A set of factories that support addition of a specific kind of item to a project.
* Property Designer Factories - These components add new ways to visualize values in the built-in property grid.
* Context Menu Commands - These commands can extend the application context menu or the solution explorer.
* System Services - An addition to Bart's internal service system. Services can be retrieved by interface
and are automatically integrated with the built in dependency injection mechanism to support multiple schemes
of automatic service injection. *Bart does not use singletons*.
* Named Pipe handlers - These are extensions to the generic named pipe server that comes built in. As it is
today we use named pipes to speed up our scene importing workflow (one-click import) and push C# compilation
requests from generated Visual Studio projects containing demos scripts.
* Custom init code - on initialization a package is handed an interface that provides it with full access to
any service it may desire. Our current demo system uses this approach to create a new tool window and prepare
rendering.

![Package](/assets/bart_package.png){: .center-image }
*Common extension points any Bart package may provide*

Bart initializes all packages it finds. It also comes with an array of core services and of course various 
UI components in order to facilitate ineraction with the user.

Core services are as follows:
* Solution manager - Provides API access for the one solution the tool can manage at every point in time.
* Shell - Various APIs for influencing docking, layout, access to the main window.
* Editor handling - Editor Factory registration, editor state management, various editor specific events.
* Named pipe server - Provides registration APIs for synchronous and asynchronous named pipe handlers.
* File Watcher - Monitors all project directories and provides file change events and APIs to determine when 
files have last been written to.
* IOC - provides access to tool services and allows registration of new services.
* Focus handling - Keeps track of the active form and gives views the opportunity to handle user input
starting with the most concrete child control and going up the ownership chain until arriving at the tool main
window.
* Log Output - Provides services similar to Visual Studio's output window.
* Packages - Supports the package management infrastructure and provides APIs for pacakge property (de)serialization.
* Project Types - Provides API access for project- and project item factory management and retrieval.
* Selection - A service that keeps track of all entities that are selectable in the current usage context
and which entities are currently selected. Selections are reflected by the designers shown in the property grid.
* Status Bar - Basic APIs for controlling the contents and color of the status bar.




![Bart](/assets/bart_tool.png){: .center-image }
![Stubble](/assets/stubble_package.png){: .center-image }