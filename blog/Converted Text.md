<!--StartFragment-->

**Introducing LASK.AI: Your Coding Partner in Crime**

Imagine this: it’s crunch time, and your code is starting to look more tangled than a Fast & Furious plot. You’re scrolling through lines and lines of code, looking for that one tiny error that’s messing everything up—like hunting for a Horcrux in the Harry Potter universe.


![File Path](../assets/ai_code_assistant/horcrus.jpg)


Wouldn’t it be amazing if you had a sidekick that understood your project, spotted what you missed, and fixed issues faster than Hulk smashes a brick wall? Enter LASK.AI: the ultimate sidekick for developers, built to understand your codebase like a buddy who knows all the inside jokes.

Imagine LASK.AI as that reliable friend who doesn’t just hand you random lines of code but actually sees the big picture—like Captain America planning the Avengers' moves. It’s not just code generation; it’s code that’s smart, context-aware, and feels like it’s tailor-made for your project


### **A Deep Dive into the Development Process**

Now, we didn’t just sit down one day and whip up an assistant this sophisticated. Nope, we went through some serious research and analysis first.


#### **Analyzing the Market: Where Other Tools Fell Short**

![File Path](../assets/ai_code_assistant/HTML-and-CSS-Jokes.jpg)


We explored tools like Codeium, GitHub Copilot, Amazon Q, Sourcegraph Cody, Tabnine, and Cursor. Each has strengths—code suggestio
ns, error-fixing, and debugging capabilities are fantastic across the board. However, one common shortcoming stood out: generic, context-lacking code. For example, suggestions were often isolated, oblivious to the bigger picture of the codebase. We realized LASK.AI could make a huge difference here by focusing on contextual, project-specific code generation that resonates with the project’s architecture and dependencies.


#### **Uncovering Developers' Real Needs**

In discussions with our dev team at Valuebound, we honed in on essential features that would transform LASK.AI from "another assistant" into a game-changer:

- Contextual Code Generation: Generating code with a focus on the current file and related files to ensure relevance.

- Repository-Based Context Extraction: For devs working with extensive codebases, this was a must-have—understanding an entire repo’s context is invaluable for accurate assistance.

- Cross-File Awareness: Helping devs find related functions or variables in other files makes the AI smarter about the big picture.

- Automated Documentation & Selective Code Fixes: Automating documentation was a recurring request—more time for coding, less for documentation!

- Automated Git Commit Message Generation: Because let's be honest, commit messages get repetitive fast.

Armed with these insights, we mapped out features for LASK.AI that directly address the real struggles developers face with existing tools.


### **Evolution of Idea 😂:**
![File Path](../assets/ai_code_assistant/evolution%20meme.jpeg)


Our initial concept was to build an AI coding assistant that could capture a project’s full context—helping developers generate code, solve common issues, and even automate routine tasks like documentation. One of the most compelling use cases we uncovered? Integrating context from GitHub or GitLab repositories. This led us to focus on a VS Code extension, given VS Code’s immense popularity among developers in every field—whether front-end, back-end, full-stack, or data science.

Picture this: you’re a frontend developer, clueless about the backend setup, but suddenly you need a reference. No more hours spent scanning code files! Just paste in the repository link, and LASK.AI will break down the files and dependencies for you. That’s the vision. It’s more than a code generator; LASK.AI offers intelligent code suggestions, fixes errors, writes comments, and even chats with you about optimization ideas. It bridges gaps in understanding and spares you from time-consuming searches, empowering you to code faster and smarter.


### **A Three-Week Sprint to LASK.AI MVP**

With a tight three-week timeline, we knew we had to prioritize. Here’s how we approached the sprint:

1. Contextual Code Generation: Top priority for generating relevant code suggestions based on the current file.

2. Repository-Based Context Retrieval: Integrating repo context for suggestions aligned with real-world code references.

3. Cross-File Function Fetching: To ensure LASK.AI could make suggestions that bridge dependencies across files.

4. Automated Documentation & Basic Fixes: Adding auto-generated comments and basic code fixes for enhanced code clarity.

5. Code Health Checks: For company-specific standards, ensuring consistency across the codebase.

6. Automated Commit Messages: A lifesaver for maintaining coherent commit histories.

With our features prioritized, we got down to the business of building.


### **Choosing the Tech Stack: A Bit of Trial, Lots of Error**

Not gonna lie; we had no clue how to build a VS Code extension at first. We decided on a tech stack that balanced simplicity and scalability:

- Extension Frontend: We chose TypeScript with VS Code API for easy webview interactions.

- Backend: Python’s FastAPI won out for its library support and developer ease.

- AI Framework: Initially, we went with LangChain, though we explored HuggingFace and Google’s generative AI options.

- Model Selection: For high-speed responses, we turned to Groq’s Fast Inference API, eventually settling on gemma-7b-it for development. Realizing the context length of the model is extremely important for this use case, we finally decided on Gemini flash 1.5.

- Embedding Selection: We researched on what is a good embedding method to embed repository code files. We asked chatGPT, searched with perplexity, and decided to move on with the Graph Code BERT model for embeddings. 



**Backend Development**

**Embedding Generation:**

1. **Whole repository at once:** We first implemented a process where the whole repository is zipped and sent to the backend. File changes are monitored with a chokidar library and sent to the backend using websocket. And embeddings will be generated and updated by looking at changes at the backend copy of the repository. This approach had several problems:-

i) Time taken to zip and upload the repository.

ii) CPU usage in tracking file changes in both frontend and backend.

iii) Embeddings were getting updated in every keystroke, updating embeddings used extra CPU resources.

2. **Gradually generating Embedding:** To solve these problems, we started generating embeddings in a gradual manner. First the file currently opened in the editor will be the first file whose embedding will be generated. And embedding will be updated when the user changes the focus from the file. Instead of using chokidar, we were now using vs code api to track file changes. This was good but generating embedding was still taking a lot of time. 




**The ohhhhhhhhhhhhhhh moment:**

We realized that at the backend, we were not utilizing gpu to generate embeddings. This slowed the process by a lot. After making a few updates in the code we now had an embedding generation process that was 10x faster than previous versions.

\
![](../assets/ai_code_assistant/embedding%20previous.png)

**Previous Code:**

File Size: 350mb

**Time: 10 hours**

![](../assets/ai_code_assistant/embedding_new.png)

**Updated Code:**


File SIze: 350 mb

**Time: 30-45 mins**

**Synchronizing Embeddings:**

**Single Thread:**  While initially the backend was a single threaded application we had to make changes. The problem with single thread was that when embeddings were being generated and synchronization was in process, the backend did not reply to other api calls as long as previous processes were not complete. To solve this we created a thread pool.

**Multiple Thread:** The multi-threaded approach allowed us to break down the workload, where each task, such as embedding generation or synchronization, was assigned to individual threads within the pool. This setup allowed the backend to continue handling other requests while processing tasks in the background, ensuring the system remained responsive.

Problem: CPU resource utilization to manage multiple threads.

**Single Tread with semaphores:** The code uses asyncio.Semaphore to control concurrent access to certain tasks (e.g., embedding generation with embedding\_semaphore and syncing with syncing\_semaphore). This is done asynchronously to limit the number of concurrent operations without using threads. 

BackgroundTasks: In FastAPI, BackgroundTasks is used to run certain operations in the background, which can simulate concurrency but is still managed within the FastAPI asynchronous loop rather than being distributed across multiple CPU threads.

\
![](../assets/ai_code_assistant/semaphoreEmbedding.png)\
\
\

**Context Retrieval:**

To test Context retrieval, we created a stream lit application to try out different context retrieval techniques.

- **Approach 1st: Simple RAG:** Here we used Graph Code Bert to generate embeddings, large files were broken into tokenized chunks, mapped these chunks and embeddings using faiss index. This approach worked for 3-4 files but beyond that, we could not get relevant chunks from the files.

- **Approach 2nd: Hybrid Search:** Here we get relevant chunks from indexes by performing cosine similarity, then get relevant chunks by performing a search on the text chunks and taking the union of both results. 

It is important to make sure that dimensions of the embedding match the faiss index.

Even though this improved the search performance, we could not get the relevant chunks from larger repositories.

The results of hybrid search were not enough as context.

![](../assets/ai_code_assistant/hybrid.png)

- **Approach 3rd Raptor:** After exploring the web for better options, we found a library called raptor. RAPTOR introduces a novel approach to retrieval-augmented language models by constructing a recursive tree structure from documents. This allows for more efficient and context-aware information retrieval across large texts, addressing common limitations in traditional language models. However this couldn't be implemented.

Problem: Needed Open AI key or used sentence transformer. Open AI embedding would incur regular token cost and sentence transformers needed better hardware to generate embedding of the whole repository.

- **Approach 4th**: **Hybrid Search, current file content:** In this approach , we passed the current file content as text. Used hybrid search to get relevant chunks. Even though this approach didn’t fetch the relevant chunks, it still did the work. At this point we had a usable solution. 

**Remote Context:** 

Most of the open source repositories were present in github but most companies use gitlab to store their private repositories. There were two ways to crawl the repositories and store the content.

 Either we clone the whole repository at backend and select files which we want to use, this seemed like a non ideal solution as we needed only the code files. Cloning the whole repository would take extra unnecessary space.

Another Approach is to store the content in a single json file, then use this to select content of specific file from the repository. This was still not the ideal solution but it was good enough.

\


**VS code Extension:**

**Flow:**

**Design 1:** Initially, the extension opened with a welcome screen featuring three tabs: _Chat Panel_, _Context Page_, and _Show Context Page_. Upon opening the extension, the welcome screen appeared, and the user had an input area to start a conversation right away. To add context, there was a separate _Context Page_ where users could add different context types like: an entire directory, single files, GitHub/GitLab repository links, code blocks, and technical documents. The _Show Context_ tab displayed all added contexts in one place, with a remove button for each.

**Design 2:** We refined the interface with a more practical UI by adding a _Project Initialization_ page. Here, the user entered their name, project name, and selected a code workspace from a dropdown before hitting the "Start" button. After this setup, they accessed the _Chat Panel_ and _Context Page_, with all added contexts shown on the same screen.

**Design 3:** To streamline onboarding, we removed the _Project Initialization_ page, showing only the welcome screen with an input area, a send button, and a "Remote Context" button for adding GitHub/GitLab repository links and access tokens. A _User Guide_ button was also added, redirecting users to the guide on the website.

**Chat Panel:**&#x20;

**Design 1**: The initial design included a chat view, a context view, an input text box, a send button, and a "Generate Code" button. The _Generate Code_ button provided only the required code based on the input, while the _Send_ button gave a general LLM response, which could include code and descriptions.

**Problem:** Users often needed to manually copy code snippets generated from the _Send_ button. Having both buttons created redundancy in the chat panel.

**Design 2:** We simplified the design by removing the _Generate Code_ button. Now, any code snippets in the LLM output come with _Copy_ and _Insert_ buttons for easy access.

****![](../assets/ai_code_assistant/newextension.png)****

**Code Generation in Editor:** (Screen shot)

We added an NLP-based code generation interface where users can enter a prompt and generate code directly in the editor. Users can accept the generated code by pressing "Alt + Enter."

![](../assets/ai_code_assistant/nlp.png)’

**Shortcut Keys**: 

To avoid conflicts with VS Code's existing shortcuts, we chose the following key mappings:

- **Alt + B**: Open code generation textbox

- **Alt + Enter**: Accept generated code

- **Alt + C**: Generate comments

- **Alt + F**: Fix code

**Color Scheme:** 

Since this is a VS Code extension, we ensured it adapts to VS Code's theme. We used theme-based color variables throughout the extension for a cohesive look.

****![](../assets/ai_code_assistant/codegen.png)****



<!--EndFragment-->
