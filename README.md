# Project README

## Overview
This project uses LangChain, BeautifulSoup, and various APIs to create a comprehensive research assistant capable of web scraping, summarizing content, performing Google searches, and interacting with Airtable. The project sets up multiple agents to handle different tasks and manage a group chat for coordinated interactions.

## Prerequisites
- Python 3.8 or higher
- Install required packages from `requirements.txt`
- Environment variables for API keys

## Installation
1. Clone the repository:
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```

2. Create and activate a virtual environment:
    ```bash
    python3 -m venv env
    source env/bin/activate   # On Windows, use `env\Scripts\activate`
    ```

3. Install the required dependencies:
    ```bash
    pip install -r requirements.txt
    ```

4. Create a `.env` file in the project directory and add your API keys:
    ```env
    BROWSERLESS_API_KEY=your_browserless_api_key
    SERP_API_KEY=your_serper_api_key
    AIRTABLE_API_KEY=your_airtable_api_key
    OAI_CONFIG_LIST=path_to_your_config_list_json
    ```

## Functionality

### Google Search
Performs a Google search using the Serper API.
```python
def google_search(search_keyword):    
    url = "https://google.serper.dev/search"
    payload = json.dumps({"q": search_keyword})
    headers = {
        'X-API-KEY': serper_api_key,
        'Content-Type': 'application/json'
    }
    response = requests.post(url, headers=headers, data=payload)
    return response.text
```

### Web Scraping and Summarization
Scrapes the content of a website and summarizes it if the content is too large.
```python
def web_scraping(objective, url):
    response = requests.post(f"https://chrome.browserless.io/content?token={brwoserless_api_key}", ...)
    if response.status_code == 200:
        soup = BeautifulSoup(response.content, "html.parser")
        text = soup.get_text()
        if len(text) > 10000:
            return summary(objective, text)
        else:
            return text
```

### Airtable Integration
Fetches and updates records in Airtable.
```python
def get_airtable_records(base_id, table_id):
    url = f"https://api.airtable.com/v0/{base_id}/{table_id}"
    headers = {'Authorization': f'Bearer {airtable_api_key}'}
    response = requests.get(url, headers=headers)
    return response.json()

def update_single_airtable_record(base_id, table_id, id, fields):
    url = f"https://api.airtable.com/v0/{base_id}/{table_id}"
    headers = {
        'Authorization': f'Bearer {airtable_api_key}',
        "Content-Type": "application/json"
    }
    data = {"records": [{"id": id, "fields": fields}]}
    response = requests.patch(url, headers=headers, data=json.dumps(data))
    return response.json()
```

## Agents and Group Chat
The project sets up multiple agents for specific tasks and coordinates them in a group chat managed by `autogen`.

### User Proxy Agent
Handles user input.
```python
user_proxy = UserProxyAgent(name="user_proxy", ...)
```

### Researcher Agent
Performs web scraping and Google searches.
```python
researcher = GPTAssistantAgent(name="researcher", ...)
```

### Research Manager Agent
Coordinates research tasks.
```python
research_manager = GPTAssistantAgent(name="research_manager", ...)
```

### Director Agent
Interacts with Airtable.
```python
director = GPTAssistantAgent(name="director", ...)
```

### Group Chat
Manages the interaction between all agents.
```python
groupchat = autogen.GroupChat(agents=[user_proxy, researcher, research_manager, director], ...)
group_chat_manager = autogen.GroupChatManager(groupchat=groupchat, llm_config={"config_list": config_list})
```

## Starting the Conversation
Initiates a conversation with a research task.
```python
message = "Research the funding stage/amount & pricing for each company in the list: ..."
user_proxy.initiate_chat(group_chat_manager, message=message)
```

## Usage
Run the main script to start the agents and perform the tasks:
```bash
python main.py
```
