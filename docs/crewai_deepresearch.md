CrewAI – Overview of Agents, Tasks, Tools, and Crews

What is CrewAI?

CrewAI is an open-source Python framework for building autonomous AI agent teams that work together to tackle complex tasks ￼. It was built from scratch (with no LangChain dependency) to be lean and fast ￼. With CrewAI, you can orchestrate multiple AI agents – each with a specific role and goal – into a coordinated crew that collaboratively achieves an objective. Just as a company has specialized departments working under a common mission, CrewAI lets you create an organization of AI agents with specialized roles collaborating to accomplish complex tasks ￼. The framework provides high-level simplicity for quick setup, along with low-level control to fine-tune agent behaviors and workflows.

Key idea: Instead of a single large language model handling everything, CrewAI enables a team of smaller, role-focused AI agents to divide and conquer tasks. This approach yields more natural decision-making, dynamic task delegation, and flexible problem-solving by combining each agent’s strengths ￼. Whether you’re automating content creation, data analysis, or decision workflows, CrewAI’s mix of Crews and Flows helps balance autonomous collaboration with precise control ￼.

Agents

An Agent in CrewAI is an autonomous AI unit – think of it as a team member with a defined role and expertise. Each agent operates with its own goal and can perform tasks, make decisions, use tools, and even communicate with other agents ￼. Agents are powered by an underlying language model (e.g. GPT-4 by default) and can be given a persona or backstory to guide their behavior ￼. For example, you might have a Researcher agent skilled at gathering information, and a Writer agent better at composing content ￼.

Key properties of an Agent include:
	•	Role: A short label defining the agent’s function or specialty (e.g. “Research Analyst” or “Content Writer”) ￼. This sets what the agent is supposed to do in the crew.
	•	Goal: A specific objective that guides the agent’s decisions. This ensures each agent works toward a sub-goal that contributes to the overall mission.
	•	Backstory: Background context or personality for the agent ￼. While optional, this can enrich the agent’s interactions (for instance, a creative backstory might make a writer agent more imaginative).
	•	LLM: The language model powering the agent’s reasoning. By default, agents use the model specified in your environment (e.g. GPT-4) ￼, but you can assign a different model or even a local LLM.
	•	Tools: A list of tools or functions the agent is allowed to use ￼. (We discuss tools below.) By default an agent has no tools, meaning it can only rely on the LLM; but giving it tools like web search or code execution can greatly extend its abilities.
	•	Memory: Whether the agent should maintain memory of its interactions (True by default) ￼. With memory, an agent can recall previous context in the conversation or task sequence.
	•	Delegation Ability: Whether the agent can delegate tasks to other agents ￼. By default this is off, but in advanced scenarios (especially hierarchical flows) you might enable certain agents to spawn or assign new tasks.
	•	Constraints: Additional settings like max iterations (how many thinking steps before giving up) ￼, timeouts, rate limits ￼, and verbosity for logging. These help control the agent’s operation in a long workflow.

In practice, creating an agent is straightforward. For example, one could create a researcher agent and a writer agent as:

from crewai import Agent

researcher = Agent(
    role="Researcher",
    goal="Conduct foundational research on a topic",
    backstory="An experienced analyst with a passion for uncovering insights"
)
writer = Agent(
    role="Writer",
    goal="Draft a comprehensive article based on research findings",
    backstory="A creative writer skilled at explaining technical concepts clearly"
)

Each agent now independently understands its mission. The researcher might later use tools to gather data, and the writer will eventually produce the final content. Agents can communicate and share information via the crew’s coordination (for example, the writer can be given the researcher’s output as input), enabling true collaboration.

Tasks

A Task in CrewAI represents a specific unit of work to be completed by an agent ￼. You can think of a task as a single assignment or objective, like “Research the latest AI trends” or “Write a summary report of the findings.” Tasks encapsulate all the details needed for execution – such as what needs to be done, which agent will do it, and what tools are allowed – so that the framework can carry it out autonomously.

Key properties of a Task include:
	•	Description: A clear description of what the task entails (what the agent should do) ￼. This is essentially the instruction given to the agent.
	•	Expected Output: A description of what a successful outcome looks like ￼. This guides the agent on how to format its result or what criteria it should meet. For example, an expected output might be “a list of three key findings in bullet points.”
	•	Assigned Agent: Optionally, you can specify which agent is responsible for this task ￼. If you leave this blank, the crew’s process (see Crews below) will determine assignment, but typically you bind each task to a particular agent role.
	•	Allowed Tools: Optionally, a list of tools that the agent can use for this task ￼. This restricts the agent to only those tools during this task’s execution – useful if certain tasks should only use specific resources (e.g. a database search tool for a data query task).
	•	Context (Dependencies): Optionally, you can list other tasks whose outputs will serve as input context for this task ￼. This is how you chain tasks together. For example, a “Write report” task might have the “Research findings” task as context, meaning the writer agent will be given the researcher’s output. The CrewAI framework ensures that dependent tasks run in the correct order and passes the results along.
	•	Output Handling: You can direct task outputs to files or enforce a data schema if needed. For instance, you can specify an output_file path to save the agent’s result to disk, or provide a Pydantic model class in output_json/output_pydantic to validate and structure the result JSON ￼. (These are optional, but useful for integrating with other systems.)

In CrewAI, tasks can be simple or complex and even collaborative. A collaborative task means multiple agents contribute to it or it depends on multiple inputs – the Crew’s orchestration will ensure agents share information and work together as needed ￼. For example, you might design a task where a Researcher gathers data and a Reviewer agent double-checks it before proceeding. This would be set up as two tasks with the second one taking the first as context, or as part of a hierarchical process (described later) where the manager coordinates the collaboration.

Defining tasks in code is straightforward. For example:

from crewai import Task

research_task = Task(
    description="Research the latest trends in AI and provide a summary.",
    expected_output="Summary of the top 3 emerging AI trends with brief explanations.",
    agent=researcher  # the Researcher agent we defined earlier will handle this
)
write_task = Task(
    description="Write an article explaining the AI trends identified by the research.",
    expected_output="A clear, well-structured article covering each trend in detail.",
    agent=writer,     # the Writer agent will handle this
    context=[research_task]  # use output of the research_task as input context
)

In this example, write_task depends on research_task – CrewAI will ensure the research is done first and its findings are passed to the writer agent. This way, tasks can be chained to form workflows. You can create as many tasks as needed and specify their order either by listing them sequentially or by using the context dependencies. CrewAI also provides a Visual Task Builder (in the Enterprise version) to design complex task flows visually ￼, but when coding, you’ll typically just compose Task objects as shown.

Tools

In CrewAI, Tools are plugins or functional modules that extend an agent’s capabilities. By default an agent can only converse and reason using its LLM, but with tools, it can take actions like calling APIs, performing web searches, writing to files, running code, and more. A tool in CrewAI is essentially a function or skill that an agent can invoke when needed (often via the model’s function-calling abilities) ￼.

CrewAI comes with a rich library of built-in tools for common needs – for example, web search tools, web scraping tools, various retrieval-augmented generation (RAG) searchers for documents, a code execution interpreter, and even image generation via DALL-E ￼ ￼. Each tool typically has a specific purpose, such as searching a website, reading a PDF, querying a database, etc. All tools are designed with error handling and caching, so agents can use them reliably and efficiently (they won’t repeat expensive calls unnecessarily) ￼.

You assign tools to agents based on what you want that agent to be able to do. For example, if our Researcher agent needs internet access, we might give it a WebSearchTool. If the Writer agent needs to save output to a file, we give it a FileWriteTool. In code, this looks like:

from crewai.tools import WebSearchTool, FileWriteTool

# Augment agents with tools
researcher.tools = [WebSearchTool()]
writer.tools = [FileWriteTool()]

Now the researcher can perform web queries, and the writer can write files. During task execution, the agent’s LLM is aware of its tools and can decide to invoke them to fulfill the task (for instance, the researcher’s prompt might cause it to call the search tool to gather data). CrewAI handles the tool invocation via function calling under the hood, and returns the tool results back to the agent’s context.

Developers can also create custom tools if needed (for specialized APIs or functions not covered by the built-ins). CrewAI’s documentation provides a guide on writing custom tools ￼, which typically involves subclassing a BaseTool and defining a run method. By equipping agents with the right tools, you ensure they have the actions needed to accomplish their tasks in an autonomous workflow.

Crews

A Crew is the top-level construct in CrewAI that brings everything together – it’s essentially the “team” of agents and the set of tasks they need to complete. When you set up a Crew, you specify which agents are part of it, what tasks they need to work on, and how those tasks should be executed (the process/strategy) ￼ ￼. The Crew is responsible for orchestrating the agents’ collaboration and ensuring the tasks get done in the desired way.

When creating a Crew, you typically provide:
	•	Agents: the list of Agent objects that will participate in this crew ￼. Each agent has its role and tools as configured earlier.
	•	Tasks: the list of Task objects that the crew must complete ￼. These can be ordered or interdependent; the crew will manage the execution order based on the chosen process (see below).
	•	Process: the execution strategy the crew will follow, such as sequential or hierarchical (default is sequential) ￼. This determines how the crew coordinates the tasks (explained in the next section).
	•	Manager LLM/Agent: (for hierarchical processes only) a language model or a special agent to act as the “manager” overseeing the crew ￼. This is required if you choose a hierarchical flow, since CrewAI needs a manager agent to delegate and validate tasks.
	•	Other settings: optional parameters like verbose (for logging), memory (shared memory store), max_rpm (rate-limit for API calls), cache toggles, etc. ￼ ￼. These fine-tune how the crew runs but are not required for a basic setup.

Think of a Crew as a project team: you assemble the team members (agents), give them a project plan (tasks), and decide on a management style (process). Once the crew is assembled, you kick off the execution, and CrewAI will manage the workflow. The primary method to start a crew is:

result = my_crew.kickoff()

Calling kickoff() will initiate the task execution loop according to the defined process flow ￼. The result returned is typically the final output of the workflow (for example, the output of the last task, or an aggregated result defined by you). CrewAI handles running each task with the appropriate agent and collecting their outputs.

Note: CrewAI also provides alternative kickoff methods for advanced usage – for example, kickoff_for_each() to run the same crew logic on a list of inputs one by one, or asynchronous variants like kickoff_async() ￼. These are useful for batch processing or non-blocking operation, but for most cases, a simple kickoff() is enough to run the crew from start to finish.

Execution Flows: Sequential vs. Hierarchical

When a crew runs, there are different ways to orchestrate the tasks. CrewAI currently supports two main process flows for task execution within a crew: Sequential and Hierarchical ￼.
	•	Sequential Process: Tasks are executed one after another in the order they were defined ￼. This is a straightforward linear flow. It’s ideal when you have a clear step-by-step procedure where each task’s output feeds into the next. In sequential mode, each task will simply run in sequence (respecting any context dependencies). For example, if you have tasks [A, B, C], the crew will run A, then B, then C, each by its assigned agent. This process is simple to track and reason about, and it’s the default behavior if you don’t specify otherwise. Most basic workflows (e.g., research then write then review) fit well in a sequential flow.
	•	Hierarchical Process: Tasks are executed under the guidance of a manager agent, which coordinates the crew’s activities ￼. This mode introduces a dynamic, collaborative flow similar to a project manager delegating tasks to team members. In a hierarchical flow, one agent takes on the role of “manager” (CrewAI can auto-create this if you provide a manager_llm, or you can designate one of your agents as the manager) ￼ ￼. The manager agent reads the overall goal and the list of tasks, then decides which task to delegate to which agent at each step, possibly adjusting the plan as it goes. It will also validate the outcomes of tasks: for instance, after an agent completes a task, the manager checks if the result is satisfactory or if follow-up is needed ￼. This process is powerful for complex or open-ended projects because the AI manager can make decisions on the fly about what to do next, rather than strictly following a predefined sequence.

In hierarchical mode, the crew essentially simulates a traditional corporate structure: the manager agent acts like a team lead, and the other agents are specialists. The manager might say “Agent A, you handle Task 1. Once you’re done, I (the manager) will review the output, then decide Task 2 should be done by Agent B,” and so on ￼. This collaborative flow ensures optimal use of each agent’s expertise and can catch errors or adjust tasks dynamically. However, it requires a strong language model for the manager (ideally GPT-4 or similar) to handle the reasoning – therefore CrewAI requires you to specify manager_llm (or a custom manager_agent) when using Process.hierarchical ￼. The framework uses that model to power the manager’s decisions.

Choosing a flow: If your workflow is well-defined and sequential, use the sequential process for simplicity. If your workflow might need more decision-making or could benefit from an AI project manager coordinating multiple agents, the hierarchical process is appropriate. You can start sequentially and later upgrade to hierarchical as needed – just by changing the crew’s process and adding a manager LLM. Both flows support multi-agent collaboration; the difference is that hierarchical gives the agents more autonomy to decide the order and validate results, whereas sequential follows a fixed order that you specify upfront ￼.

CrewAI Flows (Event-Driven Workflows)

In addition to crew process flows, CrewAI provides a separate abstraction called Flows (with a capital “F”), which are used to orchestrate more complex or conditional workflows in code. A CrewAI Flow is an event-driven workflow defined in Python, allowing you to chain together multiple steps (which can include running crews, calling functions, etc.) with fine-grained control ￼. Whereas a Crew by itself runs a predetermined set of tasks, a Flow lets you embed that into a larger logic – for example, handling inputs, branching conditions, looping, or coordinating multiple crews in sequence.

Key features of CrewAI Flows include:
	•	Structured Multi-Step Workflows: You can easily chain together multiple tasks or even multiple crews, creating complex pipelines of AI actions ￼ ￼. For instance, a flow might first execute a crew for data collection, then perform some custom Python processing on the results, then execute another crew for report generation.
	•	State Management: Flows provide built-in state sharing between steps ￼. Every flow run can carry a state (like a context dictionary), so data produced in one step can be accessed by subsequent steps without manual plumbing.
	•	Event-Driven and Decorator API: You define flow steps as Python methods annotated with decorators (like @start for the entry point, @listen for subsequent steps listening to events or conditions) ￼ ￼. This makes it easy to trigger the next action when a certain event or result is ready, enabling reactive behavior.
	•	Conditional Logic and Loops: Within flows, you have full Python control to use if/else logic, loops, error handling, etc. You can branch the execution based on agent outputs (for example, if a result meets some criteria, take one path, otherwise take another) ￼. This is essential for real-world scenarios where the workflow isn’t strictly linear.
	•	Integration with Code and Systems: Because flows are Python classes, you can integrate them with other code and external systems. For example, you might call a web API, do some calculations, or integrate with a database in between AI tasks. This allows hybrid workflows that combine traditional software operations with AI agent operations ￼.
	•	Reusability: You can encapsulate a Flow and reuse it or compose flows from sub-flows, leading to modular design.

In summary, CrewAI Flows give you production-level control over your AI automation. You might use a Flow when you need to coordinate at a higher level than a single crew. For instance, imagine an end-to-end content pipeline: a Flow could orchestrate one crew to brainstorm topics, then move to the next step (maybe a human approval or a filtering step), then run another crew to produce content for each approved topic, then perhaps call an external API to publish the content. All of this can be coded in a Flow class. Meanwhile, within each crew, the agents still collaborate autonomously on their part of the job.

It’s important to note that Crews and Flows are complementary. Crews provide the autonomous multi-agent teamwork, while Flows provide the glue and control logic to build bigger workflows ￼ ￼. You can even have a Flow that contains a Crew step, and that crew might itself use a hierarchical process internally – combining structured control with AI autonomy. This synergy allows developers to build complex, real-world AI applications: you use crews where you want the AI agents to have freedom to collaborate, and you use flows where you need deterministic orchestration or integration with other systems ￼.

Example: Putting It All Together

Let’s walk through a simple example of building an agentic workflow with CrewAI. Suppose we want to automate a media content creation process – for instance, researching AI trends and writing a blog post about them. We will set up two agents with distinct roles, give them appropriate tools, define two tasks (research and write), and run a crew sequentially.

Step 1: Define Agents. We create a Researcher agent and a Writer agent:

from crewai import Agent
from crewai.tools import WebSearchTool, FileWriteTool

researcher = Agent(
    role="Researcher",
    goal="Gather information on a given topic",
    backstory="A data analyst skilled at finding reliable information",
    tools=[WebSearchTool()]   # enable web searching capability
)
writer = Agent(
    role="Writer",
    goal="Produce a written article based on provided information",
    backstory="A creative writer with expertise in explaining technical topics",
    tools=[FileWriteTool()]   # enable file writing capability
)

Here, the Researcher can use the internet to search for info, and the Writer can save content to a file. (In a real case, you might also give the Writer a tool to retrieve the research results, but CrewAI will pass the context directly since we’ll link the tasks.)

Step 2: Define Tasks. Next, we create two tasks corresponding to each agent’s job:

from crewai import Task

task1 = Task(
    description="Research the latest trends in AI and summarize the top 3 findings.",
    expected_output="A summary of the top 3 AI trends, each with a brief explanation.",
    agent=researcher
)
task2 = Task(
    description="Write a blog post explaining the AI trends identified in the research.",
    expected_output="A well-structured blog post in Markdown covering the 3 trends.",
    agent=writer,
    context=[task1]  # use the output of task1 as context for task2
)

Task1 is assigned to the researcher (it will perform web searches and compile a summary). Task2 is assigned to the writer, and we link it to Task1’s output. By setting context=[task1], we ensure the Writer agent receives the Researcher’s summary as input context ￼.

Step 3: Create a Crew and Execute. Now we assemble the crew with both agents and both tasks, and specify a sequential process:

from crewai import Crew, Process

content_crew = Crew(
    agents=[researcher, writer],
    tasks=[task1, task2],
    process=Process.sequential  # execute tasks in order (research then write) [oai_citation_attribution:61‡docs.crewai.com](https://docs.crewai.com/concepts/tasks#:~:text=Code)
)

result = content_crew.kickoff()  # start the workflow

Because we chose a sequential process, the crew will execute Task1 then Task2 in order ￼. CrewAI will internally prompt the Researcher agent to fulfill Task1. The researcher will likely call the WebSearchTool to gather info, then produce the summary. Once Task1 is done, CrewAI takes the output and provides it as context to Task2. Then the Writer agent is prompted to fulfill Task2, using the summary to write the article. The writer might use the FileWriteTool to save the output to output_file if one was specified (in our case, we just expect a text result).

After crew.kickoff() completes, the result variable would contain the final output of the workflow (here, presumably the blog post content drafted by the Writer). We could then print it or otherwise use it. Because we set an expected output format, we know the result should be an article covering the three trends.

This simple scenario demonstrates the core pattern of using CrewAI:
	1.	Define agents with roles and tools appropriate to the subtasks.
	2.	Define tasks with clear instructions and link them if one depends on another.
	3.	Assemble a crew with those agents and tasks, choosing a process (sequential vs. hierarchical).
	4.	Run the crew (kickoff) to let the agents autonomously work through the tasks and produce the result.

From here, you could extend the example in many ways. For instance, you could add a Reviewer agent and a third task to have the content proofread (perhaps in a hierarchical flow, the reviewer could be a manager agent that verifies the writer’s output). Or you could wrap this entire crew in a higher-level Flow to repeat it for multiple topics or to integrate a human approval step before publishing. CrewAI is flexible: the same pattern of agents + tasks can be applied to a wide range of use cases – from media content pipelines to business data analysis, customer support automation, or even game-like simulations of agent teams. The key is to break your problem into roles and tasks, and let CrewAI handle the coordination.

Conclusion

With CrewAI, developers can build Agentic workflows where each component of a complex job is handled by a specialized AI agent, and those agents work together under an organized process. We covered the fundamental pieces: Agents (the autonomous actors), Tasks (units of work), Tools (extended capabilities), and Crews (the orchestrating container). We also discussed how tasks can be executed in simple sequences or with collaborative hierarchical coordination, and how CrewAI’s Flows can script together larger processes. This mix of conceptual overview and practical patterns should equip you to start building your own AI agent teams with CrewAI – whether for a media ops pipeline or any other domain requiring a bit of collaborative AI problem-solving. Use this as a reference as you code: define your agents and tasks clearly, choose the right execution flow, and gradually incorporate more complexity (like advanced tools or flows) as needed. CrewAI will take care of the rest, enabling your “AI crew” to autonomously drive toward your goal ￼ ￼.