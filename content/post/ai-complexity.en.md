+++
title = 'The Paradox of Complexity in LLM Code Development'
date = 2026-02-11T23:11:24+08:00
draft = false

tags = ['LLM']

+++

From the initial Copilot to the current Claude Code, I've tried using these AI-assisted development tools.  
The blogger doesn't write code to make a lot of money; it's more for fun, to be used by many people, and to bring peace of mind.  
And writing code just happens to support me; if not, I would still find another job and continue writing what I consider useful code in my spare time.  
My dislike of AI isn't about code; it's more about feeling a comprehensive impact. Simply put, human value is being diluted.  
However, today we won't discuss this topic. Let's talk about some issues with LLMs replacing humans in writing code.  

Anyone who frequently writes code understands that a core aspect of maintaining a codebase is layered design.  
Complexity is hidden layer by layer, providing clean interfaces to the upper layers.  
As the amount of code gradually increases, most of the time, what's maintained are multi-layered nested interfaces from bottom to top.  
This is just on the application side; there are also architectural layering and complexity hiding on the compiler, standard library, kernel, and hardware sides.  
Within a single layer, one can completely disregard the implementation of the lower layers and only focus on the logic of the current layer. For example, in many high-level managed languages, system interfaces are not accessible by default; one only needs to build upon the interfaces available in the current environment.  
In some cases, it might be necessary to rebuild at a lower level, such as programming for hardware to achieve extreme performance.  
This is actually very similar to context limitations. Humans deal with complex systems by breaking them down to compress "context." In specific situations, this means hiding implementation details within clear interfaces to reduce complexity, which is essentially compressing "context."  

Let's first establish a premise: AI cannot generate the current Linux system and its compilation toolchain all at once.  
Leaving aside accuracy, the point here is to illustrate that its context is limited.  
Let's establish another premise: AI does not possess autonomy.  
That is, it won't ask questions and iterate on its own.  

Human thought is not always necessary.  
The process of thinking can be ignored; LLMs actually only need the result.  
For code architecture design, after extensive iteration, people converge on a nearly perfect optimal design, and this design can be embedded in LLMs.  
For iterating code, people can summarize common chains of thought, and these chains can be embedded in LLMs.  
For breaking down requirements, people can combine past experience to give a rough result, and this experience can be embedded in LLMs in text form.  
Finally, a large amount of open-source code has laid the foundation for AI to generate some "high-quality" code today.  

Returning to the topic of replacing humans, we propose requirements, and AI executes them.  
In development, this means having AI generate code to meet requirements.  
Generating several, dozens, or even thousands of times the amount of code from a very limited language description is asymmetrical.  
This means that the more detailed your requirements, the less room AI has to play, and the more precisely it needs to converge.  
But to achieve this precision, you need to stand at the top of the current layer, look down, and propose the general path and design to implement your requirements.  
Of course, if code precision is not required, and one only cares that the AI-generated code runs with the expected effect, then it seems that this effect is not guaranteed by AI generation. Rather, it's because the underlying framework is good enough to allow existing AI to generate usable code with limited context and prompts.  

After trying AI code generation for a period, it's easy to find that the generated code relies on existing infrastructure, code infrastructure maintained by humans.  
Directly generating common application functions without relying on existing facilities involves a terrifying amount of code, and the code internally is interconnected. If the layering is not done well, a small modification can lead to serious and difficult-to-debug problems. This corresponds to our first premise, which is that AI cannot directly generate a complete set of code from the bottom to the top.  
And when we want AI to directly face a large codebase with no additional code dependencies, this throws the complexity back to the person making the request, forcing the AI user to deal with layered design and some implementation details. Here, we are only considering code-level dependencies, not yet compilers and operating systems.  

Of course, in the long run, layering and complexity issues will be alleviated as context expands and more experience is embedded in LLMs. However, regarding the second assumption, that AI lacks autonomy and cannot self-iterate,  
then humans will still need to deal with complexity.  
Perhaps in the future, the accumulation of experience will reach a singularity, or other factors will be introduced, and LLMs will suddenly gain autonomy and be able to self-iterate.  
Will humans still have value then, in terms of their brains?  

A little extension:  
AI is destined to dilute the value of writing code. Personally, I will still show some respect and tolerance for those who maintain open-source components. If you are using AI-generated code to make money, please do not condemn or belittle open-source components and their maintainers. If there are issues, you can normally raise issues and pull requests.  
