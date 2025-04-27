# agent2agent-server
This repository is a wrapper for building google a2a servers, without explicitly copying the code to the native development codebase. This repository support - a2a server core, - a2a server utils,  - a2a server types under the package called commons. Infact this package is direct lift and shift from google's repository.

# How to install
### clone this repository
* option-1
    - `git clone https://github.com/pavanjava/agent2agent-server.git`
    - `pip install -e .`
* option-2

    - `pip install agent2agent-server` from pypi.org

# How to use this?
```python
# --- Agent Card Definition ---
from common_server.types import AgentCapabilities, AgentSkill, AgentCard

SEARCH_AGENT_CARD = AgentCard(
    name="Search Agent",
    description="A simple Agno A2A agent that search internet for user messages.",
    url="http://localhost:8001/agno-a2a", # Where this server will run
    version="0.1.0",
    capabilities=AgentCapabilities(
        streaming=False, # This simple agent won't stream
        pushNotifications=False,
        stateTransitionHistory=False
    ),
    authentication=None, # No auth for this simple example
    defaultInputModes=["text"],
    defaultOutputModes=["text"],
    skills=[
        AgentSkill(
            id="search-agent",
            name="Search Message",
            description="Receives a text message and searches it in internet.",
            inputModes=["text"],
            outputModes=["text"]
        )
    ]
)
```
```python
#--- agent_task_manager.py-----
import asyncio
from typing import Union, AsyncIterable
import logging

from common_server.server import TaskManager
from common_server.types import TaskResubscriptionRequest, SendTaskResponse, JSONRPCResponse, \
    GetTaskPushNotificationRequest, \
    GetTaskPushNotificationResponse, SetTaskPushNotificationRequest, SetTaskPushNotificationResponse, \
    SendTaskStreamingRequest, SendTaskStreamingResponse, SendTaskRequest, CancelTaskRequest, CancelTaskResponse, \
    GetTaskRequest, GetTaskResponse, Task, JSONRPCError, TextPart, TaskStatus, TaskState, Message

from search_agent import search_agent_team

# Configure basic logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


class SearchAgentTaskManager(TaskManager):
    def __init__(self):
        pass

    async def on_get_task(self, request: GetTaskRequest) -> GetTaskResponse:
        pass

    async def on_cancel_task(self, request: CancelTaskRequest) -> CancelTaskResponse:
        pass

    async def on_send_task(self, request: SendTaskRequest) -> SendTaskResponse:
        # call your agent here.
        pass

    async def on_send_task_subscribe(self, request: SendTaskStreamingRequest) -> Union[
        AsyncIterable[SendTaskStreamingResponse], JSONRPCResponse]:
        pass

    async def on_set_task_push_notification(self,
                                            request: SetTaskPushNotificationRequest) -> SetTaskPushNotificationResponse:
        pass

    async def on_get_task_push_notification(self,
                                            request: GetTaskPushNotificationRequest) -> GetTaskPushNotificationResponse:
        pass

    async def on_resubscribe_to_task(self, request: TaskResubscriptionRequest) -> Union[
        AsyncIterable[SendTaskResponse], JSONRPCResponse]:
        pass

```

```python
#---- agent_server.py-------
import logging

from common_server.server import A2AServer

from a2a_agent_cards import SEARCH_AGENT_CARD
from a2a_task_manager import SearchAgentTaskManager
from dotenv import load_dotenv, find_dotenv

load_dotenv(find_dotenv())

# Configure basic logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


if __name__ == "__main__":
    task_manager = SearchAgentTaskManager()
    server = A2AServer(
        host="localhost",
        port=8001,
        endpoint="/search-a2a", # Matches AgentCard URL path
        agent_card=SEARCH_AGENT_CARD,
        task_manager=task_manager
    )
    print("Starting A2A Server on http://localhost:8001")
    # Use server.start() which calls uvicorn.run
    # Note: For production, use a proper ASGI server like uvicorn or hypercorn directly
    server.start()
    # Alternatively, run directly with uvicorn:
    # uvicorn.run(server.app, host="localhost", port=8001)
```
