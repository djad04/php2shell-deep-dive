# The Story Behind the Research

Honestly, I was getting tired of finding explanations on the web that only skimmed the surface without actually explaining what was happening internally. I wanted to make something different: a real, in-depth explanation of the entire process, going through every little detail and explaining not just WHAT happens, but WHY and HOW it happens.

I also wanted to make all of this as accessible as possible without sacrificing technical depth. The goal was to make the entire process understandable and fun to read, even if you have absolutely no experience with low-level memory exploitation. Some basic C and general programming knowledge should be enough to follow along.

I also wanted to share information that, at least from everything I have found so far, is barely available online in this level of detail. Honestly, I haven't come across a resource that brings all these pieces together and explains them this deeply.

## Now, a little story behind it.

When the PHP vulnerabilities were first announced, I spent a HUGE amount of time trying to figure out the logic behind them.

As many others who tried investigating them probably noticed, there was barely any technical information available at the time. We had a few basic hints: something related to `PHP_SESSION_UPLOAD_PROGRESS`, some kind of Out of Memory condition, and, of course, that video Calif published showing a dancing girl with a shell running the exploit in the background. 

I found that pretty funny, especially considering the scale of the vulnerability and how many servers around the world could potentially be affected. Anyway, that was pretty much all we had to work with!

Even AI assistants weren't helpful at all. At the time, they were heavily restricted when it came to this subject. A simple technical question about an internal PHP function could get flagged just because you mentioned PHP and something technical in the same sentence. For those of us without access to a stronger cybersecurity-focused model, even asking basic questions and trying to investigate things with AI was practically impossible.

So I started digging into the PHP codebase myself (I was honestly getting a little mentally insane by that point).

I spent long days trying to understand everything happening around `PHP_SESSION_UPLOAD_PROGRESS`, how `zend_bailout` works, how the Out of Memory condition is reached, and what happens internally when it gets triggered.

Eventually, I managed to understand how the problem actually works and how the vulnerability is triggered. But understanding the vulnerability was only one part of the problem. Turning that understanding into a working exploit required EXTREMELY precise memory manipulation.

Then came October 25, when the Steal My Bitcoin challenge went live.

I was among those trying to solve it. I spent quite some time studying the memory layout with Ghidra, trying to understand how to achieve the required memory placement and connect all the pieces, but honestly, I was getting lost.

Eventually, I came across someone in the challenge discussion who had access to a stronger cybersecurity-focused AI model with fewer restrictions: ChatGPT Daybreak.

And here's something worth mentioning: even with access to that model, he hadn't managed to make any meaningful progress on the challenge before we started talking.

We exchanged ideas, and I shared all the notes I had been keeping during my investigation: the PHP code paths, where exactly the Out of Memory condition was triggered, what happened afterward, how the internal mechanisms worked, and everything else I had managed to understand.

I then suggested asking the model to try forging a `zend_object` and making it land EXACTLY where the stale pointer was expected to point in memory.

My idea was that the handlers table inside that forged object could potentially lead to the execution of `free_obj` or another freeing-related handler, ultimately giving us code execution.

After providing all that context and the direction I had in mind, the model came up with surprisingly precise memory calibrations and a layout that placed the forged object exactly where we wanted it.

But there was a problem: NO CODE EXECUTION AT ALL.

The object was landing in the right place, but forging a complete Zend object turned out to be much more complicated than I had initially expected. It required a valid `zend_class_entry` pointer (ce) and several other internal structures and pointers that would be extremely difficult to leak and arrange correctly.

So the approach itself wasn't giving us what we wanted.

By the time we were investigating this further, we discovered that the challenge had been solved by KillaBoi. Congratulations to him!

When he published his exploit, I finally had the opportunity to compare the approaches. And that's when things became even more interesting.

The memory calibrations and the layout were essentially correct, but the crucial difference was that he was forging a Zend array and leveraging its `pDestructor` callback, rather than forging a Zend object.

And that's something I had completely missed! I didn't even know that a Zend array's underlying HashTable had a destructor callback that could be leveraged in this way. I found that particularly interesting.

I kept my investigation going, diving even deeper into the memory layout and trying to understand WHY those exact calibrations worked.

Why did the exploit need those specific allocations? Why did it need that exact PHP build? Why would changing just a few things completely break the memory layout? How were all these pieces connected to the vulnerable PHP code?

I kept looking at the information available online, but most explanations were still just scratching the surface. I could see exploits made for very specific PHP builds, precise memory allocations, and carefully chosen offsets, but nobody seemed to be explaining the reasoning behind all of that.

Why does it need those exact memory calibrations? WHY? HOW? How do we actually arrive at those values? What is happening in memory at every step?

I wanted answers to all of those questions, not just a working exploit.

So I started writing everything down.

After another few extremely intense days, on October 29, I finally finished the first version of the document.

I brought together everything I had learned and understood: from the PHP internals and the vulnerability itself, to the memory manipulation, the detailed memory snapshots I captured during my investigations, and the exploit code.

But that wasn't the end. I then spent even more days trying to turn all those notes, explanations, and memory dumps into a clean, well organized document that people could actually read and follow, even if they had absolutely no prior knowledge of memory exploitation.

And here it is.

## Acknowledgements

A HUGE thanks to:
- **James Pavur (@pavja2)**, for creating the Steal My Bitcoin challenge and giving us the opportunity to investigate all of this.
- **KillaBoi**, for publishing his exploit and making it possible to study and compare the final approach. Congratulations again!
- **The person I exchanged ideas with** during the challenge, for the discussions and collaboration.

And finally, thank you to everyone who takes the time to read this document.

I can finally get some sleep and recover.