### N8N
curl -fsSL https://get.n8n.io | sh

________

### LangGraph - fluxo determinístico

```
from typing import TypedDict
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1", temperature=0)

class WorkflowState(TypedDict):
    previous_message: str
    task: str
    linear_issue_id: str
    pr_url: str
    test_report: str

# 1. Cria a tarefa no Linear
def planner(state: WorkflowState):
    prompt = f"""
    Mensagem original:
    {state['previous_message']}

    Transforme a mensagem em uma tarefa técnica objetiva,
    com objetivo, critérios de aceite e testes esperados.
    """
    task = llm.invoke(prompt).content

    # TODO: Criar issue via Linear API / MCP
    issue_id = create_linear_issue(task)

    return {"task": task, "linear_issue_id": issue_id}


# 2. Implementa e abre o Pull Request
def developer(state: WorkflowState):
    prompt = f"""
    Solicitação original:
    {state['previous_message']}

    Tarefa do Linear ({state['linear_issue_id']}):
    {state['task']}

    Implemente a tarefa no repositório e abra um PR.
    Respeite os critérios de aceite.
    """
    # TODO: Executar agente com ferramentas de GitHub,
    # acesso ao repositório e capacidade de executar comandos.
    pr_url = implement_and_open_pr(prompt)

    return {"pr_url": pr_url}


# 3. Executa testes e valida o PR
def tester(state: WorkflowState):
    prompt = f"""
    Contexto original:
    {state['previous_message']}

    Tarefa:
    {state['task']}

    Pull Request:
    {state['pr_url']}

    Valide os critérios de aceite, execute os testes
    e reporte falhas com evidências.
    """
    # TODO: Executar testes reais no repositório/CI.
    report = run_tests_and_validate(prompt)

    return {"test_report": report}


# Fluxo determinístico: Planner -> Developer -> Tester
graph = StateGraph(WorkflowState)

graph.add_node("planner", planner)
graph.add_node("developer", developer)
graph.add_node("tester", tester)

graph.add_edge(START, "planner")
graph.add_edge("planner", "developer")
graph.add_edge("developer", "tester")
graph.add_edge("tester", END)

app = graph.compile()

result = app.invoke({
    "previous_message": "Adicionar autenticação JWT à API",
    "task": "",
    "linear_issue_id": "",
    "pr_url": "",
    "test_report": "",
})

print(result["test_report"])


```
