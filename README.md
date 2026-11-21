# AI Fitness Assistant Agent

An AI-powered chatbot integrated into an existing fitness application. It helps users understand their sport-specific workout roadmaps, ask questions about exercises, review their workout history, and get fitness and nutrition guidance.

The application already stores exercise information and workout roadmaps. The agent does **not** create a new roadmap or replace the existing database. It reads the user's existing data and uses it to provide relevant answers and recommendations.

## 1. Use Case

The application is designed for people who go to the gym to improve their performance in another sport, such as:

- MMA
- Karate
- Tennis
- Swimming

Each user has a sport-specific workout roadmap stored in the application.

The chatbot acts as a personal fitness assistant. For example, users can ask:

- Why is this exercise included in my roadmap?
- Is it a good idea to increase the weight I use?
- How does this exercise help my sport?
- How has my workout performance changed?
- What should I eat to support my training?

The agent uses the user's existing application data and relevant fitness knowledge to answer these questions.

## 2. LLM & Provider

- **Provider:** GroqCloud
- **Model:** GPT-OSS 120B
- **Model ID:** `openai/gpt-oss-120b`
- **Agent Framework:** LangGraph

**What they do:**

- **GroqCloud:** Provides API access to run the language model.
- **GPT-OSS 120B:** Understands user questions, reasons about the available information, and selects tools.
- **LangGraph:** Manages the agent workflow and the steps taken to answer a request.

Documentation:
- [GroqCloud](https://console.groq.com/docs)
- [GPT-OSS](https://openai.com/index/introducing-gpt-oss/)
- [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview)

> Check the provider's current model availability and tool-calling support before implementation.

## 3. Agent Tools

The agent has access to six tools.

### A. Tools for Getting Data from the Application

**Tool 1: `get_user_context`**

- **Description:** Retrieves the user's basic information from the application.
- **Inputs:** None. The authenticated application session identifies the user.
- **Outputs:** `sport` (string), `fitness_goal` (string), `fitness_level` (string), `training_preferences` (object).

**Tool 2: `get_user_roadmap`**

- **Description:** Retrieves the user's current sport-specific workout roadmap.
- **Inputs:** None. Uses the authenticated user.
- **Outputs:** `roadmap_id` (string), `sport` (string), `exercises` (array), `training_schedule` (object).

**Tool 3: `get_exercise_details`**

- **Description:** Retrieves information about a specific exercise from the application's existing exercise database.
- **Inputs:** `exercise_id` (string).
- **Outputs:** `exercise_name` (string), `target_muscles` (array), `recommended_sets` (integer), `recommended_reps` (string), `exercise_purpose` (string).

**Tool 4: `get_user_workout_history`**

- **Description:** Retrieves the user's previous workout records, including sets, reps, weights, and training dates.
- **Inputs:** `period` (string), `exercise_id` (string, optional).
- **Outputs:** `workout_history` (array), `weights_history` (array), `training_dates` (array).

### B. Tools for Fitness and Nutrition Knowledge

**Tool 5: `search_fitness_knowledge`**

- **Description:** Searches a fitness knowledge base for information about exercise science, strength training, recovery, and sport-specific training.
- **Inputs:** `query` (string), `category` (string), `sport` (string, optional).
- **Outputs:** `found` (boolean), `results` (array).

**Tool 6: `search_nutrition_knowledge`**

- **Description:** Searches a nutrition knowledge base for information about food, nutrition, and general dietary guidance related to fitness and athletic performance.
- **Inputs:** `query` (string), `category` (string).
- **Outputs:** `found` (boolean), `results` (array).

## 4. How the Agent Works

The user sends a message through the chatbot. The agent understands the request and decides which tools are needed.

It can retrieve information from two places:

1. **The existing application database:** User profile, sport-specific roadmap, exercise details, and workout history.
2. **The knowledge base:** General fitness, training, recovery, and nutrition information.

The agent combines the relevant results and uses the language model to generate an answer.

### Example 1: Asking About an Exercise

**User:** "Why am I doing squats in my MMA roadmap?"

Possible steps:

1. `get_user_context`
2. `get_user_roadmap`
3. `get_exercise_details`
4. `search_fitness_knowledge` (if additional information is needed)

The agent explains why the exercise is included and how it may support the user's training.

### Example 2: Evaluating a Training Suggestion

**User:** "I think I should increase my squat weight. What do you think?"

Possible steps:

1. `get_user_context`
2. `get_user_roadmap`
3. `get_user_workout_history`
4. `search_fitness_knowledge`

The agent reviews the user's current training information and relevant guidance, then explains whether increasing the weight appears reasonable based on the available evidence.

### Example 3: Asking About Nutrition

**User:** "What should I eat to support my tennis training?"

Possible steps:

1. `get_user_context`
2. `search_nutrition_knowledge`

The agent uses the user's sport context and retrieved nutrition information to provide general guidance.

The agent does not need to call every tool for every question. It selects only the tools relevant to the user's request.

## 5. System Prompt

The system prompt defines the agent's role, available tools, and behavior.

Example:

```text
You are an AI Fitness Assistant integrated into a fitness application.

Your role is to help users understand their existing workout roadmaps,
answer exercise-related questions, evaluate training suggestions,
and provide general fitness and nutrition guidance.

Available tools:
- get_user_context
- get_user_roadmap
- get_exercise_details
- get_user_workout_history
- search_fitness_knowledge
- search_nutrition_knowledge

Instructions:
1. Understand the user's question before selecting tools.
2. Retrieve the user's sport, goals, and roadmap when relevant.
3. Use actual workout history when evaluating training suggestions.
4. Use the fitness knowledge base for exercise-related guidance.
5. Use the nutrition knowledge base for nutrition-related questions.
6. Explain how exercises may relate to the user's sport.
7. Give recommendations based on available data and retrieved knowledge.
8. Ask follow-up questions when essential information is missing.
9. Never invent user information, exercise details, or workout history.
10. Clearly distinguish existing application data from your own suggestions.

Safety:
- Do not diagnose injuries or medical conditions.
- Do not recommend training through pain.
- Do not provide medical treatment or rehabilitation instructions.
- Refer medical concerns to qualified professionals.
- Avoid unsupported training or nutrition claims.

Response style:
Be clear, conversational, and practical.
Explain the reasons behind recommendations instead of simply saying yes or no.
```

## 6. Simple System Architecture

```text
User
 |
 v
Chatbot Interface
 |
 v
Agent (GPT-OSS 120B via GroqCloud)
 |
 v
LangGraph
 |
 +-----------------------------+
 |                             |
 v                             v
Application Database       Knowledge Base
 |                             |
 +-- User Profile              +-- Fitness Knowledge
 +-- Workout Roadmap           +-- Nutrition Knowledge
 +-- Exercise Details
 +-- Workout History
 |
 v
Agent combines relevant information
 |
 v
Personalized Answer
```

## 7. Main Components

| Component | Purpose |
|---|---|
| GroqCloud | Runs the language model through an API |
| GPT-OSS 120B | Understands requests and selects tools |
| LangGraph | Manages the agent workflow |
| Existing Application Database | Provides user, roadmap, exercise, and workout data |
| Knowledge Base | Stores fitness and nutrition information |
| RAG | Retrieves relevant information from the knowledge base |
| Backend API | Connects the agent to the application securely |

### Important Design Notes

- Use the application's existing database for user profiles, roadmaps, exercises, and workout history.
- Do not create a second exercise database just for the chatbot.
- Use RAG for additional fitness and nutrition knowledge.
- The agent should only retrieve the information needed for each question.
- User identity should come from the authenticated application session, not from an ID invented or supplied by the language model.
- Keep API keys in the backend, not in the tablet application.
- The agent provides guidance; it does not replace medical or professional advice.

## 8. Expected Outcome

An AI-powered fitness assistant that understands each user's sport and existing workout roadmap, answers exercise-related questions, evaluates training suggestions using workout history, and provides relevant fitness and nutrition guidance.

The goal is to make the existing fitness application more helpful and personalized without replacing its current database or workout-planning features.
