# Getting Started - Preparing Our Resources

## Programs Required:
* Cheat Engine<br>
Cheat Engine is a program that scans and analyses the memory of a given program (including itself). We'll be using it to find pointers, addresses and offsets. It is integral to this tutorial, and is the only
program I would describe as being "non-negotiable". Later in the tutorial, we'll be writing a function that emulates Cheat Engine's behaviour, albiet, in an incredibly simplified fashion.<br>
There's a repo for it here on GitHub, alternatively you can download the installer from the website. Now I feel as though this should be a litmus test of some kind - being able to install Cheat Engine without
downloading 16 other programs with it - but I cannot in good conscience tell you to download it without warning you: <br><br>
**Cheat Engine will attempt to install bloatware on your system. In order to avoid this, actually read what the installer tells you.**<br>
* A Code Editor<br>
I use Visual Studio Code, but Visual Studio is also a viable option. This tutorial won't use any weird libraries or other dependencies, so the build process will be super simple.<br>
For those using a text editor, not an IDE, you'll need a compiler. You'll want to get MinGW-x64 for Windows to faciliate this, Microsoft themselves (I know crazy), provide a good tutorial for getting this to
work on Visual Studio Code: https://code.visualstudio.com/docs/languages/cpp.<br>
Whilst the tutorial is very well written, it has many relatively complicated steps in quick succession, so it's very easy to make a mistake.
I'd suggest making sure "Hello World!" works before moving forward, then that's the build process completely covered.<br>
Don't forget to forget to edit your PATH and abuse ChatGPT as to why CMD won't run G++.

## Resources:
* My KOTOR Trainer<br>
Only 40 lines in and I've sold out. I've already made a KOTOR trainer, its open source and here on GitHub, so you're free to read it, use it and steal its intellectual property. I'd recommend using it as a guide
if you get lost during the tutorial. <br> I apologise in advance for my **cursed** bastardisation of Hungarian notation - I do it to game the intellisense I swear. <br>
* Game Specific Resources<br>
What we're doing here is far from new ground, check out modding/speedrunning circles and see what information has already been mined from the game you're working on. I'd suggest making your trainer, then leveraging
information already disseminated the wider community.
* Windows API Index<br>
If you thought C++ was verbose and poorly designed, then dear Lord above do I have something for you: The Windows API. Whilst I have mountains of respect for the Windows programmers, their API is hell to use.
Making a window for Windows in C++ makes me want to jump out a window.<br>
Fortunately, they made a weapon of a website to explain how the API works: https://learn.microsoft.com/en-us/windows/win32/apiindex/windows-api-list <br>
We'll be using the Windows API to read and write to process memory, so our interactions with it will be quite limited in scope. I'll do my best to explain how the functions from the API we'll use work, or are at
least used, but having the documentation at hand is always helpful, particularly when the Stack Overflow community accuses you of not reading it - quote it like the Bible.<br>
That being said, I'm convinced it works in a fashion similar to planes - the moment we stop believing, it'll fall out of the sky.

# Getting Started - Our First "hack"

We'll be spending most of our time in the beginning with Cheat Engine, then C++, and then Cheat Engine until you feel as if your trainer is complete. So let's get a bit of Cheat Engine 101 action happening, and do
our first every memory 'injection'. If you're following along with KOTOR, we'll be changing our experience to 999999, well beyond what you'd expect in game. For everyone else, pick a game with money, or another
easy to manipulate number, that you know the exact value of, you'll be manipulating that.

## Scanning the process memory
The most basic general 'operational loop' of memory analysis in Cheat Engine is memory scanning. Memory scanning involves opening a target module, and searching through every avaliable memory address for a specific
value. This process is repeated over and over, until the correct value, at the correct memory address is located. For some values this takes a few minutes, for others (particularly boolean flags) it can take
much, much longer. Sorry for this, but we're going to talk about memory now.
### How does memory work?
When an executable wants to run, it must polietly ask Windows to section off some memory for it, the executable operates within this space, often refered to as a module by the Windows API. Think of Windows as a 
landlord, and a program as a tenant. The program tells Windows its needs 'x' amount of space, and Windows creates [a] 'room' for it. Cheat Engine simply enters this space, rummages through another program's 
stuff, and tells you what it finds. <br>
After being provided memory by the Windows kernel, programs become their own landlords, and (for the most part) are permitted to organise their own space as they see fit. Imagine the apartment rented by the 
program has millions of rooms that the program is permitted to sublet out to pieces of data. There's tenants that sublet out a room every single time, like video settings, pointers to code, vital flags and, other
pieces of data required for the initialisation of the program. These tenants are put into the same room every single time. Note that much like real accomodation, these pieces of data aren't always ordered one 
after the other, there may be large jumps in room numbers - this doesn't mean that vacant rooms don't exist, it just means that if you looked inside, nobody would be home.<br><br>
More volatile pieces of data have to be allocated dynamically. Think of game like Skyrim. The player has an inventory that may have five items in it, or eighty-five. When the program sublets its apartment, it does
know exactly how much rooms it'll have to rent out for each piece of data, so it just reserves a pre-determined number of rooms and assigns items to each room as you collect them throughout the game.<br><br>
If you're firing on all cylinders, you might ask, if Windows doesn't always put Skyrim in the same place, then how does Skyrim know where to sublet its data?<br><br>
That's a fantastic question, the answer is relatively simple. Much like a regular hotel, the rooms are numbered sequentially, based off the address that Windows gives to the program initally. Let's say Windows
gives a program a room at address 0x3AFC, the program doesn't remember that the first item in your inventory is at 0x3BFC, it just remembers it is always in room 0x100. This way, no matter where the program is 
assigned to in memory, it can always find where it is. <br><br>
**Most simply: Memory locations are relative to where the program is loaded by the OS**
### Value Types
In everyday programming, knowing the size of a data type is only important in the event that Coding Jesus tries to make you feel bad about your educational background on stream. Turns out, that knowing how many
bytes are in an integer is relatively important. If you've done the Cheat Engine tutorial (this was a test and if you didn't, you failed, your parents will be contacted) you'll realise that Cheat Engine works
most effectively when you tell it what 'Value Type' to look for. But before we get too carried away, let's gloss over a century of computer science fundamentals.<br><br>
Data is stored in transistors, they are either in the on position or the off position. This is a binary system, in which 0 is off, and 1 is on. A 0 or a 1 is a bit. Put eight of these together, and you've got a
byte. On the surface this may seem as though we now just know if eight transistors are on or off, but if used as a counting system, we can now represent a number between 0 and 255. C/C++ gang will know this as a
char.<br>
Because anything smaller than a byte is not awfully helpful in modern computing, memory is stored and counted in bytes. In our previous landlord - tentant dichotomy, a byte is equal to one room inside an 
apartment, and depending on how much memory a program requests, it could have gigabytes worth of rooms. This means that a value type can take up multiple rooms, and so the value type we're searching for will 
determine how many rooms we need to look through.<br><br>
Remember that Windows API thing I was talking about? It becomes incredibly important right now. Statically typed enjoyers will already be aware of types like int, float and string, but the Windows API has its own
type system that whilst isn't necessarily MANDATORY, Bill Gates would really rather you used it. 
<div align="center">
  
| Size (Bytes) | Windows API Type | C++ Type |
|:----------:  |:--------:|:---------:|
| One         | BYTE     | char / bool   |
| Two         | WORD     | char16_t / short   |
| Four        | DWORD    | int   |
| Eight       | QWORD    | long long   |

</div>

## Attaching our process

As previously mentioned, Cheat Engine simply enters the memory of another process, and makes itself at home, looking through everything.<br>
Consequently, we must tell Cheat Engine where it needs to look. Here's the process to do this:
1. Start the game you wish to analyse.
2. Start Cheat Engine.
3. Underneath 'File', there's an image of a magnifying glass and a screen, it literally has a flashing light, click it.
4. Select the target process.

At this point, nothing much should happen in Cheat Engine, but hopefully your game doesn't crash. If your game does instantly crash, it generally means there is some kind of built in protection against memory
scanners, and debuggers. This is bad news, and fixing this problem goes beyond the scope of this tutorial. Sorry about that.

### *Actually* Scanning the Memory

Alright enough theory, let's ruin the singleplayer gamic experience. Ensure your Cheat Engine process has successfully attached to your game process. You can confirm this occurred by looking at the top of your
Cheat Engine GUI, you should see a hexadecimal number, followed by a process ID, for example mine says "00004D14-swkotor.exe" - if you're playing a different game you'll get a different title.<br>
Tab into your game, and find a value that is easily identifiable, is *likely* a DWORD (four BYTE value - like a number that can go into the thousands) and can be easily modified, either increased or decreased.
I'm going to focus on experience in my KOTOR game, my character currently has 76,610 experience, so under the 'First Scan' button I'm going to enter that number into the "Value:" box in Cheat Engine. I'm then
going to click "Scan Type" and select "Exact Value" from the drop down list. Exact Value means that I know what the value of the target memory address is, and that when Cheat Engine is searching through the 
subletted rooms, it should check for the 76,610 EXACTLY. Next, I'm going to click on the drop down menu associated with "Value Type" and select "4 Bytes". This signals to Cheat Engine that the target value will
be located across four rooms, also known as four bytes, or four addresses. After this is done I'm going to click "New Scan" above the "Value:" text.<br><br>
The table inside of Cheat Engine should have popped up with a series of addresses and values. In my case, I only got six, which has narrowed my search quite significantly already. Depending on your game, 
experience (in game), or any other number of factors, you may have gotten tens, hundreds, or potentially thousands of potential addresses. Don't stress, this is normal. <br>
At this point, we know that one, or potentially more of these values, are associated with my character's experience, but changing these values randomly has the potential to crash the game, or otherwise corrupt
my save. I want to narrow down these search results to avoid unnecessarychanges to my game, further, we'll need the exact value when I write my trainer, so there's no point in rushing now.<br><br>
I'm going to tab back into KOTOR and change my character's experience. I could do anything to make this occur, but because I just so happen to be on Dantooine (objectively worst planet), I'll kill a Kath Hound.
He gave my character 75 experience points, making my experience 76,685. I'll type this number in the same location as I did 76,610 and select "Next Scan" instead of "New Scan".<br><br>
Selecting "Next Scan" will cause Cheat Engine to only scan addresses from the previous scan and exclude all others, this helps us narrow our search and not introduce new addresses.<br>
This time around I got four values, but the "Previous" and "First" columns have changed, and now some text is red? The "First" column simply records what the value of the address was when we initially scanned,
the "Previous" column tells us what the value was on the LAST scan, and as one could imagine "Value" is the current value. If a value changes, it will go red, until it is updated with a scan, and becomes equal to
the current value. In fact, if you do more to change your value, you'll see it change inside of cheat engine. This is a very, very good sign that you've located the correct offset.<br><br>
Four values is pretty low, and I'm quite confident that all of these values are associated with my character's experience, one is likely the text in the character screen, another is my actual experience and the
others could be anything. I'm going to double click these values, which will save them to my memory table, here I can give them names, and most excitingly, edit their values. I double clicked the "value" of the
first address in my table (which just so happened to be 07E6CED0, but yours will definitely not be the same), and changed it to 999999, and I noticed in the table, a bunch of other values changed, and went red in
the search table. Checking in game confirmed my suspicions, my character's XP is 999999, and I've instantly hit max level. If this your first time, congratulations, you're performed you very first hack.

#### Didn't Work? Common Issues:
* You can scan as many times as you like, I was lucky to get it on my second, it can take, many, many scans.
* I know that experience is stored as a DWORD in KOTOR, if you can't find your value AT ALL, try changing types. For instance, a WORD, or a BYTE (if the value is always below 255).
* Some games actively circumvent this method of cheating, this method is *incredibly* simple, and is easy to protect against.
* User error: There's a chance you clicked the wrong button, or entered the wrong number. This isn't a dig, everyone does it.
* Try a new game, or a new value.

### Side Bar: Hexadecimal

I did want to mention this before, but I could tell you were getting bored - now that we've actually achieved something, I'm going to get boring again but this is super important. You've likely heard of, or seen
hexadecimal, it looks like this 0x45 or like this 0xF. Hexadecimal is a base 16 counting system, this means that 0x10 is 16, because each place value is 16, not 10, like in decimal. Obviously, wondering 
how we could achieve this makes sense, if 0x10 is 16, then how do we write 15? I'll show you:
| Hexadecimal | Decimal |
|:----------: |:--------:|
| 0x1         | 1     |
| 0x2         | 2     |
| 0x3         | 3    |
| 0x4       | 4    | 
| 0x5         | 5     |
| 0x6         | 6     |
| 0x7        | 7    |
| 0x8       | 8    | 
| 0x9         | 9     |
| 0xA         | 10     |
| 0xB        | 11    |
| 0xC       | 12    | 
| 0xD         | 13     |
| 0xE         | 14     |
| 0xF       | 15    |

Nobody is expecting you to learn how to count in hexadecimal, but because memory is always represented in hexadecimal, you'll need to get used to using it. Fortunately, the Windows calculator has a setting
called "Programmer", which lets you toggle between decimal and hexadecimal. This is very exciting news because we'll be doing hexadecimal arithmetic to calculate offsets.<br><br>
The 0x prefix to hexadecimal simply indicates that the following value *is* hexadecimal. It is important to note, that Cheat Engine always writes hexadecimal WITHOUT this prefix, so 12 in Cheat Engine is not 
10 + 2, it is 16 + 2. From this point onward, I will always write hexadecimal with the 0x prefix, as this text shifts between decimal and hexadecimal often, further C++ requires us to use the 0x prefix.

# Getting Started - Dude where's my value?

If you've saved your game, your character will permanently have 999999 experience points, however there's a massive caveat that comes with this, that the ambitious among you may have already discovered. If you
load a different save, or quit the game and reloaded the same character in an attempt to show your Mum your sick haxor skills, you'll find Cheat Engine did you a Judas.<br>
This is because we found a dynamic address, or a tenant that doesn't always stay in the same room, KOTOR doesn't always load experience in exactly the same spot every time for every character, for every play
session. Does this mean our trainer is over before it has even started? Of course not, but it won't be as straight forward as your may have initially thought - we're going to have to find a **pointer chain**. A
series of addresses that tell us where we can find experience.

## Pointers & Pointer Chains

### What's a pointer?

It'd be rude to claim a beginner friendly tutorial, just to assume you know what a pointer is, if you do know what and why a pointer is, then feel free to skip this portion. If you don't know the what or why, I'd
highly recommend you stick around, pointers are integral to memory analysis, but are also one the most powerful features in programming and computer science. Even if you don't make a game trainer, if you expect
to ever program in a C style language with any level of competency beyond beginner, you must learn pointers. Yes they are confusing, but they are a staple in computer science.<br>
I'm going to explain pointers in the reverse order to how I've tackled everything else, I'm going to explain the technical code side of them, then provide an analogy. The technical explanation  will make no sense
without the analogy, but the technical explanation  frames the analogy.

#### Pointer Defined

A pointer is a variable that stores the memory address that another variable is currently stored in. They are called pointers because they "point" to other pieces of data, as opposed to being "variables" in of
themselves. Of course, being a variable, pointers can be assigned to new addresses, but we typically don't treat them the same way we would an integer, or string, in that they aren't used to store data, they are
used to track the location of data in memory.<br>
The best way to understand pointers is to create and use one, so let's do precisely this. Pointers are hard, so we're going to do this *very* slowly.

Typically, variables are declared thusly:
```cpp
int x = 5;
```
When this occurs, your program assigns memory on the stack, and places four bytes that represent 5 as an integer. From this point onward, the token *x* will be interpreted as whatever is stored at this position in
memory. We can re-assign *x*, or perform operations on it that change its value, and the location on the stack that represented 5, will now represent something else.
```cpp
x++; // x is now 6
x += 4; // x is now 10
```
Of course, we can create a new variable and assign to the value of *x*.
```cpp
int y = x; // y now equals 10
```
However, if we change *x*, *y* stays the same, and if we change *y*, *x* remains the same. This is because *y* doesn't reference the value on *x* in memory, it is a copy of *x* at the moment of assignment. The
values are not inexorably linked, or really linked at all. You may have encountered this concept when you wrote your first function that takes parameters.
```cpp
void Sum(int x, int y) {
  x += y;
}
int main() {
  int x = 5;
  int y = 2;
  Sum(x, y);
  std::cout << x; // x is still equal to 5, even though we thought that Sum() would alter it.
}
```
This occurs because even though int *x* and int *x* have the same name, they are copies of each other, in the same way that int *x*, *y* are copies of each other. If we want to either change the actual value of 
*x*, or identify where exactly on the stack it is, we'll have to create a pointer. Which is done thusly:
```cpp
int* px = &x; 
```
Note the * operator and & operator. Whilst the * operator is typically used to **dereference** pointers, it is also used to declare them. The & operator **references** variable locations in memory, instead of
representing the variable's *value*, it represents its memory address, which will look like 0x07e6ced0 when piped into std::cout.<br> Now we can manipulate the value of *x* indirectly through its pointer, as *px*
is inexorably linked to *x*, as a consequence of it storing *x*'s memory address. To do this, we need to deference *px* to change it to representing an integer, instead of a memory address, and then assign a
value to it.
```cpp
  int x = 5;
  int* px = &x;
  *px = 2;
  std::cout << x << '\n'; // x is now equal to is 2, not 5
```
Although the deference operator makes *px* act as if it were an integer, not a memory address, *px* still points to the location of *x*, consequently, when *px* acts like an integer, it acts like an integer at the
location of *x*. Consequently, we can deference *px* with the intention of accessing its value - this would be unusual in a main() function, but is common is more complex programs.
```cpp
  int x = 5;
  int* px = &x;
  int y = *px + 2;
  std::cout << y << '\n'; // y is now 7
```
Of course, variables can have many, many pointers assigned to them, but typically, you try to keep pointers to a minimum, particularly these kinds of raw pointers. Despite their power, raw pointers, as shown here
are unwieldy and a common source of bugs, syntax errors and security vulnerabilities - for this exercise this isn't too much of an issue. However, I feel as though I should at least mention that pointers are
as dangerous as they are powerful, lest Low Level Learning come after me for encouraging unsafe programming practices.<br><br>
If you've been paying attention you may have realised something, we're searching for memory addresses, pointers are just memory addresses. Is a game trainer just a program that does this?
```cpp
  DWORD* px = pLocationOfXInGame;
  DWORD NewX = 999999;
  *px = NewX;
```
The answer is yes. (in b4 average reddit user: Yes I am aware of DWORD_PTR, we'll get to that later)<br><br>
Still don't understand pointers? Good. You're following the syllabus as intended.

#### Pointers Illustrated

Remember our tenancy business from earlier? Where our program is given an apartment, which it then sublets to pieces of data, with each room taking up a byte? Imagine, you wanted to look at an room in a (human)
apartment complex, would you expect the landlord to hire a carpentry team to build an exact replica of the room in your front yard, or do you think they'd tell you that the apartment is at 12 GeForce Lane, 
Computerville, and to inspect the property there. Copying a variable by simply assigning it to a new variable (*x* = *y*), is precisely the same as having a builder make you a duplicate room. No matter how much
you change, manipulate, or destroy this duplicate apartment, you'll never change the original, and if the landlord had thousands of clients, building replicas of apartments would become too expensive and time
consuming. Computers are similar, in that they have limited time and resources to complete tasks. Of course, a PC can create a variable much faster than a person could build a house, but computers deal with
millions of variables, eventually, it will run out of memory if we build new apartments each time we want to add two numbers.<br>
This is why pointers exist, instead of building new rooms, we can simply make a real estate posting online, and people can reference this posting as to be able to locate, and attend the actual property. This also
means that if the landlord wants to change an aspect of the room, they don't need to go around and renovate all of the duplicates, they can just manipulate the room itself. Much like driving to a place, a computer
can access memory far faster than it can create it. This is why pointers exist, and are such a powerful tool.

### What's a pointer chain?

It is entirely possible to create a pointer to a pointer. Just as creating a pointer to a value is helpful to manipulate and reference values, sometimes we need to reference and manipulate pointers indirectly. 
Although programmers do this relatively rarely, compilers LOVE creating pointers to pointers that point to pointers. In our landlord example, imagine the concierge for the apartment doesn't actually store the
room that values are staying in in a book or logging system, but writes down the room number on a piece of paper, and then puts that piece of data in a room. Eventually, this system becomes quite convoluted for
humans to follow, but because there is an underlying system that dictates how this system operates, computers are incredibly well equipped to follow these pointer chains.<br><br>
In the context of our use of Cheat Engine, these pointers are offsets, or hexadecimal values that effectively say "you're looking for 'x' value? Last I heard he was 32 rooms ahead of me". When we move 32 rooms
forward, there's a chance we'll land on the actual value, or we'll land on another offset, which may tell us to move another number of rooms forward. Typically, these chains are around 5 offsets long, but can be
longer or shorter depending on the value, and the program. Eventually, the chain will take us to the value we are looking for, and if the chain is stable enough (it is used every play session), we can follow
the offsets to consistently arrive at the room that a given value is stored at, such as experience.<br><br>
In summary, a pointer chain is simply a series of pointers that when followed, eventually land on a value - because this path uses offsets, not constant values, we can follow them across multiple loads of the
game. <br><br>
Our trainer will use these chains to consistently determine dynamic addresses, relieving us of the task of finding them every time the game starts.

## Where pointer chain?

Fortunately, Cheat Engine has a built in method for creating pointer maps, which are then used to perform pointer scans, subsequently outputting prospective pointer chains. We're not going to get into the technical details of pointer maps, nor how Cheat Engine performs pointer scans, as knowing this doesn't facilitate our writing of a trainer. So let's find our first pointer chain!

### Locate the dynamic address

Fortunately, you already know how to do this! If you still have a valid dynamic address from before, then fantastic, we're ready to go. If you've closed the target process or have lost the dynamic address since
our first hack, you'll simply have to find it again. Don't feel bad if you lost the address, and don't feel smug if you still have it. We'll have to find it a few more times either way.

### Generate a pointer map

Make sure you've double clicked your address as to move it from the scan table into the cheat table, give it a new name so you can keep track of it, I'm going to name mine 'Experience 1'. I'm now going to right
click on the entry we just renamed, and select "Generate pointermap". You'll be greeted with a Windows explorer window, prompting you to "Specify the filename for the pointermap you're about to generate". Once
again, I'm going to call it "ExperienceScan1", I'll be referencing it later, so I need to keep track of which address was used to create it. Cheat Engine will work some magic, and show off some stuff only 
nerds care about, we're cooler than Fonzie, so we'll move on. Go back to our cheat table, and right click on 'Experience 1' again - the address. This time select "Pointer scan for this address", its directly
below the generate pointermap option from before. You'll now be presented with a window titled "Pointerscanner scanoptions", this window might feel a little intimidating, but relax, there's only one thing we need
to focus on right now - the "Use saved pointermap" checkbox. Click that badboy, and yet again, you'll be treated to a Windows Explorer window. Navigate to "ExperienceScan1.scandata" that you created before, and
select it. If you're using your whole brain, you'll realise that the address underneath the "Use saved pointermap" box is the same address as your dynamic address - this match is important, because the scandata is
directly associated with this address. After you've loaded the correct scandata file, click OK and Cheat Engine will generate a pointer scan. You can call this whatever you want, we won't really be using it.

### Nice, 100,000 results...

**Unfortunately, we can't narrow the results further, we'll need to test all 100,000 results individually, loading the game up each time.** <br><br>
Just kidding, of course we can narrow it down. After performing my pointer scan I ended up with 69,807  pointer paths ( ͡~ ͜ʖ ͡°), there is no way I can find the correct, stable path, in such a large sample size. 
But what are we even looking at? <br><br>
These are the pointer chains we learnt about before, look at the table, and you'll see each entry starts at a base address, plus an offset. This address then tells us which room to check first, and Offset 0 tells
us how many rooms we should move forward to move toward the final column of "Points to:" which has our dynamic address, and our value. This a direct, visual representation of what a pointer chain is, and is our
theory in practice - bet you're happy you trudged through that theoretical  snoozefest? <br>
Unfortunately, our work is not over, much like the dynamic address from before, these pointer paths are not necessarily going to work on every load of the game, for every character, every single time. We'll need 
to extract a single path that we are confident works every time. <br><br>
I hope you enjoyed generating this pointer scan, because you'll be doing it again! Then again! Then probably again! Then if we're lucky we'll go again. Then lunch. Then again!

### GOTO: CreatePointerScan

I'm being dead serious, we'll be repeating this process over and over until we have a more reasonable number of pointer paths, I typically aim for around 100 or less, and then test all 100, removing them as they
don't work, until I end up with a stable path. There is a slight deviation in the process though, so we'll touch on that right now.<br><br>
Close the game, and restart it. Then re-attach the process to Cheat Engine, and find a new dynamic address for the same value, it should definitely be different. You'll also notice that whatever address you had
for 'Experience 1', or whatever you've called it, now pointers a different value - this is normal. Generate the pointer map as you did before. When you go to generate your pointerscan, make sure you check the
"Use saved pointermap" box, and select your NEW dynamic address' pointer map.<br>
Then make sure you select the box that reads "Compare results with other saved pointermap(s)", it'll prompt you to select a memory address from your cheat table, in addition to another pointer map. Select 
"Experience1scan" and "Experience 1". Not dissimilar to our memory scanning technique, of first scan, then next scan of the same data pool, Cheat Engine will now only find pointer paths present in both pointer
maps. This massively decreases the number of pointer paths that'll be generated. We can compare results with more than one other pointer map, so each time we iterate through the process, we'll add another 
pointermap and address, further constricting the sample size.<br><br>
After a handful of scans, you should have a very small number of pointer paths (less than one hundred).

### Finding the right path for you <3

Its now time to find your final pointer path, fortunately, this is a relatively straightforward affair. Depending on how thorough you want to be, you'll want to select upwards of 20 candidates. They can be 
selected by double clicking on them. This moves the whole pointer chain into your cheat table, it'll look the same as a regular entry, but in the address position will be P->[address], instead of only an address.
The hexadecimal after the P-> is the current dynamic memory address, you'll notice this change as you start and exit your program - this is intended behaviour, it means your pointer chain is dynamically loading
addresses. I like to move as many paths as possible into my cheat table, but here's the traits that make a path a good (or better than average) candidate:
* The pointer path uses the process as a base address ("swkotor.exe"+00434364). This is **very important**.
* The pointer path has few offsets. This increases its chances of stability.

Don't bother being super picky here, just select a bunch and we'll see how we go.<br>
Go back to your cheat table, close the game, reload the same save you were using the do the initial scans. Chances are a fair few of the chains will either point to nothing (??), or a value that is not what you
anticipated. This is fine, remove them from your cheat table and repeat the process. Eventually, you'll find a handful that are incredibly stable and *always* point to the intended value.<br>
This is incredibly good news, now for the final test. Check that these static pointers work across multiple saves, from different characters, in different parts of the game. Eventually, you'll narrow your choices
down to the point that all of your candidates are stable and point to the correct value every single time, without fail. This means that the pointer path is tied to a universal  data structure within the game, 
applicable to all characters, not just an individual one. This will be your final pointer path.<br><br>
Double click on the "address" value (the one that looks like P->[address]). A box will pop up, showing the pointer path. Note that this list goes from the bottom up, its a little bit indie like that. However, the
manner in which it represents the pointer path should make this relatively clear, if you follow its "logic". The base address is at the bottom, and from there we move upward the given hexadecimal values until we
arrive at the final pointer, which equals the value.<br><br>
Mine looked like this:
* Base Address - "swkotor.exe"+003A39FC
* Offset 0 - 0x4<br>
**remember** Cheat Engine does not add the 0x prefix to hexadecimal values, it will read 4 in the table, not 0x4 - but it should be treated as 0x4! (in b4 0x4 = 4)
* Offset 1 - 0x4
* Offset 2 - 0x3C0
* Offset 3 - 0xF8
* Offset 4 - 0xA74
* Offset 5 - 0x68
* Final path = {0x003A39FC, 0x4, 0x4, 0x3C0, 0xF8, 0xA74, 0x68}


If you're following along with the tutorial in KOTOR, and you didn't get the same result, don't freak out, this is a problem with multiple solutions. If you were following along and were unable to find any 
experience pointer path, then I'd recommend you stick with it and find one, rather than just take my one.<br><br>
**It is now safe to tell your Mum to get the camera**<br><br>
This hack will work every time.
  
