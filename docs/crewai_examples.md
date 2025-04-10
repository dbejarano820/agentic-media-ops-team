# CrewAI Examples

## 📚 Example 1: Content Creation Workflow

### Overview
This example shows how to build an automated content generation pipeline with a Researcher and a Writer agent.

### Agents
```python
from crewai import Agent
from crewai.tools import WebSearchTool

researcher = Agent(
    role="Researcher",
    goal="Collect detailed information about Web3 trends",
    backstory="An experienced analyst skilled at web-based research",
    tools=[WebSearchTool()],
    verbose=True
)

writer = Agent(
    role="Writer",
    goal="Compose engaging articles using research",
    backstory="A creative content writer with expertise in technical storytelling",
    verbose=True
)
```

### Tasks
```python
from crewai import Task

research_task = Task(
    description="Find and summarize the top 5 Web3 trends of 2024.",
    expected_output="A clear summary of top 5 Web3 trends.",
    agent=researcher
)

writing_task = Task(
    description="Write a compelling blog post based on the research summary.",
    expected_output="A detailed and engaging blog post.",
    agent=writer,
    context=[research_task]
)
```

### Crew Execution
```python
from crewai import Crew

content_crew = Crew(
    name="Web3 Content Crew",
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process="sequential"
)

result = content_crew.kickoff()
print(result)
```

## 📈 Example 2: Data Analysis Workflow

### Overview
This example illustrates how to automate data analysis and report generation.

### Agents
```python
from crewai import Agent

analyst = Agent(
    role="Data Analyst",
    goal="Analyze datasets to uncover insights",
    backstory="Detail-oriented analyst skilled in data visualization and statistical analysis",
    verbose=True
)

reporter = Agent(
    role="Report Writer",
    goal="Produce clear, concise data reports",
    backstory="Expert in translating complex data insights into understandable reports",
    verbose=True
)
```

### Tasks
```python
analysis_task = Task(
    description="Analyze sales data and identify trends over the past year.",
    expected_output="Insights and visualizations highlighting sales trends.",
    agent=analyst
)

report_task = Task(
    description="Write a detailed report based on the analysis.",
    expected_output="A comprehensive report summarizing key insights.",
    agent=reporter,
    context=[analysis_task]
)
```

### Crew Execution
```python
analysis_crew = Crew(
    name="Sales Data Analysis Crew",
    agents=[analyst, reporter],
    tasks=[analysis_task, report_task],
    process="sequential"
)

report = analysis_crew.kickoff()
print(report)
```

## 🔍 Example 3: Collaborative Research Workflow

### Overview
Demonstrates hierarchical execution with a manager agent coordinating collaborative research tasks.

### Agents and Tasks
```python
manager_llm = "gpt-4"

researcher1 = Agent(role="Market Researcher", goal="Identify market trends.", backstory="Market research expert.")
researcher2 = Agent(role="Competitor Analyst", goal="Analyze competitors' strengths and weaknesses.", backstory="Strategic analyst.")

market_task = Task(description="Research emerging markets for our product.", expected_output="Detailed market trends.", agent=researcher1)
competitor_task = Task(description="Evaluate main competitors.", expected_output="Competitor analysis report.", agent=researcher2)

crew = Crew(
    name="Research Collaboration Crew",
    agents=[researcher1, researcher2],
    tasks=[market_task, competitor_task],
    process="hierarchical",
    manager_llm=manager_llm
)

final_report = crew.kickoff()
print(final_report)
```

These examples provide a starting point for building practical CrewAI workflows tailored to various applications.

